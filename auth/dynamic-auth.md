---
description: Resolving a strategy dynamically in runtime
---

# Dynamic auth

When you deal with multiple SharePoint environments with different strategies and concidering same code automatically resolving a strategy use the dynamic auth helper.

With the dynamic auth you'd need extending a `private.json` content with `strategy` property, containing a name of a strategy. E.g.:

```json
{
  "strtegy": "addin",
  "siteUrl": "https://contoso.sharepoint.com/sites/site",
  "clientId": "...",
  "clientSecret": "..."
}
```

Available strategy names are: azurecert, azurecreds, device, addin, adfs, fba, ntlm, saml, tmg.

### Usage

```go
package main

import (
	"flag"
	"log"

	"github.com/koltyakov/gosip"
	"github.com/koltyakov/gosip/api"
	"github.com/koltyakov/gosip/auth"
)

func main() {
	config := flag.String("config", "./config/private.json", "Config path")

	flag.Parse()

	authCnfg, err := auth.NewAuthFromFile(*config)
	if err != nil {
		log.Fatalf("unable to get config: %v", err)
	}
	
	// or
	// authCnfg, _ := NewAuthFromFile(strategyName)
	// _ = auth.ParseConfig(byteJsonCreds)

	client := &gosip.SPClient{AuthCnfg: authCnfg}
	sp := api.NewSP(client)

	// ...

}
```

With a custom auth involved it can be extended like [this](https://github.com/koltyakov/cq-source-sharepoint/blob/main/resources/auth/auth.go).
