---
description: AddIn Only authentication
---

# AddIn Only

This type of authentication uses AddIn Only policy and OAuth bearer tokens for authenticating HTTP requests.

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

See more details of [AddIn Configuration and Permissions](https://github.com/s-kainet/node-sp-auth/wiki/SharePoint-Online-addin-only-authentication).

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
