## Core Idea

Territories are a data-sharing structure that allows you to hide certain records from users based off record-level criteria. When you assign a territory to an Account, the system automatically assigns that territory to all related Contacts. However, when there is more than one territory assigned, the workflow does not repeat (eg; if an Account is assigned Territory1, Territory2 and Territory3, the related Contacts are only assinged Territory1). This script makes sure that ALL territories assigned in an Account gets assigned to ALL related Contacts by getting the territory IDs off the Account, then assigning them in Contacts via Territories API.

## Configuration
### Zoho CRM Connection
In order to call the Zoho CRM API within a Deluge Script, you must first create a connection to it.

#### Connections In Zoho CRM
1. Go to the *Zoho CRM homepage* and click **setup**.
2. Under *Developer Space*, select **Connections**.
3. Click **Add Connection**, then select **Zoho OAuth**.

**Zoho OAuth Connections needed:**
* ZohoCRM.modules.accounts.ALL
* ZohoCRM.modules.ALL
* ZohoCRM.modules.contacts.ALL

## Tutorial

1. Use this API to get details of the territories assigned to the Account.

```
response = invokeurl
[
	url :"https://www.zohoapis.com/crm/v2.1/accounts/" + accountid
	type :GET
	connection:"zohoterritories" //--> change this to whatever the connection name is
];
```

2. Create a ```LIST``` variable and use a ```LOOP``` to add all the territory IDs in the list container.

```
territoryids = List();
response = response.get("data").get(0).get("Territories");
for each  r in response
{
	territoryids.add(r.get("id"));
}
```

3. Get all Related Contacts to the Account
```
account = zoho.crm.getRecordById("Accounts",accountid);
contacts = zoho.crm.getRelatedRecords("Contacts","Accounts",accountid);
```

4. Create a ```LOOP``` to iterate through every Contacts, and another ```LOOP``` inside it to iterate through every ID in the territory ID list. Here, another API call is used to assign the territories to the Contacts.

```
for each  c in contacts
{
	contactid = c.get("id");
  	for each  id in territoryids
	{
		paramap = {"data":{{"Territories":{{"id":id}}}}};
		response2 = invokeurl
		[
			url :"https://www.zohoapis.com/crm/v2.1/Contacts/" + contactid + "/actions/assign_territories"
			type :POST
			parameters:paramap + ""
			connection:"zohoterritories"
		];
		info response2;
	}
}
```
*Note: +"" is added after paramap to convert it to string for the function to work*







