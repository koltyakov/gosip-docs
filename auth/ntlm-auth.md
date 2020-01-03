---
description: NTLM handshake
---

# NTLM Auth

This type of authentication uses HTTP NTLM handshake in order to obtain authentication header.

```go
// AuthCnfg - NTML auth config structure
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL  string `json:"siteUrl"`
    Domain   string `json:"domain"`   // AD domain name
    Username string `json:"username"` // AD user name
    Password string `json:"password"` // AD user password
}
```

Gosip uses `github.com/Azure/go-ntlmssp` NTLM negotiator, however a custom one also can be [provided](https://github.com/koltyakov/gosip/issues/14) in case of demand.
