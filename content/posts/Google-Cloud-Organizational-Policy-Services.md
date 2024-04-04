---
title: "Secure Google Cloud with Organizational Policy Services"
description: "Protect your Google Cloud resources and workloads with  "
cascade:
  featured_image: '/images/google-dc-eemshaven-security-station.webp'

date: 2022-12-?T09:14:33+01:00

---

Here comes some great content about how to use Organizational Policy Services (Org. Policies) in Google Cloud to secure your Cloud resources and workloads.

Org structure

What is it?

Constraints

- Protect
- Limit
- Be in control

How to use it?

My favorite Organizational Policy to apply
1. Name + What it does? + 
2.
3.
4.
5.

===

Google Cloud gives you the ability to restrict the usage of resources and services when utilising Organisation Policy Services. The restrictions are named constraints. Example of Organization Policies Service Constraints are: Limit the regions where services can be deployed or whitelist the Disk Images that can be used by developers. Organisation Policy Services will help you to increase the level of control and increase the overall security in a Cloud native service and centralised way within your organisation. Be aware that Organisation Policies will only apply on newly created resources and not on existing resources.

We seen a lot of customer who don't utilise the power of Organisation Policy Services in their Cloud.

Our recommendation is to apply additional Organisation Policy Services (constraints) on the organisation and/or folder level. We recommend looking into and applying the following core Organisation Policy Services constraints. The list is curacted based on our experience as Xebia Cloud and Google Cloud best practices.

Organisation Policy Services Core Constraints


Domain-restricted sharing (iam.allowedPolicyMemberDomains)
Resource Location Restriction (gcp.resourceLocations)
Define trusted image projects (compute.trustedImageProjects)
Skip default network creation (compute.skipDefaultNetworkCreation)
Prevent removal of Shared VPC Project (compute.restrictXpnProjectLienRemoval)
Disable VM serial port access (compute.disableGlobalSerialPortAccess)
Require OS Login (compute.requireOsLogin)
Define allowed external IPs for VM instances (compute.vmExternalIpAccess)
Restrict Public IP access on Cloud SQL instances (sql.restrictPublicIp)
Disable service account key creation (iam.disableServiceAccountKeyCreation)
Disable Guest Attributes Access (compute.disableGuestAttributesAccess)
Prevent auto grant permissions to default App Engine and Compute Engine Service Account (iam.automaticIamGrantsForDefaultServiceAccounts)
Enforce Public Access Prevention (storage.publicAccessPrevention)
Enforce uniform bucket-level access (storage.uniformBucketLevelAccess)


Additional Policy Controls
In addition to the Organisation Policy Services Core Constraints, you can also apply the additional Policy Controls to extend and further increase the security of your Google Cloud.. 


1. Limit session and GCloud timeouts
2. Disable Cloud Shell
3. Use phishing resistant security keys
4. Enable access transparency
5. Enable access approval

More details about Organization Policy Constraints can be found here: https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints 