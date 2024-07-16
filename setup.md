## Installation and Configuration

### Entra Id Application Registration
This solution package relies entirely on a single Entra ID application registration for authentication to the Microsoft Graph.  All authentication is performed using an app-only context with a Client Id and Secret. To operate in a least privileged configuration, create a new app registration with the following permissions:

| API | Permission | Justification |
|-----|------------------------|---------------|
| Microsoft&nbsp;Graph | Application.Read.All | Required to read all applications from Entra ID using Microsoft Graph’s applications endpoint. |
| Microsoft&nbsp;Graph | Sites.Read.All       | Required to read all site collections from Microsoft Graph's GetAllSites endpoint. |
| Microsoft&nbsp;Graph | Sites.Selected       | "Manage" permissions to the “host” site collection to allow list provisioning, list item operations. |

Example reference configuration:
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/entra-perms.png" width="800"></kbd>
</p>

### Site Collection Permissions
In order for the Power Automate Flows to provision/update the SharePoint List and schema, the new Entra ID App Registration requries "Manage" access to the target SharePoint site.  This least privilege role can be to the app registration by executing the following PowerShell code as user (or app registration) having Microsoft Graph API > Sites.FullControl.All rights.

``` PowerShell
#requires -modules "PnP.PowerShell"

$tenant   = "contoso" # tenant name
$clientId = "00000000-0000-0000-0000-000000000000" # Entra Id Client Id
$siteUrl  = "https://contoso.sharepoint.com/sites/sitename" # target SharePoint Site Url

# connect to spo admin center
Connect-PnPOnline `
    -Url "$tenant-admin.sharepoint.com" `
    -Interactive `
    -ForceAuthentication


# define new site app permissions with "write" perms
$r = Grant-PnPAzureADAppSitePermission `
            -DisplayName "SharePoint App Registrations Dashboard" `
            -AppId $clientId `
            -Site $siteUrl `
            -Permissions "Write"


# update new association with "manage" permissions
Set-PnPAzureADAppSitePermission `
            -PermissionId $r.Id `
            -Site $siteUrl `
            -Permissions "Manage"

```
 
### Solution Import

#### Power Platform Environment Creation (optional)
Optionally, create a new Power Platform Environment using the following article as reference:
[Create and manage environments in the Power Platform admin center](https://learn.microsoft.com/en-us/power-platform/admin/create-environment)

#### Environment Solution Import
In the target environment, import the solution package using the following article as reference:
[Import Solution](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/import-update-export-solutions)
 
### Solution Environment Variable Configuration
After the solution has been successfully imported, select the Application Principal Reporting solution.  

<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/solution-overview.png" width="800"></kbd>
</p>

#### Configure the following environment variables:

| Environment&nbsp;Variable      | Description |
|---------------------------|-------------|
| Tenant Id                 | Microsoft 365 Tenant Id |
| Client Id                 | Application Registration Client/App Id |
| Secret&nbsp;(Azure&nbsp;Key&nbsp;Vault)  | All flows check for the existence of the AKV secret value first. If the AKV secret is not configured or fails,the flows fall back to the reading the Secret (Plaintext) environment variable. Do not configure this variable if you plan to only use the Secret (Plaintext) environment variable for secret storage. [AKV Configuration Details](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables-azure-key-vault-secrets)|
| Secret (Plaintext)        | This value is a fallback value if the Secret (Azure Key Vault) variable is not configured. |
| Site URL                  | Full URL of the SharePoint site hosting the list/dashboard |
| List Name                 | Title of the SharePoint List.  The default value is "SharePoint Application Principals" "

#### Provision SharePoint List
Manually run the *Application Principal Reporting - Provision SharePoint List Schema* Flow one time to automatically provision the SharePoint list and list schema used for data storage.  This flow will only take a few seconds to run.  It may be necessary to run the flow each time an solution update is released in the event there is a change to the SharePoint list schema.

#### Configure Column Formatter
(Optional) If you would like all highly privilaged application scoped permssions to be highlighted in red, copy and pasted the JSON sample [column-formatter.json](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/column-formatter.json) into the *Format this Column* section on both the *Graph Application Permissions* and *SharePoint Application Permissions* columns.

<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/column-format-result.png" width="800"></kbd>
</p>

#### Schedule Configuration
(Optional) Open and edit each of the three scheduled Power Automate Flows and adjust the recurrance schedule to your desired frequency and start time.

|Flow Name|Default Start Time|Default Frequency|
|-|-|-|
|*Application Principal Reporting - Import Application Principals*       | 08:00 PM Eastern | Daily |
|*Application Principal Reporting - Import Site Collection Associations* | 09:00 PM Eastern | Daily |
|*Application Principal Reporting - Import Sign-In Activity*             | 11:00 PM Eastern | Daily |


