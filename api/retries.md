---
description: Requests retries on error statuses
---

# Retries

Gosip HTTP client is preconfigured with retry policies so different kinds of failures can be back off. Current implementation of policies assumes a specific number of retries for a specific response HTTP Status Code. One would only consider retries for "non-ok" status codes and only those which represents server or networking error which has chances for backing off on a next try.

### Defaults

The default policies are:

| Status code | Retries | Description |
| :--- | :--- | :--- |
| 401 | 5 | Unauthorized. A retry might help if apply authentication restores an overdue token or handshake. |
| 429 | 5 | Too many requests throttling error response. In the case of API throttling \(relevant for SharePoint Online\), retrying is also aware of Retry-After header for delay detection. |
| 500 | 1 | Internal Server Error. Rarely can be restored. |
| 503 | 5 | Service Unavailable. Fixes intermittent issues with the service. |

Retries' delay increases with each attempt: 200ms, 400ms, 800ms, 1.6s, 3.2s, and so on using the following progression formula:

```go
time.Duration(100*math.Pow(2, float64(retry))) * time.Millisecond
```

For the responses with `Retry-After` header, retry after value is used in preference as threshold is usually require longer wait until next request permitted.

### Custom policy

A custom policy can be provided on demand in the following way:

```go
var authCnfg gosip.AuthCnfg
// ... auth config initiation is omitted

&gosip.SPClient{
	AuthCnfg: authCnfg,
	RetryPolicies: map[int]int{
		// merged with default policies
		500: 2,  // overwrites default
		503: 10, // overwrites default
		401: 0,  // disables default
	},
}
```

### Disabling retries for a specific request

When a specific request assumes no retries, e.g. resolving a folder which potentially doesn't exist but you know that 500 is returned in case of failure and it's a faster algorithm to try optimistically and check or create on an error only, it can be handy disabling any retries but only scoped to a specific request or series of requests. 

For that purposes `X-Gosip-NoRetry` header is the resolution.

```go
headers := map[string]string{
  "X-Gosip-NoRetry": "true",
}
conf := &RequestConfig{
  Headers: headers,
}
if _, err := sp.Web().GetFolder(folderURI).Conf(conf).Get(); err != nil {
  fmt.Println(err)
}
```

