---
description: Advanced scenarios for getting list items
---

# Advanced item requests

There are so many nuances connected with requesting items in a list. By default, it's recommended using OData operations as the most simple, straightforward and RESTful approach.

{% page-ref page="basic-crud.md" %}

But OData doesn't cover all of the needs and you got to switch back but not necessarily backward to [CAML](https://docs.microsoft.com/en-us/sharepoint/dev/schema/query-schema) methods. There a lot of gaps which can force you doing this: single valued MMD fields, group by requests, getting items from a view, to name just a few. Sometimes it's old known bugs or limitations, sometimes a specific functionality which was in the CAML days.

### Get items by CAML

```go
list := sp.Web().GetList("Lists/MyList")

caml := `
	<View>
		<Query>
			<Where>
				<Eq>
					<FieldRef Name='ID' />
					<Value Type='Number'>3</Value>
				</Eq>
			</Where>
		</Query>
	</View>
`

data, err := list.Items().GetByCAML(caml)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("%s\n", data)
```

Of course, CAML query can be slightly bit more complex than just this. 😁

For example, you can request data as in a list view:

```go
list := sp.Web().GetList("Lists/MyList")
viewResp, err := list.Views().DefaultView().Get()
if err != nil {
	log.Fatal(err)
}

data, err := list.Items().GetByCAML(viewResp.Data().ListViewXML)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("%s\n", data)
```

However, when working with CAML there is more powerful methods.

### Render List Data

```go
list := sp.Web().GetList("Lists/MyList")

caml := `
	<View>
		<Query>
			<GroupBy Collapse="TRUE">
				<FieldRef Name="Competed" />
			</GroupBy>
		</Query>
	</View>
`

data, err := list.RenderListData(caml)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("%s\n", data)
```

RenderListData\* methods have completely different response structure. It's good and bad at the same time. The cons are that the results are not compatible and really different from OData methods, not only the shape but also values format. Sometimes it aches. The pros are that render list data methods provide something which is missed in OData responses: group by with collapsed data and only subtotals are possible, recurrent calendar events can be requested \(yet it's now so shiny\), to name a few.

RenderListData deals with GET request, which is a disadvantage as CAML query length is limited. This limitation is rarely can be met in practice, but the ridiculously complex conditions could fail due to this fact.

### Render List Data as Stream

This method is super powerful, SharePoint Modern UI list views work using `.RenderListDataAsStream`. The method is the continuation of evolving of `.RenderListData` enhanced by the vendor for their practical needs of building the modern UI views.

The method deals with POST requests, almost have no length limitation, and operates with many-many options for covering all those aspects and features of the Modern UI view.

{% hint style="info" %}
The method is in our plans to implement in Gosip Fluent API. But you always can craft that API consumption using AdHoc queries and [HTTP Client](../api/http-client.md#post).
{% endhint %}

### Pagination

Pagination in SharePoint lists is painful if you misses couple of moments. In a contrast to many databases queries where `top` and `skip` are straightforward thing to build an pagination, OData's `$skiptoken` is not what many think. 

First of all, it's not a number of rows to skip before starting returning items on a next paged collection.

The simplest format of skip token is: `Paged=TRUE&p_ID=5`

The simplest reverse skip token is: `Paged=TRUE&PagedPrev=TRUE&p_ID=5`

{% hint style="info" %}
Reverse token returns previous page content obviously.
{% endhint %}

By looking at `p_ID` part you'd think that item's ID is enough to construct a correct skip token, but it's not so. It's only correct for not sorted collections. If any `$orderby` modifier is applied, the amount of `p_*` parameters changes. Let's assume you sorted the list by Title, it will add something like `p_Title=Smth`, where "Smth" is the last row's Title value on current page collection.

Fortunately, skip tokens should not be constructed manually. REST return next page collection URI together with the responses for the current page collection.

In Gosip we have helper methods which makes it simpler working with pagination.

```go
list := sp.Web().GetList("Lists/MyList")

page, err := list.Items().Select("Id").Top(100).GetPaged()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Page items %d\n", len(page.Items.Data()))

if page.HasNextPage() {
	nextPage, err := page.GetNextPage()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Next page items %d\n", len(nextPage.Items.Data()))
}
```

{% hint style="warning" %}
We're planning some improvements with pagination interfaces and scale the approach to all possible paged collections APIs but will try to introduce any backward incompatibilities as little as possible.
{% endhint %}

### Requesting large lists

Large lists in SharePoint are those which amount of items is larger than view throttling limitation, the default limitation is 5000 items in a view. In On-Prem this value can be tweaked, in SPO this is a hard limit.

REST API can't return more than 5K items at once. Filter conditions based on indexing fields must be applied to trim down items number. It can be hard, though it's common for server-side processing requesting all items even if there are tens of thousands of items in a list. Such operations are not for immidiate actions but long running syncronizations.

In Gosip, `.GetAll` method is at disposal.

```go
list := sp.Web().GetList("Lists/MyList")

allItems, err := list.Items().Select("Id,Title").Top(5000).GetAll()
if err != nil {
	log.Fatal(err)
}

// process allItems data
```

The method disables any ordering and filtering if applied, as ordering and filtering are not compatible with large lists.

Recommendations for getting all items from a large list:

* Always specify only really required fields to retrieve in `Select`
* Use `Top` equal to 5000 \(as the default Top=100\)
* Consider event-based synchronizations and partial [getting changes](change-api.md)
* Consider [search-based](search-api.md) logic 

