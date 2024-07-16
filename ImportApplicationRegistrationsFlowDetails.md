### Application Principal Reporting - Import Application Principals

#### Variable Declaration
<table border=0>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow15.png">
                </kbd>
            </p>
        </td>
        <td>
            This section of the flow is variable declaration. 
            <br/><br/>
            The following variable values are populated directly from their respective environment variables:
            <ul>
                <li>vClientId</li>
                <li>vClientSecret</li>
                <li>vTenantId</li>
                <li>vSiteUrl</li>
                <li>vListName</li>
            </ul>
            <br/>
            The Compose action is a JSON mapping from all the Microsoft Graph and SharePoint Online permissions; including id, display name and scope.  The flow uses this array as a way to easily determine the scope and display name for permissions returned application list returned by Microsoft Graph.  The display name values map directly to the choice field values in the SharePoint list.
            <br/><br/>
            The remaining variables (not shown) are internal arrays used within the flow.
        </td>
    </tr>
</table>

#### Secret Retrieval
<table>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow16.png">
                </kbd>
            </p>
        </td>
        <td>This section attempts to read the client secret value from the AKV environment variable.
            <br/><br/>
            If the Secret (Azure Key Vault) environment variable is not configured or fails to load, the flow will attempt to read the Secret (Plaintext) environment variable.
        </td>
    </tr>
</table>

#### SharePoint List Item Cache
<table>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow17.png">
                </kbd>
            </p>
        </td>
        <td>This section queries for all existing SharePoint list items and builds an array of objects consisting of the (List) ItemId and ApplicationId.  This mapping is used later to determine if the flow needs to update an existing entry or add a new entry.
        </td>
    </tr>
</table>

#### Sync Application Principal to SharePoint List
<table>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow18.png">
                </kbd>
            </p>
        </td>
        <td>
            This section is a “do until” loop that keeps fetching pages of applications from Graph API, until there are no more results. The format of the JSON response is as follows:
            <br/>
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow19.png">
                </kbd>
            </p>
            <br/>
            The flow then iterates through each application result on the current page.
        </td>
    </tr>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow20.png">
                </kbd>
            </p>
        </td>
        <td>
            For each application found, the flow will iterate all objects in the requiredResourceAccess (consented services) array.
            <br/><br/>
            The Switch action is checking if the current consented service, identified by the resourceAppId GUID, is either the SharePoint Online or Microsoft Graph service.  All other services are out of scope for this flow.
        </td>
    </tr>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow21.png">
                </kbd>
            </p>
        </td>
        <td>
            The flow will branch according to the GUID value for resourceAppId.
        </td>
    </tr>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow22.png">
                </kbd>
            </p>
        </td>
        <td>
            If the GUID value for resourceAppId maps to Microsoft Graph, the flow enumerates each object in the resourceAccesss (consented role) array.
            <br/><br/>
            The Filter array action is looking up the role details from the JSON mapping to determine the scope and display name of the application role.
            <br/><br/>
            The Condition action checks if the permission exists to verify that the Microsoft Graph role is one in scope.
            <br/><br/>
            The later Condition action determines if the permission scope is either Delegated or Application scope, adding the permission’s display name to the appropriate permissions array.
        </td>
    </tr>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow23.png">
                </kbd>
            </p>
        </td>
        <td>
            If the GUID value for resourceAppId maps to SharePoint Online, the flow enumerates each object in the resourceAccesss (consented role) array.
            <br/><br/>
            The Filter array action is looking up the role details from the JSON mapping to determine the scope and display name of the application role.
            <br/><br/>
            The Condition action determines if the permission scope is either Delegated or Application scope, adding the permission’s display name to the appropriate permissions array.
        </td>
    </tr>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow24.png">
                </kbd>
            </p>
        </td>
        <td>
            The Condition action checks if any permissions have been added to any of the four permissions arrays.
            <br/><br/>
            If added, the flow looks for an existing entry in the list item cache with a matching ApplicationId. 
            <br/><br/>
            If not found (TRUE), the list item is added (POST), otherwise (FALSE) the existing list item is updated (PATCH).
        </td>
    </tr>

</table>