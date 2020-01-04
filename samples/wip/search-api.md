---
description: Searching content via SharePoint API
---

# Search API

Search and search API is a huge and complex topic. In this article we're going mostly cover some basics in combination of Gosip plus REST API.

### Basic search call

While executing search call there is no difference what web/site is used as a context, search API returns data due to search query object.

The simplest request  looks this way:

```go
// `res`ult here is a byte array with some helper methods as .Results()
res, err := sp.Search().PostQuery(&api.SearchQuery{
	QueryText: "*",
	RowLimit:  5,
})

if err != nil {
	log.Fatal(err)
}

fmt.Printf("%+v\n", res.Results())
```

### Search query

Search query struct \(`api.SearchQuery`\) contains some, let's say, reasonable amount of options 🙈. 

| Option | Description |
| :--- | :--- |
| QueryText | A string that contains the text for the search query |
| QueryTemplate | A string that contains the text that replaces the query text, as part of a query transform |
| ... | ... |

. 

