---
description: "\U0001F528 Provides low-level communication with any SharePoint API"
---

# HTTP Client

Gosip HTTP client for SharePoint allows consuming any HTTP resource or API, it can even bypass SharePoint site pages \(with all assets\) within a dev toolchain \([proxy, dev server](https://github.com/koltyakov/gosip/tree/master/cmd/samples/proxy)\).

Gosip HTTP client is SharePoint nuances-aware, it takes care under the hood of such things as headers, API calls retries, threshholds, error handling, POST API requests Digests, and, of course, authentication and its renewal.

However, dealing at low-level, means you should know SharePoint API rather well and you're ok with the verbosity of Go. If you just starting with SharePoint please consider [Fluent API](fluent-api/) client first and HTTP client for none covered methods and custom stuff.

### Usage

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
    auth := &strategy.AuthCnfg{}
    configPath := "./config/private.json"
    if err := auth.ReadConfig(configPath); err != nil {
        log.Fatalf("unable to get config: %v\n", err)
    }

    spClient := api.NewHTTPClient(&gosip.SPClient{AuthCnfg: auth})

    endpoint := auth.GetSiteURL() + "/_api/web?$select=Title"

    data, err := spClient.Get(endpoint, nil)
    if err != nil {
        log.Fatalf("%v\n", err)
    }

    // spClient.Post(endpoint, []byte(body), nil) // generic POST
    
    // generic DELETE helper crafts "X-Http-Method"="DELETE" header
    // spClient.Delete(endpoint, nil)
    
    // generic UPDATE helper crafts "X-Http-Method"="MERGE" header
    // spClient.Update(endpoint, nil)
    
    // CSOM helper (client.svc/ProcessQuery)
    // spClient.ProcessQuery(endpoint, []byte(body))

    fmt.Printf("response: %s\n", data)
}
```

### Methods

HTTP Client methods covers those which used in SharePoint all the way.

In a contract to default Go `http.Client`'s `.Do` method, Gosip HTTP Client's methods proceed and read response body to return array of bytes. Which reduce amount of scaffolded code a bit and more handy for the REST API consumption, in our opinion, which is 90% of use cases.

#### Get

Sends `GET` request, embeds `"Accept": "application/json;odata=verbose"` header as a default. Use it for REST API `GET` calls.

#### Post

Sends `POST` request, embeds `"Accept": "application/json;odata=verbose"` and `"Content-Type": "application/json;odata=verbose;charset=utf-8"` headers as default. X-RequestDigest is received, cached, and embed automattically. Use it for REST API `POST` calls.

#### Delete

Sends `POST` request, embeds `"Accept": "application/json;odata=verbose"`, `"Content-Type": "application/json;odata=verbose;charset=utf-8"`, and `"X-Http-Method": "DELETE"` headers as default. X-RequestDigest is received, cached, and embed automattically. Use it for REST API `POST` calls with delete resource intention.

#### Update

Sends `POST` request, embeds `"Accept": "application/json;odata=verbose"`, `"Content-Type": "application/json;odata=verbose;charset=utf-8"`, and `"X-Http-Method": "MERGE"` headers as default. X-RequestDigest is received, cached, and embed automattically. Use it for REST API `POST` calls with update resource intention.

### ProcessQuery

Sends `POST` request to `/_vti_bin/client.svc/ProcessQuery` endpoint \(CSOM\). All required headers for a CSOM request are embed automatically. Method's body should stand for a valid CSOM XML package. The response body is parsed for error handling, yet returned in it's original form.

### Low-level HTTP client

Sometimes more control is required, e.g. when downloading files or large responses you could prefer precessing a response in chunks. This can be achieved with the low-level usage of Gosip HTTP client.

```go
client := &gosip.SPClient{AuthCnfg: auth}

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

So you can dive down to native `*http.Request` at this point it's just a standard request from "net/http" package, but authenticated to SharePoint and with some batteries under the hood for a seemles API integration.

