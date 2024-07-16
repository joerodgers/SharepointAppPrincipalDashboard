## Overview
### Solution Package Overview
The solution package is comprised of three Power Automate Flows, six Solution Environment Variables and one Connection Reference.  The Connection Reference is only used if you chose to the Azure Key Vault secret storage option.


![App Registation](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/list-1.png)

![App Registation](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/list-2.png)
## Setup

### Entra Id Application Registration
This solution package relies entirely on a single Entra ID application registration for authentication to the Microsoft Graph.  All authentication is performed using an app-only context with a Client Id and Secret. To operate in a least privileged configuration, the associated app registration requires the following permissions:

| Application Permission | Justification |
|------------------------|---------------|
|Application.Read.All    | Required to read all applications from Entra ID using Microsoft Graph’s applications endpoint. |
|Sites.Read.All          | Required to read all site collections from Microsoft Graph’s GetAllSites endpoint. |
|Sites.Selected          | “Manage” permissions to the “host” site collection to allow list provisioning, list item operations. |

Example reference configuration:
![Entra Permssions](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/entra-perms.png)

### Site Collection Permissions

### Solution Environment Variable Configuration
After the solution has been successfully imported, select the Application Principal Reporting solution.  Configure the following environment variables:

Environment Variable	Description
Tenant Id	Microsoft 365 Tenant Id
Client Id	Application Registration Client/App Id
Secret (Azure Key Vault)	All flows check for the existence of the AKV secret value first. If the AKV secret is not configured or fails, the flows fall back to the reading the Secret (Plaintext) environment variable.

Do not configure this variable if you plan to only use the Secret (Plaintext) environment variable for secret storage.

Secret (Plaintext)	This value is a fallback value if the Secret (Azure Key Vault) variable is not configured.
Site URL	URL of the SharePoint site hosting the list/dashboard
List Name	Default value is “SharePoint Application Principals”
  
 

 
