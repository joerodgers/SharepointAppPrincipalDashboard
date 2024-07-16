
### Application Principal Reporting - Import Site Collection Associations
The *Application Principal Reporting - Import Site Collection Associations* flow should be scheduled to run on a periodic basis.  This flow is responsible for importing the site collection URL associations to each application principal in the list.  If not matching application principal is found in the list, a new entry is added with a "NOT FOUND" title.  App Principals deleted from Entra ID are not automatically deleted from the SharePoint Site, which is likely the primary reason an existing application registration entry is missing.
```
Note: this flow is currently to large to be displayed in the new flow GUI.
```

#### Variable Declaration
<table border=0>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow6.png">
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
            <br/>The remaining variables are either temporary arrays for storing results or are static values that do not need to be customized.
        </td>
    </tr>
</table>

#### Secret Retrieval
<table>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow7.png">
                </kbd>
            </p>
        </td>
        <td>This section attempts to read the client secret value from the AKV environment variable.
            <br/><br/>
            If the Secret (Azure Key Vault) environment variable is not configured or fails to load, the flow will attempt to read the Secret (Plaintext) environment variable.
        </td>
    </tr>
</table>

#### Site Collection URL Retrieval
<table>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow8.png">
                </kbd>
            </p>
        </td>
        <td>This section paged HTTP calls to the Microsoft Graph GetAllSites endpoint to retrieve a list of all OD/SP sites.
            <br/><br/>
            The Filter Array action is filtering out all sites with /personal/ in the WebUrl value in order to exclude all OD4B sites.
            <br/><br/>
            The Select action is used to generate an array of JSON objects for all sites in the page.
            <br/><br/>
            The Compose action is unioning the current page of JSON objects with the JSON objects of previous pages.
            <br/><br/>
            The Set variable action updates the vSiteUrlsBatchChunks variable with the unioned result.
        </td>
    </tr>
</table>

#### SharePoint List Item Cache
This section is responsible for creating a cache of all existing list items.  This cache of items allows the flow to know if an application registration exits in the list, requiring a list item update, or if an existing item needs to be updated.
<br/><br/>
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow10.png" width="800"></kbd>
</p>

#### Site Collection Enumeration and Permission Checking
This section first chunks all the JSON objects into groups of 20 objects.  The flow then enumerates each chunk, executing an HTTP call to the Microsoft Graph BATCH API, requesting permissions data for each of the 20 JSON objects.
<br/><br/>
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow11.png" width="800"></kbd>
</p>
<br/><br/>
The HTTP: Execute Graph Batch Request action in the above image has a permissions JSON response as follows.
<br/><br/>
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow12.png" width="800"></kbd>
</p>
<br/><br/>
A series of two Filter Array actions will remove all batch results that do not have a 200 repsonse code and batch results with no app registration permissions.  These new filters (v1.0.0.9+) will reduce the number of interations executed in the following Apply to each action.
<br/><br/>
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow13.png" width="800"></kbd>
</p>

#### Sync Application Principal Associations to SharePoint List
After collection all the site collection to application registration mappings in the vSharePointSiteToApplicationIdMappingArray array, it needs to then merge those results to the SharePoint list.
<table>
    <tr>
        <td width="500" height="500">
            <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow14.png">
        </td>
        <td>
            The Select: ApplicationIds action is way to create an array of all ApplicationId values from the mapping array.  The Compose: Unique Applcations then creates an array of all the unique values.
            <br/><br/>
            The flow then enumerates each of those unique values.
            <br/><br/>
            For each unique application id, the flow determines if the application id exists in the list item cache. If found in cache, execute a list item update (PATCH), otherwise execute a list item addition (POST).
        </td>
    </tr>
</table>