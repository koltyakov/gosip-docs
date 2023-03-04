---
description: Azure AD authorization with username and password
---

# Azure Creds Auth

This article is the sample showing Gosip custom auth with [AAD Username/Password Authorization](https://docs.microsoft.com/en-us/azure/developer/go/azure-sdk-authorization).

### Azure App registration

{% hint style="info" %}
Follow the [steps](azure-environment-auth.md#azure-app-registration)
{% endhint %}

### JSON

`private.json` sample:

```javascript
{
	"siteUrl": "https://contoso.sharepoint.com/sites/test",
	"tenantId": "e4d43069-8ecb-49c4-8178-5bec83c53e9d",
	"clientId": "628cc712-c9a4-48f0-a059-af64bdbb4be5",
	"username": "user@contoso.com",
	"password": "password"
}
```

### Usage sample

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/koltyakov/gosip"
	"github.com/koltyakov/gosip/api"
	strategy "github.com/koltyakov/gosip/auth/azurecreds"
)

func main() {

	// authCnfg := &strategy.AuthCnfg{
	// 	SiteURL:  os.Getenv("SPAUTH_SITEURL"),
	// 	TenantID: os.Getenv("AZURE_TENANT_ID"),
	// 	ClientID: os.Getenv("AZURE_CLIENT_ID"),
	// 	Username: os.Getenv("AZURE_USERNAME"),
	// 	Password: os.Getenv("AZURE_PASSWORD"),
	// }
	// or using `private.json` creds source

	authCnfg := &strategy.AuthCnfg{}
	configPath := "./config/private.json"
	if err := authCnfg.ReadConfig(configPath); err != nil {
		log.Fatalf("unable to get config: %v", err)
	}
	
	client := &gosip.SPClient{AuthCnfg: authCnfg}
	sp := api.NewSP(client)

	res, err := sp.Web().Select("Title").Get()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Site title: %s\n", res.Data().Title)

}
```

