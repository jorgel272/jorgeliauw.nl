---
title: "Google Cloud CFT Scorecard"
description: "Scan your Cloud with Cloud Foundation Toolkit (CFT)"
cascade:
  featured_image: '/images/google-dc-eemshaven-security-station.webp'

date: 2023-03-10T12:00:00+01:00
---
Check your current Google Cloud setup against Google Cloud Foundation Toolkit (CFT), 
with the CFT Scorecard.

We will export data, IAM, Org Policy and Access Policy data from Cloud Asset Inventory (CAI).

The CAI exports can be done on a organization, folder or project level.  
# could also use --project --folder or --organization

The CFT Scorecard is integrated into the CFT CLI and provides an easy integration with Forseti Config Validator. It can be used to print a scorecard of your GCP environment, for resources and IAM policies in Cloud Asset Inventory (CAI) exports. The policies tested are based on constraints and constraint templates from the Config Validator policy library.

You'll need to use CAI to generate the resource and IAM policy information for the project.

https://github.com/GoogleCloudPlatform/cloud-foundation-toolkit/blob/master/cli/docs/scorecard.md


What?
Why?
How?

{{< figure src="/images/Jorge-Liauw-Certified-Google Cloud Certified-Professional Cloud Security Engineer-30122022.png" title="" >}}

https://github.com/GoogleCloudPlatform/cloud-foundation-toolkit/blob/master/cli/docs/scorecard.md

https://cloud.google.com/asset-inventory/docs/access-control#required_permissions

Scorecard has two dependencies:
Cloud Asset Inventory (CAI)
Policy Library


```
export GOOGLE_PROJECT=$DEVSHELL_PROJECT_ID
export CAI_BUCKET_NAME=CAI-$GOOGLE_PROJECT
```

# For downloading CAI data from GCS

```
gcloud config set core/project $GOOGLE_PROJECT   
```

# For exporting CAI data to GCS
```
gcloud services enable cloudasset.googleapis.com
```

#Create Service Account
```
gcloud beta services identity create --service=cloudasset.googleapis.com --project=$GOOGLE_PROJECT
```

#Grant Storage admin role to the Cloud Asset Service account

```
gcloud projects add-iam-policy-binding ${GOOGLE_PROJECT}  \
   --member=serviceAccount:service-$(gcloud projects list --filter="$GOOGLE_PROJECT" --format="value(PROJECT_NUMBER)")@gcp-sa-cloudasset.iam.gserviceaccount.com \
   --role=roles/storage.admin
```

# Create Google Cloud Storage Buckets

```
gsutil mb gs://$CAI_BUCKET_NAME
gsutil mb gs://$PUBLIC_BUCKET_NAME
```

# Export resource data

```
gcloud asset export --output-path=gs://$CAI_BUCKET_NAME/resource_inventory.json \
    --content-type=resource \
    --project=$GOOGLE_PROJECT 
```

# Check export status
```
gcloud asset operations describe projects/819251468686/operations/ExportAssets/RESOURCE/fb84873413de923aa1818964564fec8d
```

# Export CAI data

# Export resource data
```
gcloud asset export \
    --output-path=gs://$CAI_BUCKET_NAME/resource_inventory.json \
    --content-type=resource \
    --project=$GOOGLE_PROJECT
```

# Export IAM data
```
gcloud asset export \
    --output-path=gs://$CAI_BUCKET_NAME/iam_inventory.json \
    --content-type=iam-policy \
    --project=$GOOGLE_PROJECT
```

# Export org policy data

```
gcloud asset export \
    --output-path=gs://$CAI_BUCKET_NAME/org_policy_inventory.json \
    --content-type=org-policy \
    --project=$GOOGLE_PROJECT
```
    
# Export access policy data

```
gcloud asset export \
    --output-path=gs://$CAI_BUCKET_NAME/access_policy_inventory.json \
    --content-type=access-policy \
    --project=$GOOGLE_PROJECT
```

# OS X
```
curl -o cft https://storage.googleapis.com/cft-cli/latest/cft-darwin-amd64
```

# Linux
```
curl -o cft https://storage.googleapis.com/cft-cli/latest/cft-linux-amd64
```

# Make cft executable
```
chmod +x cft
```

```
git clone https://github.com/forseti-security/policy-library.git

cp policy-library/samples/storage_denylist_public.yaml policy-library/policies/constraints/
```

#Run CFT scorecard application

```
./cft scorecard --policy-path=./policy-library/ \
    --bucket=$CAI_BUCKET_NAME
```

===

Clone the Policy Library:

git clone https://github.com/forseti-security/policy-library.git
Copied!
You realize Policy Library enforces policies that are located in the policy-library/policies/constraints folder, in which case you can copy a sample policy from the samples directory into the constraints directory.

cp policy-library/samples/storage_denylist_public.yaml policy-library/policies/constraints/


===

Add more constraints to CFT scorecard
Add the following constraint to ensure you are entirely aware who has the IAM roles/owner role aside from your allowlisted user:

```
# Add a new policy to blacklist the IAM Owner Role
cat > policy-library/policies/constraints/iam_allowlist_owner.yaml << EOF
apiVersion: constraints.gatekeeper.sh/v1alpha1
kind: GCPIAMAllowedBindingsConstraintV3
metadata:
  name: allowlist_owner
  annotations:
    description: List any users granted Owner
spec:
  severity: high
  match:
    target: ["organizations/**"]
    exclude: []
  parameters:
    mode: allowlist
    assetType: cloudresourcemanager.googleapis.com/Project
    role: roles/owner
    members:
    - "serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com"
EOF
```

Rerun CFT scorecard:

./cft scorecard --policy-path=policy-library/ --bucket=$CAI_BUCKET_NAME


Ok, it all looks clean, but let's look at roles/editor, too.

Set two extra variables to help with the new constraint creation:

```
export USER_ACCOUNT="$(gcloud config get-value core/account)"
export PROJECT_NUMBER=$(gcloud projects describe $GOOGLE_PROJECT --format="get(projectNumber)")
```

Create the following constraint that will allowlist all the valid accounts:

```
# Add a new policy to allowlist the IAM Editor Role
cat > policy-library/policies/constraints/iam_identify_outside_editors.yaml << EOF
apiVersion: constraints.gatekeeper.sh/v1alpha1
kind: GCPIAMAllowedBindingsConstraintV3
metadata:
  name: identify_outside_editors
  annotations:
    description: list any users outside the organization granted Editor
spec:
  severity: high
  match:
    target: ["organizations/**"]
    exclude: []
  parameters:
    mode: allowlist
    assetType: cloudresourcemanager.googleapis.com/Project
    role: roles/editor
    members:
    - "user:$USER_ACCOUNT"
    - "serviceAccount:**$PROJECT_NUMBER**gserviceaccount.com"
    - "serviceAccount:$GOOGLE_PROJECT**gserviceaccount.com"
EOF
```

Rerun CFT scorecard:

./cft scorecard --policy-path=policy-library/ --bucket=$CAI_BUCKET_NAME