---
description: AddIn Only authentication
---

# AddIn Only

{% hint style="danger" %}
SharePoint Add-Ins will stop working for new tenants as of November 1st, 2024 and they will stop working for existing tenants and will be fully retired as of April 2nd, 2026. See [more](https://techcommunity.microsoft.com/t5/microsoft-sharepoint-blog/sharepoint-add-in-retirement-in-microsoft-365/ba-p/3982035).
{% endhint %}

### Struct

```go
type AuthCnfg struct {
  // SPSite or SPWeb URL, which is the context target for the API calls
  SiteURL string `json:"siteUrl"`
  // Client ID obtained when registering the AddIn
  ClientID string `json:"clientId"`
  // Client Secret obtained when registering the AddIn
  ClientSecret string `json:"clientSecret"`
  // Your SharePoint Online tenant ID (optional)
  Realm string `json:"realm"`
}
```

Realm can be left empty or filled in, that will add small performance improvement. The easiest way to find tenant is to open SharePoint Online site collection, click `Site Settings` -> `Site App Permissions`. Taking any random app, the tenant ID (realm) is the GUID part after the `@`.

See more details of [AddIn Configuration and Permissions](configuration.md).

### JSON

`private.json` sample:

```javascript
{
  "siteUrl": "https://contoso.sharepoint.com/sites/test",
  "clientId": "e2763c6d-7ee6-41d6-b15c-dd1f75f90b8f",
  "clientSecret": "OqDSAAuBChzI+uOX0OUhXxiOYo1g6X7mjXCVA9mSF/0="
}
```

### Code sample

```go
package main

import (
	"log"
	// "os"

	"github.com/koltyakov/gosip"
	strategy "github.com/koltyakov/gosip/auth/addin"
)

func main() {
	// authCnfg := &strategy.AuthCnfg{
	// 	SiteURL:      os.Getenv("SPAUTH_SITEURL"),
	// 	ClientID:     os.Getenv("SPAUTH_CLIENTID"),
	// 	ClientSecret: os.Getenv("SPAUTH_CLIENTSECRET"),
	// }
	// or using `private.json` creds source

	authCnfg := &strategy.AuthCnfg{}
	configPath := "./config/private.json"
	if err := authCnfg.ReadConfig(configPath); err != nil {
		log.Fatalf("unable to get config: %v", err)
	}

	client := &gosip.SPClient{AuthCnfg: authCnfg}
	// use client in raw requests or bind it with Fluent API ...
}
```

### Extending client secrets

It's important to know that the legacy AddIn authentication's Client Secrets are issued for a limited time. After expiration, if not managed right way there is a risk to get a service connection aunothorized with the following message:&#x20;

AADSTS7000222: The provided client secret keys for app '\*\*\*' are expired. Visit the Azure portal to create new keys for your app: [https://aka.ms/NewClientSecret,](https://aka.ms/NewClientSecret,) or consider using certificate credentials for added security: [https://aka.ms/certCreds.](https://aka.ms/certCreds.)

To renew an AddIn please follow [https://docs.microsoft.com/ru-ru/sharepoint/dev/sp-add-ins/replace-an-expiring-client-secret-in-a-sharepoint-add-in\
](https://docs.microsoft.com/ru-ru/sharepoint/dev/sp-add-ins/replace-an-expiring-client-secret-in-a-sharepoint-add-in)or, long story short:

```powershell
Install-Module -Name AzureAD
Install-Module MSOnline

Connect-MsolService # provide tenant admin account creds
```

```powershell
$clientId = 'e2763c6d-7ee6-41d6-b15c-dd1f75f90b8f' # replace with your clientId
$bytes = New-Object Byte[] 32
$rand = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rand.GetBytes($bytes)
$rand.Dispose()
$newClientSecret = [System.Convert]::ToBase64String($bytes)
New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Symmetric -Usage Sign -Value $newClientSecret -StartDate (Get-Date) -EndDate (Get-Date).AddYears(1)
New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Symmetric -Usage Verify -Value $newClientSecret -StartDate (Get-Date) -EndDate (Get-Date).AddYears(1)
New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Password -Usage Verify -Value $newClientSecret -StartDate (Get-Date) -EndDate (Get-Date).AddYears(1)
$newClientSecret # outputs new clientSecret
```

### Known issues

AddIn Only auth is considered a legacy, in a production [Azure Cert](../azure-certificate-auth.md) is vendor recommended.

In new subscriptions you can face Grant App Permission disabled. You'll be getting the following error:

```json
{
  "error": "invalid_request",
  "error_description": "Token type is not allowed."
}
```

To enable this feature, connect to SharePoint using Windows PowerShell and then run:

&#x20;`set-spotenant -DisableCustomAppAuthentication $false`.

```powershell
Install-Module -Name Microsoft.Online.SharePoint.PowerShell  
$adminUPN="<the full email address of a SharePoint administrator account, example: jdoe@contosotoycompany.onmicrosoft.com>"  
$orgName="<name of your Office 365 organization, example: contosotoycompany>"  
$userCredential = Get-Credential -UserName $adminUPN -Message "Type the password."  
Connect-SPOService -Url https://$orgName-admin.sharepoint.com -Credential $userCredential  
set-spotenant -DisableCustomAppAuthentication $false  
```
