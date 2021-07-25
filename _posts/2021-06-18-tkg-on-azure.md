---
layout: post
title: TKG Installation on Azure Environment which doesn't support multi AZ
categories: [TKG, Azure, Troubleshooting]
---

## Environment
- VMWare Tanzu Kubernetes Grid 1.3.1
- Azure Cloud

I was trying to install TKG in Azure Region Korea Central. Basically, TKG support High Availability, and needs multi AZ
environment. Thus, it doesn't show Korea Central in the region list in TKG UI.

This feature is planned to support in 2021, but currently not supported yet. Please refer "Availability Zones presence" in [this](https://azure.microsoft.com/en-us/global-infrastructure/geographies/#geographies).

Though you change the value AZURE_LOCATION to Korea Central in the editor after you configured in installation UI, you will see InvalidAvailabilityZone error during installation of management cluster.
```text
E0517 17:47:36.454745 1 controller.go:257] controller-runtime/controller "msg"="Reconciler error" "error"="failed to reconcile AzureMachine: failed to create virtual machine: failed to create VM mgmt-cluster-md-0-b7nfk in resource group my-group: compute.VirtualMachinesClient#CreateOrUpdate: Failure sending request: StatusCode=400 -- Original Error: Code=\"InvalidAvailabilityZone\" Message=\"The zone(s) '1' for resource 'Microsoft.Compute/virtualMachines/mgmt-cluster-md-0-b7nfk' is not supported. The supported zones for location 'koreacentral' are ''\"" "controller"="azuremachine" "name"="mgmt-cluster-md-0-b7nfk" "namespace"="tkg-system"​
I0517 17:48:12.269768 1 azuremachine_controller.go:227] controllers/AzureMachine "msg"="Reconciling AzureMachine" "AzureCluster"="mgmt-cluster" "azureMachine"="mgmt-cluster-control-plane-p4842" "cluster"="mgmt-cluster" "machine"="mgmt-cluster-control-plane-sphph" "namespace"="tkg-system"​
```
Then, how could I resolve this issue?

Go to ~/.tanzu/tkg/providers/infrastructure-azure/ytt and create this availability-set-workaround.yaml file.

```yaml
#@ load("@ytt:overlay", "overlay")

#@overlay/match by=overlay.subset({"kind":"MachineDeployment"}), expects="0+"
---
kind: MachineDeployment
spec:
  template:
    spec:
      #@overlay/match missing_ok=True
      failureDomain:
```

Your configuration for management cluster named mc.yaml is as below.
```yaml
AZURE_CLIENT_ID: <encoded:xxxxxx>
AZURE_CLIENT_SECRET: <encoded:yyyyyyy>
AZURE_CONTROL_PLANE_MACHINE_TYPE: Standard_D2s_v3
AZURE_CONTROL_PLANE_SUBNET_CIDR: 10.0.2.0/24
AZURE_CONTROL_PLANE_SUBNET_NAME: controlplane-subnet
AZURE_LOCATION: koreacentral
AZURE_NODE_MACHINE_TYPE: Standard_D2s_v3
AZURE_NODE_SUBNET_CIDR: 10.0.3.0/24
AZURE_NODE_SUBNET_NAME: workernode-subnet
AZURE_RESOURCE_GROUP: my-group
AZURE_SSH_PUBLIC_KEY_B64: caUNTRk1QMkdzYkVzQ2NpbTJkQXp2QlowUk1sVGxGZCV09sZHpvOThiWitvN1MxcWNybWxnUTBXS3F5Yk9lL3JvTTBQWTJSTm1QVFpibDJGeEdueEtkdHVlOFlmbmFoK2dhOEhOcTFyNm0wbWQyQ2s3b3MxeGEyblEvalpoeHlKeEFqaDIzVW1LZHB3TjlFMERRM25OL1lmV2xteEVveXE2R215YXJPUGxBR2N4RUlyMGgvUGVGUHQyOEwvVFg1WDB5dHVlZ0ZSUHpML2RvWEQyU3FFc1U2WGl5L3RXTFRDSEVyUnFoN2Y2L2JNZkFCR1RxMVVzODhzRFhrWTRWQzJyajhLa2Q2TlNhSE9FTlBQTDRJQXVMVDJCWlZmQUhMUFovNHYrUFBLTGZYaUhHQ2JvRTN2QXQ5VlZSeDhzVEh6bWVXa0xVSVlNV2lOMkJxSDlqVkF3Nm5sQXdtZ0c2WjFWTXQzS05HVFJWOUVzUUFCNGRobm1tbW9SdEJ6OHZzWUxaL09IVy8xMGs9IGdlbmVyYXRlZC1ieS1henVyZQ==
AZURE_SUBSCRIPTION_ID: 0b504234-3ba1-4543-sdfd-640221e6bbeb
AZURE_TENANT_ID: 370bxdrwe-03b0-42x1-bd8c-eb9824ecc423
AZURE_VNET_CIDR: 10.0.0.0/16
AZURE_VNET_NAME: my-group-vnet
AZURE_VNET_RESOURCE_GROUP: my-group
CLUSTER_CIDR: 100.96.0.0/11
CLUSTER_NAME: mgmt-cluster
CLUSTER_PLAN: dev
ENABLE_CEIP_PARTICIPATION: "true"
ENABLE_MHC: "true"
IDENTITY_MANAGEMENT_TYPE: none
INFRASTRUCTURE_PROVIDER: azure
LDAP_BIND_DN: ""
LDAP_BIND_PASSWORD: ""
LDAP_GROUP_SEARCH_BASE_DN: ""
LDAP_GROUP_SEARCH_FILTER: ""
LDAP_GROUP_SEARCH_GROUP_ATTRIBUTE: ""
LDAP_GROUP_SEARCH_NAME_ATTRIBUTE: cn
LDAP_GROUP_SEARCH_USER_ATTRIBUTE: DN
LDAP_HOST: ""
LDAP_ROOT_CA_DATA_B64: ""
LDAP_USER_SEARCH_BASE_DN: ""
LDAP_USER_SEARCH_FILTER: ""
LDAP_USER_SEARCH_NAME_ATTRIBUTE: ""
LDAP_USER_SEARCH_USERNAME: userPrincipalName
OIDC_IDENTITY_PROVIDER_CLIENT_ID: ""
OIDC_IDENTITY_PROVIDER_CLIENT_SECRET: ""
OIDC_IDENTITY_PROVIDER_GROUPS_CLAIM: ""
OIDC_IDENTITY_PROVIDER_ISSUER_URL: ""
OIDC_IDENTITY_PROVIDER_NAME: ""
OIDC_IDENTITY_PROVIDER_SCOPES: ""
OIDC_IDENTITY_PROVIDER_USERNAME_CLAIM: ""
SERVICE_CIDR: 100.64.0.0/13
TKG_HTTP_PROXY_ENABLED: "false"
```

Execute the command to install management cluster.
```shell
$ tanzu management-cluster create -f mc.yaml -v 9
```
