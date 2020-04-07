---
description: Request events handlers
---

# Hooks

Gosip provides an events system with a set of handlers that can be optionally defined to track different client communication aspects such as request tracking, retries and response error logging, to name just a few.

To define the handlers `Hooks` object should be configured and passed to `gosip.SPClient` struct.

```go
var authCnfg gosip.AuthCnfg
// ... auth config initiation is omitted

&gosip.SPClient{
  AuthCnfg: authCnfg,
  Hooks:    &gosip.HookHandlers{
    // handlers function definition
  },
}
```

The following handlers are available at the moment:

```go
// HookHandlers struct to configure events handlers
type HookHandlers struct {
	OnError    func(event *HookEvent) // when error appeared
	OnRetry    func(event *HookEvent) // before retry request
	OnRequest  func(event *HookEvent) // before request is sent
	OnResponse func(event *HookEvent) // after response is received
}
```

All of the handlers are optional. 

A handler receives `HookEvent` pointer which contains request pointer, response status code, and error \(if applicable for an event\), and time information to track duration since a request started an event happened.

Hooks sample:

```go
// Define requests hook handlers
client.Hooks = &gosip.HookHandlers{
	OnError: func(e *gosip.HookEvent) {
		fmt.Println("\n======= On Error ========")
		fmt.Printf(" URL: %s\n", e.Request.URL)
		fmt.Printf(" StatusCode: %d\n", e.StatusCode)
		fmt.Printf(" Error: %s\n", e.Error)
		fmt.Printf("  took %f seconds\n",
		  time.Since(e.StartedAt).Seconds())
		fmt.Printf("=========================\n\n")
	},
	OnRetry: func(e *gosip.HookEvent) {
		fmt.Println("\n======= On Retry ========")
		fmt.Printf(" URL: %s\n", e.Request.URL)
		fmt.Printf(" StatusCode: %d\n", e.StatusCode)
		fmt.Printf(" Error: %s\n", e.Error)
		fmt.Printf("  took %f seconds\n",
		  time.Since(e.StartedAt).Seconds())
		fmt.Printf("=========================\n\n")
	},
	OnRequest: func(e *gosip.HookEvent) {
		if e.Error == nil {
			fmt.Println("\n====== On Request =======")
			fmt.Printf(" URL: %s\n", e.Request.URL)
			fmt.Printf("  auth injection took %f seconds\n",
			  time.Since(e.StartedAt).Seconds())
			fmt.Printf("=========================\n\n")
		}
	},
	OnResponse: func(e *gosip.HookEvent) {
		if e.Error == nil {
			fmt.Println("\n====== On Response =======")
			fmt.Printf(" URL: %s\n", e.Request.URL)
			fmt.Printf(" StatusCode: %d\n", e.StatusCode)
			fmt.Printf("  took %f seconds\n",
			  time.Since(e.StartedAt).Seconds())
			fmt.Printf("==========================\n\n")
		}
	},
}
```

Hooks can be handy for global logging streaming and metrics collection.

It is recommended using asynchronous and only lightweight logic inside hooks.

