---
description: User credentials authentication
---

# ADFS Auth

### Struct

```go
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL      string `json:"siteUrl"`
    Username     string `json:"username"`
    Password     string `json:"password"`
    // Following are not required for SPO
    Domain       string `json:"domain"`
    RelyingParty string `json:"relyingParty"`
    AdfsURL      string `json:"adfsUrl"`
    AdfsCookie   string `json:"adfsCookie"`
}
```

See more details [ADFS user credentials authentication](https://github.com/s-kainet/node-sp-auth/wiki/ADFS-user-credentials-authentication).

Gosip's ADFS also supports a scenario of ADFS or NTML behind WAP \(Web Application Proxy\) which adds additional auth flow and `EdgeAccessCookie` involved into play.

### JSON

`private.json` On-Premises sample:

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test",
  "username": "john.doe@contoso.com",
  "password": "this-is-not-a-real-password",
  "relyingParty": "urn:sharepoint:www",
  "adfsUrl": "https://login.contoso.com",
  "adfsCookie": "FedAuth"
}
```

`private.json` On-Premises behind WAP sample:

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test",
  "username": "john.doe@contoso.com",
  "password": "this-is-not-a-real-password",
  "relyingParty": "urn:AppProxy:com",
  "adfsUrl": "https://login.contoso.com",
  "adfsCookie": "EdgeAccessCookie"
}
```

`private.json` SPO sample:

```javascript
{
  "siteUrl": "https://contoso.sharepoint.com/sites/test",
  "username": "john.doe@contoso.onmicrosoft.com",
  "password": "this-is-not-a-real-password"
}
```

