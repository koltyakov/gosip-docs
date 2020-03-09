---
description: 'Getting changes, synchronisation scenarios'
---

# Change API

When it comes to synchronization or delta processing Change API is for the rescue. There is no need in any sort of logic that used modified date comparison to detect what has been changed since the last processing time. There is no need to wait until the search crawl is done to check changes through web or site collection. And not only these benefits but also tracking permissions changes restores from recycle bin, system updates and much more.

Changes API is what is used together with Webhooks \(in SPO\), by having a context \(scope\) and change token\(s\) you can easily retrieve what has been changed and run the custom logic based on the nature of changes.

Changes can be requested from within different scopes: lists, webs, sites, etc. Change API operates change tokens to establish start and end anchors.

Change tokens received from a specific entity type \(e.g. Site\) can't be used with another entity type \(e.g. List\) while sending change query.

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

for _, change := range changes.Data() {
  fmt.Printf("%+v\n", change)
}
```

Change results are strongly typed with Gosip's API, the `changes` variable in the sample is an array of `api.ChangeInfo` struct pointers.

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

`ChangeType` is an important piece of information, it's an enumerator describing a nature of a change. See [more](https://docs.microsoft.com/en-us/previous-versions/office/sharepoint-csom/ee543793%28v%3Doffice.15%29).

{% hint style="info" %}
Change API doesn't return what specifically was changed but only where it was changed. For instance, when an item is changed the API will return its identities information but not the metadata.
{% endhint %}

In a synchronization scenario or a Webhook after getting information which items are changed the corresponding request\(s\) should be sent for getting specifics.

### Change query

When getting the changes the API requires some clarifications about what exactly should be retrieved. This is defined within a change query which is implemented `api.ChangeQuery` struct.

| Property | Description |
| :--- | :--- |
| ChangeTokenStart | Specifies the start date and start time for changes that are returned through the query |
| ChangeTokenEnd | Specifies the end date and end time for changes that are returned through the query |
| Add | Specifies whether add changes are included in the query |
| Alert | Specifies whether changes to alerts are included in the query |
| ContentType | Specifies whether changes to content types are included in the query |
| DeleteObject | Specifies whether deleted objects are included in the query |
| Field | Specifies whether changes to fields are included in the query |
| File | Specifies whether changes to files are included in the query |
| Folder | Specifies whether changes to folders are included in the query |
| Group | Specifies whether changes to groups are included in the query |
| GroupMembershipAdd | Specifies whether adding users to groups is included in the query |
| GroupMembershipDelete | Specifies whether deleting users from the groups is included in the query |
| Item | Specifies whether general changes to list items are included in the query |
| List | Specifies whether changes to lists are included in the query |
| Move | Specifies whether move changes are included in the query |
| Navigation | Specifies whether changes to the navigation structure of a site collection are included in the query |
| Rename | Specifies whether renaming changes are included in the query |
| Restore | Specifies whether restoring items from the recycle bin or from backups is included in the query |
| RoleAssignmentAdd | Specifies whether adding role assignments is included in the query |
| RoleAssignmentDelete | Specifies whether adding role assignments is included in the query |
| RoleDefinitionAdd | Specifies whether adding role assignments is included in the query |
| RoleDefinitionDelete | Specifies whether adding role assignments is included in the query |
| RoleDefinitionUpdate | Specifies whether adding role assignments is included in the query |
| SecurityPolicy | Specifies whether modifications to security policies are included in the query |
| Site | Specifies whether changes to site collections are included in the query |
| SystemUpdate | Specifies whether updates made using the item SystemUpdate method are included in the query |
| Update | Specifies whether update changes are included in the query |
| User | Specifies whether changes to users are included in the query |
| View | Specifies whether changes to views are included in the query |
| Web | Specifies whether changes to Web sites are included in the query |

### Pagination

Sometimes it can be lots of changes since a provided token. Gosip implements pagination helper using "jumping" between different change tokens under the hood. When last change item's token is used as a start token to get the "next page". This approach is used in `GetNextPage`:

```go
changesFirstPage, _ := list.Changes().Top(100).GetChanges(&ChangeQuery{
	ChangeTokenStart: token,
	List:             true,
	Item:             true,
	Add:              true,
})

changesSecondPage, _ := changesFirstPage.GetNextPage()
```

### Summary

Change API is a powerful mechanism and a robust way for processing delta changes that can and should be used in advanced and optimised synchronizations and business processes withing external workers.

