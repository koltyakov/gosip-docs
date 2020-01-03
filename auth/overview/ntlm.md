---
description: NTLM handshake authentication
---

# NTLM Auth

This type of authentication uses HTTP NTLM handshake in order to obtain authentication header.

### Struct

```go
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL  string `json:"siteUrl"`
    Domain   string `json:"domain"`   // AD domain name
    Username string `json:"username"` // AD user name
    Password string `json:"password"` // AD user password
}
```

Gosip uses `github.com/Azure/go-ntlmssp` NTLM negotiator, however a custom one also can be [provided](https://github.com/koltyakov/gosip/issues/14) in case of demand.

### JSON

`private.json` sample:

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test",
  "username": "contoso\\john.doe",
  "password": "this-is-not-a-real-password"
}
```

or

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test",
  "username": "john.doe",
  "domain": "contoso",
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
	strategy "github.com/koltyakov/gosip/auth/ntlm"
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

