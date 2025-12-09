# HelloID-Conn-Prov-Target-Adapcare-PluriformZorg-readme
> [!WARNING]
> This connector has not been tested in a production environment! Therefore, changes might have to be made accordingly.

> [!IMPORTANT]
> This repository contains the connector and configuration code only. The implementer is responsible to acquire the connection details such as username, password, certificate, etc. You might even need to sign a contract or agreement with the supplier before implementing this connector. Please contact the client's application manager to coordinate the connector requirements.

> [!IMPORTANT]
> This connector was created using an API that was still in development. Therefore, in its current state, the import scripts only work when concurrent actions are set to one. This means it may take some time to complete, depending on the amount of data that needs to be retrieved.

<p align="center">
  <img src="">
</p>

## Table of contents

- [HelloID-Conn-Prov-Target-Adapcare-Pluriform](#helloid-conn-prov-target-adapcare-pluriform)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Supported  features](#supported--features)
  - [Getting started](#getting-started)
    - [Connection settings](#connection-settings)
      - [Certificate](#certificate)
    - [Correlation configuration](#correlation-configuration)
    - [Field mapping](#field-mapping)
    - [Account Reference](#account-reference)
  - [Remarks](#remarks)
    - [Correlation based on `logonname`](#correlation-based-on-logonname)
    - [Update](#update)
    - [GET Account at correlation](#get-account-at-correlation)
    - [Scope](#scope)
    - [Permissions](#permissions)
      - [Concurrent actions](#concurrent-actions)
      - [temporary-team-member permissions](#temporary-team-member-permissions)
      - [No user batching](#no-user-batching)
      - [Usergroup permissions](#usergroup-permissions)
    - [Still under construction](#still-under-construction)
  - [Development resources](#development-resources)
    - [API endpoints](#api-endpoints)
    - [API documentation](#api-documentation)
  - [Getting help](#getting-help)
  - [HelloID docs](#helloid-docs)

## Introduction

_HelloID-Conn-Prov-Target-Adapcare-Pluriform_ is a _target_ connector. _Adapcare-Pluriform_ provides a set of REST API's that allow you to programmatically interact with its data.

## Supported  features

The following features are available:

| Feature                                   | Supported | Actions                         | Remarks                                                                                            |
| ----------------------------------------- | --------- | ------------------------------- | -------------------------------------------------------------------------------------------------- |
| **Account Lifecycle**                     | ✅         | Create, Update, Enable, Disable | No Delete                                                                                          |
| **Permissions**                           | ✅         | Retrieve, Grant, Revoke         | Normal: userGroups (Professional and Pluriform Windows Client) and Dynamic: temporary-team-members |
| **Resources**                             | ❌         | -                               |                                                                                                    |
| **Entitlement Import: Accounts**          | ✅         | -                               |                                                                                                    |
| **Entitlement Import: Permissions**       | ✅         | -                               | Normal: userGroups (Professional and Pluriform Windows Client) and Dynamic: temporary-team-members |
| **Governance Reconciliation Resolutions** | ✅         | -                               | No Delete                                                                                          |

## Getting started

### Connection settings

The following settings are required to connect to the API.

| Setting                       | Description                              | Mandatory |
| ----------------------------- | ---------------------------------------- | --------- |
| ClientID                      | The ClientID to connect to the API       | Yes       |
| ClientSecret                  | The ClientSecret to connect to the API   | Yes       |
| Tenant                        | The Tenant for which you want to connect | Yes       |
| BaseUrl                       | The URL to the API                       | Yes       |
| certificatePublicBase64String | The Base64 string of public.crt          | Yes       |

#### Certificate
Adapcare Pluriform uses a certificate to obtain the authorization token. The connector handles this by using a Base64-encoded string of the `public.crt` file, eliminating the need for an agent and allowing it to run entirely in the cloud.
To retrieve the Base64-encoded string you can open the `public.crt` and copy the the content of the file, make sure to not include the opening header and closing footer.
If you rather extract the content using powershell you can use the code example below:

```powershell
# Path to your certificate file
$crtPath = "path\public.crt"

# Read the raw bytes of the certificate
$pem = Get-Content -Path $crtPath -Raw

# Convert to Base64 string
$base64String = ($pem -replace '-----.*?-----', '' -replace '\s', '').Trim()

# Output the Base64 string to the console
$base64String
```

### Correlation configuration

The correlation configuration is used to specify which properties will be used to match an existing account within _Adapcare-Pluriform_ to a person in _HelloID_.

| Setting                   | Value                             |
| ------------------------- | --------------------------------- |
| Enable correlation        | `True`                            |
| Person correlation field  | `PersonContext.Person.ExternalId` |
| Account correlation field | `logonName`                       |

> [!TIP]
> _For more information on correlation, please refer to our correlation [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems/correlation.html) pages_.

### Field mapping

The field mapping can be imported by using the _fieldMapping.json_ file.

### Account Reference

The account reference is populated with the property `id` property from _Adapcare-Pluriform_

## Remarks

### Correlation based on `logonname`
- **Correlation based on `logonname`, populated with email/UPN**: LogonName is populated with email/UPN. If the email/upn already existed for an account before it could be possible to correlate an old account that is no longer in use.

### Update
- **Different EmployeeNumber**: When updating an account based on the Id, the Id is ignored if a different EmployeeNumber is provided in the body. In that case, the EmployeeNumber is used to update the account instead. This can lead to inconsistencies, but the design of the connector handles this to avoid incorrect updates.
- **EmployeeNumber mandatory**: The Employeenumber is a mandatory property when sending the update request. Keep in mind that this should always stay the same otherwise another account is going to change, as described in the remark above.

### GET Account at correlation
- **Extra get request**: When retrieving a user without an ID, the API does not return the full account object. Therefore, an additional GET request is implemented in the correlation action of the create script.

### Scope
- **Scope in body get authorization token**: The scope in the body of the get authorizationToken request corresponds to the functionality of the connector. If you add additional functionality, keep in mind that the scope may need to be expanded.

### Permissions
#### Concurrent actions
- **Concurrent actions set to one**: The grant and revoke permission actions use the /v1/account/:id endpoint to update a user. therefore the concurrent actions should be set to one, otherwise the actions could interfere with each other.

#### temporary-team-member permissions
- **Dynamic permissions**: The temporary-team-member permissions uses dynamic permissions to grant and revoke temporary-team-members
- **CostCenter.Code**: The Dynamic Permission are currently based on `CostCenter.Code`:
 ```Powershell
  # Script Mapping lookup values
  # Lookup values which are used in the mapping to determine the CostCenter Code
  $costCenterLookupKey = { $_.CostCenter.Code }
 ```
 - **Configure Import Script**:  Must be the same as the values used in retrieve /static permissions.
 ```Powershell
  # Configure, must be the same as the values used in retrieve permissions
  $permissionReference = 'TemporaryTeamMembers'
  $permissionDisplayName = 'TemporaryTeamMembers'
```

#### No user batching
- **No user batching in import for dynamic permissions**: When retrieving the temporary team member, only one account can exist in the permission. Therefore, user batching is not necessary in this import script.

#### Usergroup permissions
- **two types of Usergroups**: The user groups request returns two types of permission these are separated by two permission definitions with each a permission, grant and revoke script.

### Still under construction
- **This has already been discussed with the supplier and will be fixed.** The API sometimes returns an error when you update properties or grant permissions directly one after another. This results in a 500 error: *'Not all changes successfully processed.*.
- **New has already been discussed with the supplier and will be fixed.**: An error occurs during the import script. Three scripts attempt to retrieve every single user account because the getAll call does not return all properties.
Error: Request could not be handled within the given time. No results found.

## Development resources

### API endpoints

The following endpoints are used by the connector

| Endpoint                  | Description                                  |
| ------------------------- | -------------------------------------------- |
| /v1/authorization/token   | Retrieve user information                    |
| /v1/account(/:id)         | Retrieve, create and update user information |
| /v1/temporary-team-member | Retrieve temporary team members              |
| /v1/usergroup             | Retrieve usergroup information               |

### API documentation

[Swagger](https://swagger.acc.carebeat-connector.nl/)

## Getting help

> [!TIP]
> _For more information on how to configure a HelloID PowerShell connector, please refer to our [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems.html) pages_.

> [!TIP]
>  _If you need help, feel free to ask questions on our [forum](https://forum.helloid.com/forum/helloid-connectors/provisioning/5380-helloid-conn-prov-target-adapcare-pluriformzorg)_.

## HelloID docs

The official HelloID documentation can be found at: https://docs.helloid.com/
