---
description: >-
  Gosip - SharePoint authentication, HTTP client & fluent API wrapper for Go
  (Golang)
---

# Introduction

![Build Status](https://koltyakov.visualstudio.com/SPNode/_apis/build/status/gosip?branchName=master) [![Go Report Card](https://goreportcard.com/badge/github.com/koltyakov/gosip)](https://goreportcard.com/report/github.com/koltyakov/gosip) [![GoDoc](https://godoc.org/github.com/koltyakov/gosip?status.svg)](https://godoc.org/github.com/koltyakov/gosip) [![License](https://img.shields.io/github/license/koltyakov/gosip.svg)](https://github.com/koltyakov/gosip/blob/master/LICENSE) [![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fkoltyakov%2Fgosip.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fkoltyakov%2Fgosip?ref=badge_shield)

![](.gitbook/assets/gosip.png)

## Main features

* Unattended authentication using different strategies.
* Fluent API syntax for SharePoint object model.
* Simplified API consumption \(REST, CSOM, SOAP\).
* SharePoint-aware embedded features \(retries, header presets, error handling\).

### Supported SharePoint versions:

* SharePoint Online \(SPO\)
* On-Premises \(2019/2016/2013\)

### Authentication strategies:

* SharePoint On-Premises 2019/2016/2013:
  * User credentials \(NTLM\)
  * ADFS user credentials \(ADFS, WAP -&gt; Basic/NTLM, WAP -&gt; ADFS\)
  * Behind a reverse proxy \(Forefront TMG, WAP -&gt; Basic/NTLM, WAP -&gt; ADFS\)
  * Form-based authentication \(FBA\)
* SharePoint Online:
  * SAML based with user credentials
  * Add-In only permissions
  * ADFS user credentials \(automatically detects in SAML strategy\)

## Installation

```bash
go get github.com/koltyakov/gosip
```

## Usage insights

1. Understand SharePoint environment type and authentication strategy.

Let's assume it's, SharePoint Online and Add-In Only permissions. Then `strategy "github.com/koltyakov/gosip/auth/addin"` sub package should be used.

```go
package main

import (
    "github.com/koltyakov/gosip"
    "github.com/koltyakov/gosip/api"
    strategy "github.com/koltyakov/gosip/auth/addin"
)
```

2. Initiate authentication object.

```go
auth := &amp;strategy.AuthCnfg{
    SiteURL:      os.Getenv("SPAUTH_SITEURL"),
    ClientID:     os.Getenv("SPAUTH_CLIENTID"),
    ClientSecret: os.Getenv("SPAUTH_CLIENTSECRET"),
}
```

AuthCnfg's from different strategies contains different options relevant for a specified auth type.

The authentication options can be provided explicitly or can be read from a configuration file \(see [more]()\)\).

```go
configPath := "./config/private.json"
auth := &amp;strategy.AuthCnfg{}

err := auth.ReadConfig(configPath)
if err != nil {
    fmt.Printf("Unable to get config: %v\n", err)
    return
}
```

3. Bind auth client with Fluent API.

```go
client := &amp;gosip.SPClient{AuthCnfg: auth}

sp := api.NewSP(client)

res, err := sp.Web().Select("Title").Get()
if err != nil {
    fmt.Println(err)
}

fmt.Printf("%s\n", res.Data().Title)
```

## Usage samples

### Fluent API client

Provides a simple way of constructing API endpoint calls with IntelliSense and chainable syntax.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"

    "github.com/koltyakov/gosip"
    "github.com/koltyakov/gosip/api"
    strategy "github.com/koltyakov/gosip/auth/addin"
)

func main() {
    // Getting auth params and client
    client, err := getAuthClient()
    if err != nil {
        log.Fatalln(err)
    }

    // Binding SharePoint API
    sp := api.NewSP(client)

    // Custom headers
    headers := map[string]string{
        "Accept": "application/json;odata=minimalmetadata",
        "Accept-Language": "de-DE,de;q=0.9",
    }
    config := &amp;api.RequestConfig{Headers: headers}

    // Chainable request sample
    data, err := sp.Conf(config).Web().Lists().Select("Id,Title").Get()
    if err != nil {
        log.Fatalln(err)
    }

    // Response object unmarshalling
    // (struct depends on OData mode and API method)
    res := &amp;struct {
        Value []struct {
            ID    string `json:"Id"`
            Title string `json:"Title"`
        } `json:"value"`
    }{}

    if err := json.Unmarshal(data, &amp;res); err != nil {
        log.Fatalf("unable to parse the response: %v", err)
    }

    for _, list := range res.Value {
        fmt.Printf("%+v\n", list)
    }

}

func getAuthClient() (*gosip.SPClient, error) {
    configPath := "./config/private.spo-addin.json" // <- file with creds
    auth := &amp;strategy.AuthCnfg{}
    if err := auth.ReadConfig(configPath); err != nil {
        return nil, fmt.Errorf("unable to get config: %v", err)
    }
    return &amp;gosip.SPClient{AuthCnfg: auth}, nil
}
```

### Generic HTTP-client helper

Provides generic GET/POST helpers for REST operations, reducing amount of `http.NewRequest` scaffolded code, can be used for custom or not covered with a Fluent API endpoints.

```go
package main

