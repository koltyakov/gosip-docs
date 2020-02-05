---
description: Using Go context
---

# Context

Gosip client respects native Go [Context](https://golang.org/pkg/context), you can pass a context on a low level or to a Fluent API to control requests's deadlines, cancellation signals, and other request-scoped values across API boundaries and between processes. 

### Low level client

On the [low level](http-client.md#low-level-http-client) dealing with the contexts is identical to native approach:

```go
client := &gosip.SPClient{AuthCnfg: auth}

var req *http.Request
// Initiate API request
// ...

req = req.WithContext(context.Background()) // <- pass a context

resp, err := client.Execute(req)
if err != nil {
  fmt.Printf("Unable to request api: %v", err)
  return
}
```

### HTTP Client

While using HTTPClient, context is defined together with request config.

```go
spClient := api.NewHTTPClient(&gosip.SPClient{AuthCnfg: auth})

endpoint := auth.GetSiteURL() + "/_api/web?$select=Title"

reqConf := &api.RequestConfig{
  Context: context.Background(), // <- pass a context
}

data, err := spClient.Get(endpoint, reqConf)
if err != nil {
  log.Fatalf("%v\n", err)
}

// spClient.Post(endpoint, body, reqConf) // generic POST

// generic DELETE helper crafts "X-Http-Method"="DELETE" header
// spClient.Delete(endpoint, reqConf)

// generic UPDATE helper crafts "X-Http-Method"="MERGE" header
// spClient.Update(endpoint, body, reqConf)

// CSOM helper (client.svc/ProcessQuery)
// spClient.ProcessQuery(endpoint, body, reqConf)
```

### Fluent API

With Fluent API, context is managed in the same way as in the previous example with the only difference how it is chained to fluent syntax.

```go
config := &api.RequestConfig{
  Context: context.Background(), // <- pass a context
}

sp := api.NewSP(client).Conf(config)

data, err := sp.Web().Lists().Select("Id,Title").Get()
if err != nil {
	log.Fatalln(err)
}
```

`Conf` method can be used almost on any hierarchy level. It's inherited with capability to redefine.

