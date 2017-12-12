# AzureADFunctionAuthentication
Azure functions are awesome, and Azure AD is also awesome. Connecting the two is very easy, however invoking that function with an application identity (i.e. Service Principal) takes a few steps. The following outlines that process.

1) Create a new Azure Function. For this walk through I will start with the default HTTPTrigger function in C#.

![C# Http Function](https://github.com/swgriffith/AzureADFunctionAuthentication/raw/master/images/basicfunction.PNG)

2) Click on the Function App (i.e. the level above the function), go to 'Platform Features' and then select 'Authentication/Authorization'.

![Platform Features](https://github.com/swgriffith/AzureADFunctionAuthentication/raw/master/images/platformFeatures.PNG)

3) Select 'On' for 'App Service Authentication'
4) Select 'Log in with Azure Active Directory' for 'Action to take when request is not authenticated'.
5) Select the 'Azure AD' Authentication Provider
6) For 'Managed Mode' select 'Express' and then leave all the other defaulst (i.e. 'Create New AD App', Graph and Grant Common Data permissions can be left turned off). Make note of the App Name you choose. You'll need this later.

![AAD Setup](https://github.com/swgriffith/AzureADFunctionAuthentication/raw/master/images/AADSetup.PNG)

7) Click 'Ok' on the AAD setup screen and then 'Save' on the Authentication/Authorization screen.

8) Once the processing has completed on the save, exit the Authentication/Authorization settings and then go back in, this time selecting 'Advanced' for 'Management Mode'. You should see the details of your new Azure AD App populated.

9) Copy the 'Client ID' and paste it in the 'ALLOWED TOKEN AUDIENCES' section, preceeded with 'spn:'. Click 'Ok' and then 'Save'.

![AAD Setup](https://github.com/swgriffith/AzureADFunctionAuthentication/raw/master/images/AADAdvancedSetup.PNG)

10) At this point the server side setup is complete. Go to Azure AD to start the client side setup.

11) Create a new Application Registration. Make note of the 'Application ID'.

12) Navigate into the new Application and select 'Keys'. Create a new key, making note of the 'Secret' after saving.

13) Navigate to 'Permissions' and click 'Add'. Click 'API' and search for the Azure AD App name you noted in step 6. Select the app, click ok, and then under the 'Select Permissions' section, choose 'Access <AppName>'.

![AAD Setup](https://github.com/swgriffith/AzureADFunctionAuthentication/raw/master/images/AADGrantAppPermission.PNG)


14) Click Select, and then Done, and Grant Permissions (Need to test if Grant Permissions is really needed)

At this point your Azure Setup is Complete! Now lets test. For testing I use Postman, but you can use whatever mechanism you prefer.

15) Create a new POST to the following URL with the x-www-form-urlencoded data shown below:
https://login.microsoftonline.com/<YourDomain>.onmicrosoft.com/oauth2/token?api-version=1.0

|Param |Value|
|-|-|
|client_id|Application ID from step 11|
|client_secret|Key from step 12|
|grant_type|client_credentials|
|resource|Application ID from your Function App (i.e. Client ID from step 9)|

![Postman Get Token](https://github.com/swgriffith/AzureADFunctionAuthentication/raw/master/images/PostmanGetToken.PNG)


```
# Or via curl
curl -X POST \
  'https://login.microsoftonline.com/<AADDomain>.onmicrosoft.com/oauth2/token?api-version=1.0' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
    -d 'client_id=<client_id>&client_secret=<client_secret>&grant_type=client_credentials&resource=<resource>'
```

16) Click 'Send'. You should get a reponse including a token of type 'Bearer'. Copy the value of the 'access token' in the response.

17) Create a new Postman request, and under 'Authorization'/'Type' select 'Bearer Token' and paste the access token from the prior step in the 'token' field.

18) Go back to your function in Azure, get your function URL and past it in the Postman URL dialog and test. You should now be authenticating using your bearer token to the Azure Function and getting back your result.

![Postman Success](https://github.com/swgriffith/AzureADFunctionAuthentication/raw/master/images/finale.PNG)

```
# Or via curl
curl -X GET \
  '<YourFunctionURL>' \
  -H 'Authorization: Bearer <access token>' \
  -H 'Cache-Control: no-cache' 
```