import (
    "fmt"
    "log"

    "github.com/koltyakov/gosip"
    "github.com/koltyakov/gosip/api"
    strategy "github.com/koltyakov/gosip/auth/ntlm"
)

func main() {
    configPath := "./config/private.ntlm.json"
    auth := &amp;strategy.AuthCnfg{}

    if err := auth.ReadConfig(configPath); err != nil {
        log.Fatalf("unable to get config: %v\n", err)
    }

    sp := api.NewHTTPClient(&amp;gosip.SPClient{AuthCnfg: auth})

    endpoint := auth.GetSiteURL() + "/_api/web?$select=Title"

    data, err := sp.Get(endpoint, nil)
    if err != nil {
        log.Fatalf("%v\n", err)
    }

    // sp.Post(endpoint, []byte(body), nil) // generic POST
    
    // generic DELETE helper crafts "X-Http-Method"="DELETE" header
    // sp.Delete(endpoint, nil)
    
    // generic UPDATE helper crafts "X-Http-Method"="MERGE" header
    // sp.Update(endpoint, nil)
    
    // CSOM helper (client.svc/ProcessQuery)
    // sp.ProcessQuery(endpoint, []byte(body))

    fmt.Printf("response: %s\n", data)
}
```

### Low-level HTTP-client usage

Low-lever SharePoint-aware HTTP client from `github.com/koltyakov/gosip` package for custom or not covered with a Fluent API client endpoints with granular control for HTTP request, response, and http.Client parameters. Used internally but almost never required in a consumer code.

```go
client := &amp;gosip.SPClient{AuthCnfg: auth}

var req *http.Request
// Initiate API request
// ...

resp, err := client.Execute(req)
if err != nil {
    fmt.Printf("Unable to request api: %v", err)
    return
}
```

SPClient has `Execute` method which is a wrapper function injecting SharePoint authentication and ending up calling http.Client's `Do` method.

## Authentication strategies

Auth strategy should be selected correspondin to your SharePoint environment and its configuration.

Import path `strategy "github.com/koltyakov/gosip/auth/{strategy}"`. Where `/{strategy}` stands for a strategy auth package.

| `/{strategy}` | SPO | On-Prem | Credentials sample\(s\) |
| :--- | :--- | :--- | :--- |
| `/saml` | ✅ | ❌ | [sample](config/samples/private.spo-user.json) |
| `/addin` | ✅ | ❌ | [sample](config/samples/private.spo-addin.json) |
| `/ntlm` | ❌ | ✅ | [sample](config/samples/private.onprem-ntlm.json) |
| `/adfs` | ✅ | ✅ | [spo](config/samples/private.spo-adfs.json), [on-prem](config/samples/private.onprem-adfs.json), [on-prem \(wap\)](config/samples/private.onprem-wap.json) |
| `/fba` | ❌ | ✅ | [sample](config/samples/private.onprem-fba.json) |
| `/tmg` | ❌ | ✅ | [sample](config/samples/private.onprem-tmg.json) |

JSON and struct representations are different in terms of language notations. So credentials parameters names in `private.json` files and declared as structs initiators vary.

### SAML Auth \(SharePoint Online user credentials authentication\)

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

### AddIn Only Auth

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

### NTLM Auth \(NTLM handshake\)

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

### ADFS Auth \(user credentials authentication\)

```go
// AuthCnfg - ADFS auth config structure
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

### FBA/TMG Auth \(Form-based authentication\)

FBA - Form-based authentication for SharePoint On-Premises.

TMG - Microsoft Forefront Threat Management Gateway, currently is legacy but was a popular way of exposing SharePoint into external world back in the days.

```go
// AuthCnfg - FBA/TMG auth config structure
type AuthCnfg struct {
    // SPSite or SPWeb URL, which is the context target for the API calls
    SiteURL string `json:"siteUrl"`
    // Username for SharePoint On-Prem,
    // format depends in FBA/TMG settings, can include domain or doesn't
    Username string `json:"username"`
    // User password
    Password string `json:"password"`
}
```

## Secrets encoding

When storing credential in local `private.json` files, which can be handy in local development scenarios, we strongly recommend to encode secrets such as `password` or `clientSecret` using [cpass](cmd/cpass/README.md). Cpass converts a secret to an encrypted representation which can only be decrypted on the same machine where it was generated. This minimize incidental leaks, i.e. with git commits.

## Reference

Many auth flows have been "copied" from [node-sp-auth](https://github.com/s-kainet/node-sp-auth) library \(used as a blueprint\), which we intensively use in Node.js ecosystem for years.

Fluent API and wrapper syntax are inspired by [PnPjs](https://github.com/pnp/pnpjs) which is also the first-class citizen on almost all our Node.js and front-end projects with SharePoint involved.

## License

[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fkoltyakov%2Fgosip.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fkoltyakov%2Fgosip?ref=badge_large)

