---
description: 'Create, read, update and delete'
---

# Basic CRUD

CRUD is the most commonly requested type of API consumption.

This example demonstrate basic operations on a list items sample. Hovewer, SharePoint has a variety of nuances even when it comes to just getting items from a list.

### Getting list object

```go
// The recommended way of getting lists is by using their relative URIs
// can be a short form without full web relative URL prefix
list := sp.Web().GetList("Lists/MyList")

// other common but less recommended way of getting a list is
// list := sp.Web().Lists().GetByTitle("My List")
```

### Getting items

```go
// itemsResp is a byte array read from response body
// powered with some processing "batteries" as Data() or Unmarshal() methods
itemsResp, err := list.Items().
	Select("Id,Title").  // OData $select modifier, limit what props are retrieved
	OrderBy("Id", true). // OData $orderby modifier, defines sort order
	Top(10).             // OData $top modifier, limits page size
	Get()                // Finalizes API constructor and sends a response

if err != nil {
	log.Fatal(err)
}

// Data() method is a helper which unmarshals generic structure
// use custom structs and unmarshal for custom fields
for _, item := range itemsResp.Data() {
	itemData := item.Data()
	fmt.Printf("ID: %d, Title: %s\n", itemData.ID, itemData.Title)
}
```

Based on a use case, items can be reveived with alternative methods, as `.GetAll`, `.GetPaged`, `.GetByCAML`, or even lists methods as `.RenderListData`.

### Adding an item

```go
// Payload should be a valid JSON stringified string converted to byte array
// Payload's properties must be a valid OData entity types
itemPayload := []byte(`{
	"Title": "New Item"
}`)

// Constructs and sends add operation request
// itemAddRes is a byte array read from response body with extra methods
itemAddRes, err := list.Items().Add(itemPayload)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Raw response: %s\n", itemAddRes)
fmt.Printf("Added item's ID: %d\n", itemAddRes.Data().ID)
```

Payloads can be constructed in a usual Go way using marshalling struct or string maps to JSON. E.g.:

```go
itemMetadata := &struct{
	Title string `json:"Title"`
}{
	Title: "New Item",
}

itemPayload, _ := json.Marshal(itemMetadata)
```

or:

```go
itemMetadata := map[string]interface{}{
	"Title": "New Item",
}

itemPayload, _ := json.Marshal(itemMetadata)
```

up to your preferences.

### Getting a specific item

```go
itemID := 42 // should specific item ID

itemRes, err := list.Items().GetByID(itemID).Get()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Raw response: %s\n", itemRes)
fmt.Printf("Item metadata: %+v\n", itemRes.Data())
```

### Updating an item

```go
// Payload should be a valid JSON stringified string converted to byte array
// Payload's properties must be a valid OData entity types
itemUpdatePayload := []byte(`{
	"Title": "Updated Title"
}`)

itemID := 42 // should specific item ID

// Constructs and sends update operation request
// itemUpdateRes is a byte array read from response body with extra methods
itemUpdateRes, err := list.Items().GetById(itemID).Update(itemUpdatePayload)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Item is updated, %s\n", itemUpdateRes.Data().Modified)
```

Payloads can be constructed in a usual Go way using marshalling struct or string maps to JSON, same as in add operations.

### Delete an item

Item can be not only deleted but recycled with a further restore operation, which can be provide more safety.

```go
itemID := 42 // should specific item ID

// list.Items().GetByID(itemID).Delete() // or .Recycle()
if _, err := list.Items().GetByID(itemID).Recycle(); err != nil {
	log.Fatal(err)
}
```

