---
description: 'Getting changes, synchronisation scenarios'
---

# Change API

When it comes to synchronisation or delta processing Change API is for the rescue. There is no need in any sort of the logic which used modified date comparison to detect what has been changed since the last processing time. There is no need to wait until search crawl is done to check changes through web or site collection. And not only these benifits but also tracking permissions changes, restores from recycle bin, system updates and much more.

Changes API is what is used together with Webhooks \(in SPO\), by having a context \(scope\) and change token\(s\) you can easily retrieve what has been changed and run the custom logic based on the nature of changes.

Changes can be requested from within different scopes: lists, webs, sites, etc.

Change API operates change tokens to establish start and end anchors. Change tokens received from a specific entity type \(e.g. Site\) can't be used with another entity type \(e.g. List\) while sending  change query.

### Getting current change token

```go
siteChangeToken, err := sp.Site().Changes().GetCurrentToken()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Site change token: %s\n", siteChangeToken)

webChangeToken, err := sp.Web().Changes().GetCurrentToken()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Web change token: %s\n", webChangeToken)

list := sp.Web().GetList("Lists/MyList")
listChangeToken, err := list.Changes().GetCurrentToken()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("List change token: %s\n", listChangeToken)

// Site change token:
//   1;1;afbc6f5a-65b3-4b64-9f40-885d8d772c8c;637138201430100000;64696075
// Web change token:
//   1;2;7e084f68-c401-4910-b75e-4347c7848965;637138201430100000;64696075
// List change token:
//   1;3;3b542696-5863-4ff7-bb90-8224e9e8adb9;637138201430100000;64696075
```

### Getting changes

Knowing what root entities' token\(s\) you have the specific changes query can be crafted and sent, e.g.:

```go
list := sp.Web().GetList("Lists/MyList")
listChangeToken, _ := list.Changes().GetCurrentToken()
list.Items().Add([]byte(`{"Title":"New item"}`))
// error handling is omitted -- adding a dummy item to receive changes result

changes, err := list.Changes().GetChanges(&api.ChangeQuery{
	ChangeTokenStart: listChangeToken,
	List:             true,
	Item:             true,
	Add:              true,
})
if err != nil {
	log.Fatal(err)
}

for _, change := range changes {
	fmt.Printf("%+v\n", change)
}
```

Change results are strongly typed with Gosip's API, `changes` variable in the sample is an array of `api.ChangeInfo` struct pointers.

Change into struct contains the following properties:

```go
type ChangeInfo struct {
	ChangeToken       *StringValue
	ChangeType        int
	Editor            string
	EditorEmailHint   string
	ItemID            int
	ListID            string
	ServerRelativeURL string
	SharedByUser      string
	SharedWithUsers   string
	SiteID            string
	Time              time.Time
	UniqueID          string
	WebID             string
}
```

`ChangeType` is an important piece of information, it's an enumerator destribing a nature of a change. See [more](https://docs.microsoft.com/en-us/previous-versions/office/sharepoint-csom/ee543793%28v%3Doffice.15%29).

{% hint style="info" %}
Change API doesn't return what specifically was changed but only where it was changed. For instance, when an item is changed the API will return its identities information but not the metadata.
{% endhint %}

### Change query

When getting the changes the API requires some clarifications about what exactly should be retrieved. This is defined within a change query which is implemente `api.ChangeQuery` struct.

| Property | Description |
| :--- | :--- |
| ... | ... |

