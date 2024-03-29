---
description: Azure AD Device Token authentication
---

# Azure Device Flow

This article is the sample showing Gosip custom auth with [AAD Device Token Authorization](https://docs.microsoft.com/en-us/azure/go/azure-sdk-go-authorization#use-device-token-authentication).

> If you want users to sign in interactively, the best way is through device token authentication. This authentication flow passes the user a token to paste into a Microsoft sign-in site, where they then authenticate with an Azure Active Directory (AAD) account. This authentication method supports accounts that have multi-factor authentication enabled, unlike standard username/password authentication.

### Azure App registration

1\. Create or use existing app registration

2\. Make sure that the app is configured to support device flow

* Authentication settings
  * Public client/native (mobile & desktop)
  * Suggested Redirect URIs for public clients (mobile, desktop) - [https://login.microsoftonline.com/common/oauth2/nativeclient](https://login.microsoftonline.com/common/oauth2/nativeclient) - checked
  * Default client type - Yes - for Device code flow, [learn more](https://go.microsoft.com/fwlink/?linkid=2094804)
* App permissions
  * Azure Service Management :: user\_impersonation
  * SharePoint :: based on your application requirements
* etc. based on application needs

### Auth configuration and usage

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/koltyakov/gosip"
	"github.com/koltyakov/gosip/api"
	strategy "github.com/koltyakov/gosip/auth/device"
)

func main() {

	authCnfg := &strategy.AuthCnfg{
		SiteURL:  os.Getenv("SPAUTH_SITEURL"),
		ClientID: os.Getenv("SPAUTH_AAD_CLIENTID"),
		TenantID: os.Getenv("SPAUTH_AAD_TENANTID"),
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

When started the application interacts with user using device login.

```bash
To sign in, use a web browser to open the page https://microsoft.com/devicelogin 
and enter the code CL25ZF5N7 to authenticate.
```

After opening the link, providing device code and authenticating in browser the app is ready for communication with your SharePoint site.

The strategy caches auth token in the context of the AAD ClientID. As a result, you won't see the sign in message. If it's not the desired behavior `.CleanTokenCache()` method can be called to clean the local cache.

Note, that the technique is mostly applicable when user interaction is assumed. Usage of that auth approach in the headless scenarios is not the best as it can lead "stuck" application if no-one expects sign in interaction.
