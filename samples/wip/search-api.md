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
| EnableInterleaving | A Boolean value that specifies whether the result tables that are returned for the result block are mixed with the result tables that are returned for the original query |
| EnableStemming | A Boolean value that specifies whether stemming is enabled |
| TrimDuplicates | A Boolean value that specifies whether duplicate items are removed from the results |
| EnableNicknames | A Boolean value that specifies whether the exact terms in the search query are used to find matches, or if nicknames are used also |
| EnableFQL | A Boolean value that specifies whether the query uses the FAST Query Language (FQL) |
| EnablePhonetic | A Boolean value that specifies whether the phonetic forms of the query terms are used to find matches |
| BypassResultTypes | A Boolean value that specifies whether to perform result type processing for the query |
| ProcessBestBets | A Boolean value that specifies whether to return best bet results for the query. This parameter is used only when EnableQueryRules is set to true, otherwise it is ignored. |
| EnableQueryRules | A Boolean value that specifies whether to enable query rules for the query |
| EnableSorting | A Boolean value that specifies whether to sort search results |
| GenerateBlockRankLog | Specifies whether to return block rank log information in the BlockRankLog property of the interleaved result table. A block rank log contains the textual information on the block score and the documents that were de-duplicated. |
| SourceID | The result source ID to use for executing the search query |
| RankingModelID | The ID of the ranking model to use for the query |
| StartRow | The first row that is included in the search results that are returned. You use this parameter when you want to implement paging for search results. |
| RowLimit | The maximum number of rows overall that are returned in the search results. Compared to RowsPerPage, RowLimit is the maximum number of rows returned overall. |
| RowsPerPage | The maximum number of rows to return per page. Compared to RowLimit, RowsPerPage refers to the maximum number of rows to return per page, and is used primarily when you want to implement paging for search results. |
| SelectProperties | The managed properties to return in the search results |
| Culture | The locale ID (LCID) for the query |
| RefinementFilters | The set of refinement filters used when issuing a refinement query (FQL) |
| Refiners | The set of refiners to return in a search result |
| HiddenConstraints | The additional query terms to append to the query |
| Timeout | The amount of time in milliseconds before the query request times out |
| HitHighlightedProperties | The properties to highlight in the search result summary when the property value matches the search terms entered by the user |
| ClientType | The type of the client that issued the query |
| PersonalizationData | The GUID for the user who submitted the search query |
| ResultsURL | The URL for the search results page |
| QueryTag | Custom tags that identify the query. You can specify multiple query tags |
| ProcessPersonalFavorites | A Boolean value that specifies whether to return personal favorites with the search results |
| QueryTemplatePropertiesURL | The location of the queryparametertemplate.xml file. This file is used to enable anonymous users to make Search REST queries |
| HitHighlightedMultivaluePropertyLimit | The number of properties to show hit highlighting for in the search results |
| EnableOrderingHitHighlightedProperty | A Boolean value that specifies whether the hit highlighted properties can be ordered |
| CollapseSpecification | The managed properties that are used to determine how to collapse individual search results. Results are collapsed into one or a specified number of results if they match any of the individual collapse specifications. In a collapse specification, results are collapsed if their properties match all individual properties in the collapse specification. |
| UIlanguage | The locale identifier (LCID) of the user interface |
| DesiredSnippetLength | The preferred number of characters to display in the hit-highlighted summary generated for a search result |
| MaxSnippetLength | The maximum number of characters to display in the hit-highlighted summary generated for a search result |
| SummaryLength | The number of characters to display in the result summary for a search result |
| SortList | The list of properties by which the search results are ordered |
| Properties | Properties to be used to configure the search query |
| ReorderingRules | Special rules for reordering search results. These rules can specify that documents matching certain conditions are ranked higher or lower in the results. This property applies only when search results are sorted based on rank. |

. 

