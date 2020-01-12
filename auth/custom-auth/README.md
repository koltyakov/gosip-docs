---
description: Custom authentication mechanisms
---

# Custom Auth

Gosip allows providing custom authentication mechanisms. For example, you are considering reusing Fluent API helpers and HTTP Client but existing authentication strategies do not feet your environment specifics. Maybe your tenant configured with custom ADFS provider, maybe it's 2FA and there are no alternatives and you need On-Demand auth, or maybe it's something integrated to Azure AAD auth, but it missed in Gosip strategies list? Fortunately, this is not any sort of stopper. All included authentication strategies are a sort of a pluging and it's rather affordable to add a new stratagy on your own.

Let's take a look at any strategy binding:

```go
package main

import (
	"log"
	"os"

	"github.com/koltyakov/gosip"
	strategy "github.com/koltyakov/gosip/auth/saml"
)

func main() {

	authCnfg := &strategy.AuthCnfg{
		SiteURL:  os.Getenv("SPAUTH_SITEURL"),
		Username: os.Getenv("SPAUTH_USERNAME"),
 		Password: os.Getenv("SPAUTH_PASSWORD"),
	}

	client := &gosip.SPClient{AuthCnfg: authCnfg}
	// use client in raw requests or bind it with Fluent API ...

}
```

What we can see? Some strategy is imported into the `strategy` namespace. A strategy has `AuthCnfg` struct with some public properties which are obviously taking place in authentication flow. This struct is then passed to `&gosip.SPClient{AuthCnfg: authCnfg}` and somehow after following binding the requests are authenticated.

For this construction to work `strategy.AuthCnfg` should implement `gosip.AuthCnfg` interface which is:

```go
type AuthCnfg interface {
	ReadConfig(configPath string) error
	WriteConfig(configPath string) error
	GetAuth() (string, error)
	GetSiteURL() string
	GetStrategy() string
	SetAuth(req *http.Request, client *SPClient) error
}
```

Philosophy of the strategies is to have two initiation modes, the first is a strict declaration of the creds and the second one is reading credentials from the config. That config is not necessarily a file on the file system it can be a request to a key vault or OS credential manager, etc.

There should be also `WriteConfig` method, which can be a dummy in case of read-only configs.

As the interface is passed to `gosip.SPClient` struct, Gosip knows nothing about the creds and the context, for that reason `GetSiteURL` method is vital to target requests to a correct root URL.

`GetStrategy` method should return the string alias value of the strategy name if something specific should be happening based on its value.

`GetAuth` method is for token and cookie-based authentications, it can be omitted and return just a blank value, or it can be an actual place for authentication flow happening inside, returning a cached string which is when applied somehow to the requests making them authenticated. In case of custom logic, we'd recommend using GetAuth method and don't forget TTL caching to reduce roundtrips. With a robust external auth client, GetAuth can be dummy minimum \([NTLM example](https://github.com/koltyakov/gosip/blob/915876dc1a96ddb8db5b8ed7c403a92608db3f05/auth/ntlm/helpers.go#L15) shows this approach\).

And finally, `SetAuth`, the method where all the magic happening. `SetAuth` method is a middleware, it receives runtime request and should append authentication stuff. Check these as samples: [NTML's SetAuth](https://github.com/koltyakov/gosip/blob/915876dc1a96ddb8db5b8ed7c403a92608db3f05/auth/ntlm/auth.go#L109), [cookie-based auth](https://github.com/koltyakov/gosip/blob/915876dc1a96ddb8db5b8ed7c403a92608db3f05/auth/saml/auth.go#L94),  [Bearer token-based auth](https://github.com/koltyakov/gosip/blob/915876dc1a96ddb8db5b8ed7c403a92608db3f05/auth/addin/auth.go#L96).

By implementing AuthCnfg struct and gosip.AuthCnfg interface any custom authentication can be added to Gosip. 

{% page-ref page="azure-device-flow.md" %}

{% page-ref page="on-demand-auth.md" %}

{% page-ref page="alternative-ntlm.md" %}

