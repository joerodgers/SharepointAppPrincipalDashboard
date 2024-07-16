### Application Principal Reporting - Import Sign-In Activity   
The *Application Principal Reporting - Import Sign-In Activity* flow should be scheduled to run on a periodic basis.  This flow is responsible for importing the the last sign-in activity date value for each of the app registrations listed in the SharePoint list.

#### Variable Declaration
<table border=0>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow25.png">
                </kbd>
            </p>
        </td>
        <td>
            This section of the flow is variable declaration. 
            <br/><br/>
            The following variable values are populated directly from their respective environment variables:
            <ul>
                <li>vClientSecret</li>
            </ul>
            <br/>
            The remaining variables are internal variables uesd to define static URLs or are arrays for temporary data storage.
        </td>
    </tr>
</table>

#### Secret Retrieval
<table>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow26.png">
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
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow27.png">
                </kbd>
            </p>
        </td>
        <td>This section queries for all existing SharePoint list items and builds an array of objects consisting of the (List) ItemId and ApplicationId.  This mapping is used later to determine if the flow needs to update an existing entry or add a new entry.
        </td>
    </tr>
</table>

#### Batch Graph Requests
<table>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow28.png">
                </kbd>
            </p>
        </td>
        <td>This section executes batch API calls to Microsoft Graph to fetch the last sign-in activity for 20 (max) app registrations. 
        </td>
    </tr>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow29.png">
                </kbd>
            </p>
        </td>
        <td>The results of all the batch requests are merged into a single array of JSON objects. 
        </td>
    </tr>
</table>

#### SharePoint List Updates 
<table>
    <tr>
        <td width="500" height="500">
            <p align="center" width="100%">
                <kbd>
                    <img align="center" src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/flow30.png">
                </kbd>
            </p>
        </td>
        <td>
            This section enumerates each of the app registrations retrieved from the list in the prior step and updates each with the latest value for the last sign-in activity date.
        </td>
    </tr>
</table>
