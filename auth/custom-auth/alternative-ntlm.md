---
description: NTLM handshake authentication
---

# Alternative NTLM

The default NTLM authentication uses [go-ntlmssp](https://github.com/Azure/go-ntlmssp) - the great package by Azure team. However, we found rare environments where it [fails](https://github.com/koltyakov/gosip/issues/14) for some reason. Fortunately, there is an [alternative](https://github.com/koltyakov/gosip-sandbox/tree/master/strategies/ntlm).

### Usage sample

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/koltyakov/gosip"
	"github.com/koltyakov/gosip/api"
	strategy "github.com/koltyakov/gosip-sandbox/strategies/ntlm"
)

func main() {

	authCnfg := &strategy.AuthCnfg{
		SiteURL:  os.Getenv("SPAUTH_SITEURL"),
		Username: os.Getenv("SPAUTH_USERNAME"),
    Domain:   os.Getenv("SPAUTH_DOMAIN"),
		Password: os.Getenv("SPAUTH_PASSWORD"),
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

Use the alternative only as a fallback method if go-ntlmssp doesn't work and don't forget to post an issue to help making Open Source tools better.

