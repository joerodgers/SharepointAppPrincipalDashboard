## Overview

This solution provides SharePoint Online Administrators with insights into all Entra ID App Registrations in a tenant configured with one or more SharePoint Online or Microsoft Graph related site permissions.  The solution is packaged as a Power Platform Solution package.  It contains several Power Automate Workflows use to inventory Entra ID App Registrations, associate SharePoint Online sites (sites.selected) and import sign-in activity. 

These workflows are highly optimized and have been tested on multiple tenants (inculding multi-geo) with 100k+ site collections and 15k+ (total) App Registrations. 

#### App Registration Details
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/list-1.png" width="800"></kbd>
</p>

#### Permission Details
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/list-2.png" width="800"></kbd>
</p>

## Setup

### Entra Id Application Registration
This solution package relies entirely on a single Entra ID application registration for authentication to the Microsoft Graph.  All authentication is performed using an app-only context with a Client Id and Secret. To operate in a least privileged configuration, the associated app registration requires the following permissions:

| Application Permission | Justification |
|------------------------|---------------|
|Application.Read.All    | Required to read all applications from Entra ID using Microsoft Graph’s applications endpoint. |
|Sites.Read.All          | Required to read all site collections from Microsoft Graph’s GetAllSites endpoint. |
|Sites.Selected          | “Manage” permissions to the “host” site collection to allow list provisioning, list item operations. |

Example reference configuration:
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/entra-perms.png" width="800"></kbd>
</p>

### Site Collection Permissions
To grant the "Manage" permission to the app registration to the “host” SharePoint site collection, execute the following PnP code, substituting the necessary variable values:
``` PowerShell
#requires -modules "PnP.PowerShell"

$tenant   = "contoso"
$clientId = "00000000-0000-0000-0000-000000000000" 
$siteUrl  = "https://contoso.sharepoint.com/sites/sitename"


Connect-PnPOnline `
    -Url "$tenant-admin.sharepoint.com" `
    -Interactive `
    -ForceAuthentication


$r = Grant-PnPAzureADAppSitePermission `
            -DisplayName "SharePoint App Registrations Dashboard" `
            -AppId $clientId `
            -Site $siteUrl `
            -Permissions "Write"


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

Configure the following environment variables:

| Environment&nbsp;Variable      | Description |
|---------------------------|-------------|
| Tenant Id                 | Microsoft 365 Tenant Id |
| Client Id                 | Application Registration Client/App Id |
| Secret&nbsp;(Azure&nbsp;Key&nbsp;Vault)  | All flows check for the existence of the AKV secret value first. If the AKV secret is not configured or fails,the flows fall back to the reading the Secret (Plaintext) environment variable. Do not configure this variable if you plan to only use the Secret (Plaintext) environment variable for secret storage. |
| Secret (Plaintext)        | This value is a fallback value if the Secret (Azure Key Vault) variable is not configured. |
| Site URL                  | URL of the SharePoint site hosting the list/dashboard |
| List Name                 | Default value is "SharePoint Application Principals" "

## Power Automate Flow Details

### Application Principal Reporting - Provision SharePoint List Schema
The Application Principal Reporting - Provision SharePoint List Schema flow should only need to be executed one time.  Its purpose is to provision the necessary SharePoint custom list and associated list fields in the target SharePoint site.  The custom list is where all the application registration data is written to by the other flows.

#### Variable Declaration
<table border=0>
    <tr>
        <td width="500" height="500">
            <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow1.png">
        </td>
        <td>
            This section of the flow is variable declaration. The following variable values are populated directly from their respective environment variables:
            <ul>
                <li>vClientId</li>
                <li>vTenantId</li>
                <li>vSiteUrl</li>
                <li>vListName</li>
            </ul>
        </td>
    </tr>
</table>

#### Secret Retrieval
<table>
    <tr>
        <td width="500" height="500">
            <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow2.png">
        </td>
        <td> This section attempts to read the client secret value from the AKV environment variable.<br/><br/>If the Secret (Azure Key Vault) environment variable is not configured or fails to load, the flow will attempt to read the Secret (Plaintext) environment variable.
        </td>
    </tr>
</table>


#### SharePoint List Check
<table>
    <tr>
        <td width="500" height="500">
            <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow3.png">
        </td>
        <td> This section queries the SharePoint site for all lists. The filter array action is filtering the list results, looking for an existing list with a title field matching the value in the vListName variable.If a list is not found, the TRUE condition executes, otherwise the workflow ends.
        </td>
    </tr>
</table>

#### SharePoint List Creation
<table>
    <tr>
        <td width="500" height="500">
            <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow4.png">
        </td>
        <td>
            The TRUE condition executes if no existing list was found with the desired list title. The HTTP call creates a new SharePoint list with no custom fields.
        </td>
    </tr>
</table>

#### SharePoint List Column Creation
<table>
    <tr>
        <td width="500" height="500">
            <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow5.png">
        </td>
        <td>The first HTTP action call queries the list to retrieve all existing columns.<br/><br/>The next two Parse JSON actions are used to parse the JSON into a defined schema for use later.<br/><br/>The Apply to each look is looping through all the custom columns required by the solution.<br/><br/>The Filter Array action is checking if the current custom column exists (by name) in the list of columns retrieved from the list.<br/><br/>If not found (false), the HTTP action adding the column using the JSON schema defined in the vColumnSchema variable.Once all the columns are added, the final HTTP action updates the Title field to no longer be a required field.
        </td>
    </tr>
</table>

### Application Principal Reporting - Import Site Collection Associations
The Application Principal Reporting - Import Site Collection Associations flow should be scheduled to run on a periodic basis.  This flow is responsible for importing the site collection URL associations to each application principal in the list.  If not matching application principal is found in the list, a new entry is added with a “NOT FOUND” title.  App Principals deleted from Entra ID are not automatically deleted from the SharePoint Site, which is likely the primary reason an existing application registration entry is missing.
```
Note: this flow is currently to large to be displayed in the new flow GUI.
```