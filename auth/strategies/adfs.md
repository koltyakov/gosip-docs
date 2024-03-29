---
description: User credentials authentication
---

# ADFS Auth

### Struct

```go
type AuthCnfg struct {
  // SPSite or SPWeb URL, which is the context target for the API calls
  SiteURL      string `json:"siteUrl"`
  Username     string `json:"username"`
  Password     string `json:"password"`
  // Following are not required for SPO
  Domain       string `json:"domain"`
  RelyingParty string `json:"relyingParty"`
  AdfsURL      string `json:"adfsUrl"`
  AdfsCookie   string `json:"adfsCookie"`
}
```

See more details [ADFS user credentials authentication](https://github.com/s-kainet/node-sp-auth/wiki/ADFS-user-credentials-authentication).

Gosip's ADFS also supports a scenario of ADFS or NTML behind WAP \(Web Application Proxy\) which adds additional auth flow and `EdgeAccessCookie` involved into play.

### JSON

#### On-Premises configuration

`private.json` sample:

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test",
  "username": "john.doe@contoso.com",
  "password": "this-is-not-a-real-password",
  "relyingParty": "urn:sharepoint:www",
  "adfsUrl": "https://login.contoso.com",
  "adfsCookie": "FedAuth"
}
```

#### On-Premises behing WAP configuration

`private.json` sample:

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test",
  "username": "john.doe@contoso.com",
  "password": "this-is-not-a-real-password",
  "relyingParty": "urn:AppProxy:com",
  "adfsUrl": "https://login.contoso.com",
  "adfsCookie": "EdgeAccessCookie"
}
```

#### SharePoint Online configuration

`private.json` sample:

```javascript
{
  "siteUrl": "https://contoso.sharepoint.com/sites/test",
  "username": "john.doe@contoso.onmicrosoft.com",
  "password": "this-is-not-a-real-password"
}
```

### Code sample

```go
package main

import (
	"log"
	// "os"

	"github.com/koltyakov/gosip"
	strategy "github.com/koltyakov/gosip/auth/adfs"
)

func main() {
	// authCnfg := &strategy.AuthCnfg{
	// 	SiteURL:  os.Getenv("SPAUTH_SITEURL"),
	// 	Username: os.Getenv("SPAUTH_USERNAME"),
	// 	Password: os.Getenv("SPAUTH_PASSWORD"),
	//	/* other auth props ...*/
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

