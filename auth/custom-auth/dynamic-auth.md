---
description: Resolving a strategy dynamically in runtime
---

# Dynamic auth

Currently, Gosip requires choosing a specific strategy or strategies to import and use within the application. However, some applications or utility tools might need more. There is no official universal resolver \(yet it's planned\), but in an application, some dynamic can be added on demand.

The sample shows a simple way of importing potentially demanded strategies and selecting one in runtime based on logic, CLI flags in the case of the sample.

### Implementation sample

{% hint style="success" %}
Check out [code source](https://github.com/koltyakov/gosip-sandbox/blob/master/samples/dynauth/dynauth.go)
{% endhint %}

### Usage

```go
package main

import (
	"flag"
	"log"

	"github.com/koltyakov/gosip"
	"github.com/koltyakov/gosip/api"
	"github.com/koltyakov/gosip-sandbox/samples/dynauth"
)

func main() {

	strategy := flag.String("strategy", "ondemand", "Auth strategy")
	config := flag.String("config", "./config/private.json", "Config path")

	flag.Parse()

	authCnfg, err := dynauth.NewAuthCnfg(*strategy, *config)
	if err != nil {
		log.Fatalf("unable to get config: %v", err)
	}

	client := &gosip.SPClient{AuthCnfg: authCnfg}
	sp := api.NewSP(client)

	// ...

}
```

When the application should be started with corresponding flags and configuration files on the file system. The sample shows the idea and has a field for improvements, e.g. prompting for credentials, providing site URL within the arguments, using plugins for strategies, and so on.

We plan adding strategies resolver and dynamic environment detection in the future.

