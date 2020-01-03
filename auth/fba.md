---
description: Form-based authentication for SharePoint On-Premises
---

# FBA Auth

### Struct

```go
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL string `json:"siteUrl"`
    // Username for SharePoint On-Prem,
    // format depends in FBA settings, can include domain or doesn't
    Username string `json:"username"`
    // User password
    Password string `json:"password"`
}
```

### JSON

`private.json` sample:

```javascript
{
  "siteUrl": "https://www.contoso.com/sites/test",
  "username": "john.doe",
  "password": "this-is-not-a-real-password"
}
```

