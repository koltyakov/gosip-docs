---
description: No-auth mode
---

# Anonymous

It's not an auth strategy but a mode without any authentication flow applied to the SPClient.

Anonymous mode can be handy in a situation when Gosip SharePoint-aware helpers intended to be used however authentication is handled by any other middleware.

### Struct

Only `SiteURL` is required.

```go
type AuthCnfg struct {
	// SPSite or SPWeb URL, which is the context target for the API calls
	SiteURL string `json:"siteUrl"`
}
```

### JSON

`private.json` sample:

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test"
}
```

### Code sample

```go
package main

import (
	"log"
	// "os"

	"github.com/koltyakov/gosip"
	strategy "github.com/koltyakov/gosip/auth/anon"
)

func main() {

	// authCnfg := &strategy.AuthCnfg{
	// 	SiteURL:  os.Getenv("SPAUTH_SITEURL"),
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

