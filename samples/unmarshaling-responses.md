---
description: Parsing complex responses
---

# Unmarshaling responses

Gosip Fluent API tries providing strongly typed responses objects, but in most situations, it's not possible as there are many factors reshaping a response JSON body.

By defining different OData modes \(Verbose, Minimalmetadata, Nometadata\) within the Accept headers SharePoint REST API returns not only data of different details but also data in different forms payload shape-terms. Which can lead to runtime errors.

We're are not forcing to use one specific OData mode only \(e.g. Minimalmetadata\) as this would prevent support of some old SharePoint versions \(2013\) without additional server-side configurations, we have responses normalisation methods which reshape JSONs from the API to the close or identical form no matter the OData mode is.

Let's take a look at the example:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"

	"github.com/koltyakov/gosip"
	"github.com/koltyakov/gosip/api"
	strategy "github.com/koltyakov/gosip/auth/saml"
)

func main() {
	// Binding auth & API client
	configPath := "./config/private.saml.json"
	authCnfg := &strategy.AuthCnfg{}
	if err := authCnfg.ReadConfig(configPath); err != nil {
		log.Fatalf("unable to get config: %v", err)
	}
	client := &gosip.SPClient{AuthCnfg: authCnfg}
	sp := api.NewSP(client)

	// Getting items from a custom list
	list := sp.Web().GetList("Lists/MyList")
	data, err := list.Items().Select("Id,CustomField").Get()
	if err != nil {
		log.Fatalln(err)
	}

	// Define a stuct or map[string]interface{} for unmarshalling
	items := []*struct {
		ID     int    `json:"Id"`
		Custom string `json:"CustomField"`
	}{}

	// .Normalized() method aligns responses between different OData modes
	if err := json.Unmarshal(data.Normalized(), &items); err != nil {
		log.Fatalf("unable to parse the response: %v", err)
	}

	for _, item := range items {
		fmt.Printf("%+v\n", item)
	}

}
```

[_Open on GitHub_](https://github.com/koltyakov/gosip-sandbox/blob/master/samples/unmarshaling/main.go)

Almost any API byte array response has extension methods, such as `.Data()` and `.Normalize()`. The Data method uses predefined generic API entity items unmarshaling, it can be useful for the OOTB entities, e.g. Webs, Groups, Lists, to name just a few. But won't allow getting custom item values for example.

When it comes to custom responses, you have to unmarshal by your own, however, Gosip also helps with this by providing normalization methods.

Let's compare responses in a bit much details:

```javascript
// "Accepts": "application/json;odata=verbose"
{
  "d": {
    "results": [
      {
        "Id": 134,
        "CustomField": "CustomValue",
        "ID": 134,
        // ...
      }
    ]
  }
}

// "Accepts": "application/json;odata=minimalmetadata"
{
  "value": [
    {
      "Id": 134,
      "CustomField": "CustomValue",
      "ID": 134,
      // ...
    }
  ],
  // ...
}

// "Accepts": "application/json;odata=nometadata"
{"value":[{"Id":134,"CustomField":"CustomValue","ID":134}]}
```

As can be seen, there is a great difference in Verbose and Minimal/No metadata modes in shape-terms. The difference is mode dramatic when complex data fields come into play.

After normalization, responses shape is reduced to the following:

```go
[
  {
    "CustomField": "CustomValue",
    "ID": 134,
    "Id": 134,
    // ...
  }
]
```

This makes unmarshalling in times simpler and reduce potential errors after deciding changing OData mode globally.

