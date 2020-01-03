---
description: SharePoint Online user credentials authentication
---

# SAML Auth

This authentication option uses Microsoft Online Security Token Service `https://login.microsoftonline.com/extSTS.srf` and SAML tokens in order to obtain authentication cookie.

```go
// AuthCnfg - SAML auth config structure
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL string `json:"siteUrl"`
    // Username for SharePoint Online, e.g. `[user]@[company].onmicrosoft.com`
    Username string `json:"username"`
    // User or App password
    Password string `json:"password"`
}
```
