---
description: Configuring authentication and API client
---

# Library Initiation

## Binding authentication and API client

### Understand SharePoint environment type and authentication strategy

Based on authentication provider supported and configured in your SharePoint Farm environment different library [authentication strategies](../auth/strategies/) might be or not be applicable.

If you have no idea which strategy is used within your farm the question is better be addressed to SharePoint admins.

Let's assume it's SharePoint Online and Add-In Only permissions. Then `strategy "github.com/koltyakov/gosip/auth/addin"` sub package should be used.

```go
package main

import (
  "github.com/koltyakov/gosip"
  "github.com/koltyakov/gosip/api"
  strategy "github.com/koltyakov/gosip/auth/addin"
)
```

It could have been SharePoint On-Premise and NTLM and `strategy "github.com/koltyakov/gosip/auth/ntlm"` as imported strategy.

### Initiate authentication object

Different authentication strategies assumes different credential parameters. Sometimes it can be Username/Password, sometimes CertPath or ClientID/ClientSecret. Please refer a specific strategy documentation for relevalt for the auth type  parameters.

Credential can be passed directly in AuthCnfg's struct. Here a sample for `addin`:

```go
auth := &strategy.AuthCnfg{
  SiteURL:      os.Getenv("SPAUTH_SITEURL"),
  ClientID:     os.Getenv("SPAUTH_CLIENTID"),
  ClientSecret: os.Getenv("SPAUTH_CLIENTSECRET"),
}
```

An anternative is using a configuration file \(see [more](../auth/overview.md#authentication-strategies)\). Which is a JSON containing the same parameters as used with AuthCnfg's struct.

```go
configPath := "./config/private.json"
auth := &strategy.AuthCnfg{}

err := auth.ReadConfig(configPath)
if err != nil {
  fmt.Printf("Unable to get config: %v\n", err)
  return
}
```

### Bind auth client with Fluent API

Now when auth is bound it should be passed to client and Fluent API instance: 

```go
client := &gosip.SPClient{AuthCnfg: auth}

sp := api.NewSP(client)
```

Most of the samples starts with `sp` assuming configuration described above is already in place.

```go
res, err := sp.Web().Select("Title").Get()
if err != nil {
  fmt.Println(err)
}

fmt.Printf("%s\n", res.Data().Title)
```

