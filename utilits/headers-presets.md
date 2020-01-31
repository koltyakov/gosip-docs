---
description: OData modes headers presets
---

# Headers presets

REST API uses OData modes for controlling response verbosity.

By defining different OData modes \(Verbose, Minimalmetadata, Nometadata\) within the Accept headers SharePoint REST API returns not only data of different details but also data in different forms payload shape-terms. Which can lead to runtime errors.

{% page-ref page="../samples/unmarshaling-responses.md" %}

When dealing with different versions of SharePoint there are few gotchas to remember. In old SharePoint 2013, only `Verbose` mode was allowed by default. This can be amended by installing WCF OData extensions and enabling JSON Light support, however, in our practice, a rare farm admin considering such an update. So it's better stick with a Verbose when it comes to SharePoint 2013.

With SharePoint 2016 and newer, and, obviously, SharePoint Online, we'd recommend `Minimalmetadata` as a default. So the payloads could be a bit more smaller in size and effective overall.

`Nometadata` mode could be tricky, from one hand it's close to `Minimalmetadata` yet doesn't content some vital information such as entity identities and paged collections helpers. We prefer `Minimalmetadata` over `Nometadata` in general.

Gosip provides some presets for headers which could be handy together with `Conf` method.

```go
var client *gosip.SPClient
// ...
sp := api.NewSP(client).Conf(api.HeadersPresets.Minimalmetadata)
// api.HeadersPresets.Verbose
// api.HeadersPresets.Nometadata
```

Along with OData modes, these presets define language header `"Accept-Language": "en-US,en;q=0.9"` which forces English messages in responses if English is installed on a site.

