---
description: Microsoft Forefront Threat Management Gateway
---

# TMG Auth

Currently is legacy but was a popular way of exposing SharePoint into external world back in the days.

```go
// AuthCnfg - FBA/TMG auth config structure
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL string `json:"siteUrl"`
    // Username for SharePoint On-Prem,
    // format depends in TMG settings, can include domain or doesn't
    Username string `json:"username"`
    // User password
    Password string `json:"password"`
}
```

