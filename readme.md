## Overview

This solution provides SharePoint Online Administrators with an evergreen SharePoint list detailing all Entra ID App Registrations in a tenant which have with one or more SharePoint Online related site consented permissions.  The solution is packaged as an unmanaged Power Platform Solution package.  It contains four Power Automate Workflows use to inventory Entra ID App Registrations, import SharePoint Online sites (sites.selected) associations and import app registration sign-in activity.

These workflows are highly optimized and have been tested on multiple tenants (inculding multi-geo) with 100k+ site collections and 15k+ (total) App Registrations. 

### Dashboard Details

#### App Registration Details
Once the flows are running and data is being imported, users with access to the target SharePoint list will be able to view the important details of all app registrations in the tenant with at least one SharePoint Online related permission.  The list contains basic details about the app registration; title, object id, client id, name, created date.  For app registrations that have been associated with a SharePoint Online site using the Sites.Selected permission, the *SharePoint Site* column will contain all sites assocated with the app registration.   
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/list-1.png" width="800"></kbd>
</p>

#### Permission Details
The list will all consented SharePoint Online related permissions for both the Delegated and Application scope types for the Microsoft Graph Delegated and SharePoint Online APIs. Additionally, the *Last Sign-In Activity Date*, *Secret Expiration Date*, *Certificate Expiration Date* and the *Internal Notes* fields are also provided. 
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/list-2.png" width="800"></kbd>
</p>

### Power Automate Flow Overview

#### Licening
All of the Power Automate Flows in this package leverage one or more premium connectors, therefore the owner of the solution package must have a Power Automate Premium license assigned to their user account. 

Individual Flow Details:
- [Provision SharePoint List Schema Flow Detail](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/ProvisionSharePointListSchemaFlowDetails.md)
- [Import Site Collection Associations Flow Detail](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/ProvisionSharePointListSchemaFlowDetails.md)
- [Import Application Registrations Flow Detail](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/ImportApplicationRegistrationsFlowDetails.md)
- [Import Application Registrations Last Sign-in Activity Flow Detail](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/ImportApplicationRegistrationsSignInFlowDetails.md)


## Setup
Follow the [setup instructions](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/setup.md) for steps to install and configure the solution package.