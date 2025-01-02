---
title: Authorization in Azure Sovereign Cloud
date: 2025-01-02 14:30:00
tags:
- Azure
- Sovereign Cloud
---

This post is to:

* Solve credential type mismatch which causes errors like "object has no attribute 'get_token'"

* Share experience to create cloud management clients with Azure Python SDK"

## Authorize with ServicePrincipal (Legacy, Deprecated)

Authorize with a ServicePrincipalCredential, which contains cloud_enviroment information. 

* Create an instance of ServicePrincipalCredentials, with cloud_enviroment from msrestazure.azure_cloud provided
  ```python
  from azure.common.credentials import ServicePrincipalCredentials
  import msrestazure
  cloud_environment = msrestazure.azure_cloud.AZURE_CHINA_CLOUD
  credential = ServicePrincipalCredentials(
    tenant,
    client_id,
    secret,
    cloud_environment=cloud_environment
  )
  ```

* Then create a client instance with resource manager endpoint URL
  ```python
  from azure.mgmt.netapp import AzureNetAppFilesManagementClient
  client = AzureNetAppFilesManagementClient(
    credential,
    subscription_id,
    base_url=cloud_environment.endpoints.resource_manager
  )
  ```

Clients under this authorization path (not all listed, subject to change):

* azure.mgmt.netapp.AzureNetAppFilesManagementClient
  
  Please note **AzureNetAppFilesManagementClient** and **NetAppManagementClient** may follow different authorization pattern.

Parameters relevant to sovereign cloud:

* credential = TokenCredential (ClientSecretCredential)
* base_url = Resource Manager Endpoint
* (legacy version) credential_scopes = base_url + [/].default

## Authorize with TokenCredential

Authorize with a TokenCredential, which implemented get_token() method, typically
ClientSecretCredential. 

Clients under this authorization path (not all listed):

* azure.mgmt.network.NetworkManagementClient
* azure.mgmt.storage.StorageManagementClient
* azure.mgmt.resource.ResourceManagementClient

Parameters relevant to sovereign cloud:

* credential = TokenCredential (ClientSecretCredential)
* base_url = Resource Manager Endpoint
* (legacy version) credential_scopes = base_url + [/].default

## Notes

* Authorization process is made in the actual client api calls rather than creation of clients.

## Package Structure

### azure.identity

azure.identity.AzureAuthorityHosts

* Azure Authority
  
  azure_authority = azure.identity.AzureAuthorityHosts.[AZURE_CHINA|AZURE_PUBLIC_CLOUD]

* ClientSecretCredential

  authority = azure_authority

  has get_token()

### msrestazure

msrestazure.azure_cloud

* Endpoints

* Resource Manager Endpoint

  arm_endpoint = msrestazure.azure_cloud.[AZURE_CHINA|AZURE_PUBLIC_CLOUD].endpoints.resource_manager

<div style="display:none">
### msrestazure.azure_active_directory
</div>