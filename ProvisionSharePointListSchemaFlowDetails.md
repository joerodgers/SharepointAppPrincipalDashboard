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
        <td> This section attempts to read the client secret value from the AKV environment variable.<br/><br/> If the Secret (Azure Key Vault) environment variable is not configured or fails to load, the flow will attempt to read the Secret (Plaintext) environment variable.
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