---
description: AddIn Only Authentication
---

# AddIn Only Auth

This type of authentication uses AddIn Only policy and OAuth bearer tokens for authenticating HTTP requests.

```go
// AuthCnfg - AddIn Only auth config structure
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL string `json:"siteUrl"`
    // Client ID obtained when registering the AddIn
    ClientID string `json:"clientId"`
    // Client Secret obtained when registering the AddIn
    ClientSecret string `json:"clientSecret"`
    // Your SharePoint Online tenant ID (optional)
    Realm string `json:"realm"`
}
```

Realm can be left empty or filled in, that will add small performance improvement. The easiest way to find tenant is to open SharePoint Online site collection, click `Site Settings` -&gt; `Site App Permissions`. Taking any random app, the tenant ID \(realm\) is the GUID part after the `@`.

See more details of [AddIn Configuration and Permissions](https://github.com/s-kainet/node-sp-auth/wiki/SharePoint-Online-addin-only-authentication).

