---
description: Operations for features management
---

# Feature management

You can activate and deactivate features on sites and webs using REST API. However, there are a few nuances to know knowing which makes it simple to manage the features.

{% hint style="info" %}
You should know specific feature definition ID to add or remove it.
{% endhint %}

Unfortunately, there is no way of getting features list with names and descriptions programmatically. When getting features list on the web or site you receive a list of activated features definition IDs and nothing else.

### Getting activated features

```go
res, err := sp.Web().Features().Get()
if err != nil {
	log.Fatal(err)
}

for _, f := range res {
	fmt.Printf("%+v\n", f)
}

// &{DefinitionID:b77b6484-364e-4356-8c72-1bb55b81c6b3}
// &{DefinitionID:a7a2793e-67cd-4dc1-9fd0-43f61581207a}
// &{DefinitionID:d5a4ed08-27b9-4142-9804-45dec6fda126}
// &{DefinitionID:780ac353-eaf8-4ac2-8c47-536d93c03fd6}
// &{DefinitionID:8c6f9096-388d-4eed-96ff-698b3ec46fc4}
// ...
```

Luckily, nowadays one almost always deal only with OOTB features, if not probably something is terribly wrong on a project. 😝

### Feature activating

```go
mds := "87294c72-f260-42f3-a41b-981a2ffce37a"
if err := sp.Web().Features().Add(mds, true); err != nil {
	log.Fatal(err)
}
```

The first argument stands for Feature Definition ID, the second one is force mode.

### Feature deactivating

```go
mds := "87294c72-f260-42f3-a41b-981a2ffce37a"
if err := sp.Web().Features().Remove(mds, true); err != nil {
	log.Fatal(err)
}
```

Arguments are identical.

When a feature was not activated before removing it an error message is expected \("Feature is not activated at this scope"\).

Site level actions are absolutely identical, the only difference is `sp.Site()` entry point. 

