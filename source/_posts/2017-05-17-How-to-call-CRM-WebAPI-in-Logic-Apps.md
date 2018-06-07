---
title: How to Call CRM WebAPI in Logic Apps
date: 2017-05-17 14:16:25
tags:
- Auzre
- Logic Apps
- CRM WebAPI
---
You probably often use web API provide by Azure/Microsoft in your web applications for customized requirements. Normally, you should follow OAuth 2.0 code authorization flow to acquire the access token for the API in your web application. But in [Logic Apps](https://azure.microsoft.com/en-us/services/logic-apps/), the story is different because all the actions of a work flow are executed in backend.
In this blog, we will discuss how to consume Azure API in Logic Apps. In our experiment, we will use [CRM WebAPI](https://msdn.microsoft.com/en-us/library/mt593051.aspx) as an example.
<!-- more -->

## Part 1: Register an application in Azure AD.
1. Login to Azure Classic Portal https://manage.windowsazure.com/ 
2. Open Active Directory.
   {% asset_img image001.jpg %}
3. Click on Add button.
   {% asset_img image002.jpg %}
4. Select "Add an application my organization is developing" on the popup dialog. 
5. Set the new application properties to:
  a. Name: CRM OAuth2 Demo (you can use what name you want)
  b. Type: Web application and/or web API
6. Click Next button.
   {% asset_img image003.jpg %}
7. Set the following application properties: (Any urls we can set here I think because we can change them later, just make sure the validations can be passed.)
 a. Sign-On URL: http://localhost:1234
 b. APP ID URL: https://alphac25.onmicrosoft.com/crmoauth2demo
 Click Complete button.
 {% asset_img image004.jpg %}
8. The new application titled CRM OAuth2 Demo will be added to the list of applications in the selected Azure Active Directory.
9. Select the new application titled CRM OAuth2 Demo.
{% asset_img image005.jpg %}
10. The Application Configuration Tab is displayed with the Client Id.
{% asset_img image006.jpg %}
11. Configure a client secret for the application.
To generate an access token, we need to configure a client secret. To do this we add a key to the application in Azure AD. Below is a step by step guide to do this.
 a. Following on from the previous section, scroll down to the keys on the applications configuration screen
 b. Create a new key by selecting 2 years in the Select Duration drop down list.
 c. Click Save
 {% asset_img image007.jpg %}
12. Store the generated key (Client Secret) in a safe place because it will not be visible the next time the keys screen is displayed.
 {% asset_img image008.jpg %}
13. Assign Dynamics CRM permissions to the Demo application.
 a. Continuing from the previous section, on the applications configuration scroll down to the permissions to other applications section.
 b. Click on the Add application button
    {% asset_img image009.jpg %}
 c. The permissions to other applications dialog is displayed.
 d. Select Dynamics CRM Online.
 e. Click the Add button.
 f. Click the Complete button (tick in the lower right corner of the dialog).
    {% asset_img image010.jpg %}
 g. Dynamics CRM Online is added to the permissions to other applications list
 h. Check the Access CRM Online as organization user permission from the Delegated Permissions drop down list.
 i. Click save.
    {% asset_img image011.jpg %}

## Part 2: Acquire an access token and use it to call CRM WebAPI
1. Add an action to acquire access token
{% asset_img image012.png %}
Set Headers:
```
Cache-Control: no-cache
Content-Type: application/x-www-form-urlencoded
Set Body:
client_id: Client ID from Azure AD
client_secret: Client ID from Azure AD
username: username of the crm user
password: crm user's password
resource: crm url eg: https://alphac25.crm5.dynamics.com
grant_type: password
```
2. Add an action to Parse the HTTP response.
{% asset_img image013.png %}
3. Add another HTTP action to call CRM WebAPI:
{% asset_img image014.png %}
```
Authorization: Bearer <access_token>
Accept: application/json
OData-Version: 4.0
Cache-Control: no-cache
```
## Test result:
{% asset_img image015.png %}
