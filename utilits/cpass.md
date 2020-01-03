---
description: "\U0001F510 Simple secure string password convertor"
---

# Cpass

Cpass is simplified secured password two-ways encryption sub package [port](https://github.com/koltyakov/cpass) for Gosip.

By default, Cpass uses Machine ID as an encryption key so a secret hash can only be decrypted on a machine where it was generated.

Cpass's approach is appropriate in local development scenarios. The main goal is "not to show raw secret while presenting a desktop" or "not to commit raw secret by an incident to code source".

### Installation

```bash
go get github.com/koltyakov/gosip/cpass
```

### Convertor

```go
package main

import (
	"flag"
	"fmt"

	"github.com/koltyakov/gosip/cpass"
)

func main() {

	var rawSecret string

	flag.StringVar(&rawSecret, "secret", "", "Raw secret string")
	flag.Parse()

	crypt := cpass.Cpass()

	secret, _ := crypt.Encode(rawSecret)
	fmt.Println(secret)

}
```

### Encrypt secrets

```bash
go run ./ -secret "MyP@s$word"
#> -lywbAGD4iPYdJXDxLAQoMUbfBXBIQR2UZYl
```

When use result token/hash as a secret in `private.json` file\(s\).

