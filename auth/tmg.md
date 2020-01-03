---
description: Microsoft Forefront Threat Management Gateway
---

# TMG Auth

Currently is legacy but was a popular way of exposing SharePoint into external world back in the days.

### Struct

```go
// AuthCnfg - FBA/TMG auth config structure
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL string `json:"siteUrl"`
    // Username for SharePoint On-Prem,
    // format depends in TMG settings, can include domain or doesn't
    Username string `json:"username"`
    // User password
    Password string `json:"password"`
}
```

### JSON

`private.json` sample:

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test",
  "username": "john.doe",
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
	strategy "github.com/koltyakov/gosip/auth/tmg"
)

func main() {

	// authCnfg := &strategy.AuthCnfg{
	// 	SiteURL:  os.Getenv("SPAUTH_SITEURL"),
	// 	Username: os.Getenv("SPAUTH_USERNAME"),
	// 	Password: os.Getenv("SPAUTH_PASSWORD"),
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

