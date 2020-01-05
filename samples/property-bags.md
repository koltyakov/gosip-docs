---
description: Property bags operations
---

# Property Bags

Property bags are a nice way of storing global metadata and settings in SharePoint. Property bags are key-value pairs scoped to the container. In general, any folder can act as a container, also webs have their own property bag storage located in all properties section.

Use-cases can vary depending on how an application uses this key-value storage. For example, PnP Provisioning engine stores applied schema information in property bags. A good thing is that you don't need to provision any additional artifacts to keep some business logic or state variables.

A thing to know that all the props values are strings, dealing with different data types they should be serialized or converted to a string.

### Getting web's all properties

```go
props, err := sp.Web().AllProps().Get()
if err != nil {
	log.Fatal(err)
}

for key, val := range props.Data() {
	fmt.Printf("%s: %s\n", key, val)
}
```

### Getting specific properties

To get a limited subset of properties `.GetProps` helper method can be used:

```go
props, err := sp.Web().AllProps().GetProps([]string{
	"taxonomyhiddenlist",
	"vti_defaultlanguage",
})

if err != nil {
	log.Fatal(err)
}

for key, val := range props {
	fmt.Printf("%s: %s\n", key, val)
}
```

Selecting specific properties didn't work in old versions of SharePoint, however, the method has a fallback to getting all and filtering props on the client.

### Setting property value

```go
if err := sp.Web().AllProps().Set("my-prop", "my value"); err != nil {
	log.Fatal(err)
}
```

Modern SharePoint sites by default have custom scripting disabled mode. When custom scripting is disabled even an admin account will receive "Access denied. You do not have permission to perform this action or access this resource." error message. This is the expected behavior.

{% hint style="warning" %}
Setting props require "Custom Script" be allowed on a site. See [more](https://docs.microsoft.com/en-us/sharepoint/allow-or-prevent-custom-script).
{% endhint %}

{% hint style="success" %}
We recommend [Office 365 CLI](https://pnp.github.io/office365-cli/) to enable custom scripting.
{% endhint %}

```bash
spo site classic set --url https://contoso/sites/site --noScriptSite false
```

### Getting folder property bags

```go
props, err := sp.Web().GetFolder("MyLibrary").Props().Get()
if err != nil {
	log.Fatal(err)
}

for key, val := range props.Data() {
	fmt.Printf("%s: %s\n", key, val)
}
```

### Setting many properties values

```go
err := sp.Web().GetFolder("MyLibrary").Props().
	SetProps(map[string]string{
		"prop01": "value 01",
		"prop02": "value 02",
	})

if err != nil {
	log.Fatal(err)
}
```

### Summary

Property bags are a robust way of storing custom settings and state which requires no additional artifacts. When structuring and consuming correctly they can be a great addition to the application logic.

