---
description: Recycling methods and dealing with recycle bin
---

# Recycle Bin

You can work with recycle bins via REST API similarly as with lists. 

### Getting deleted items

```go
data, err := sp.Site().RecycleBin().
	OrderBy("DeletedDate", false).
	Top(5).
	Get() // site's Recycle Bin
// data, err := sp.Web().RecycleBin().Get() // web's one
if err != nil {
	log.Fatal(err)
}

for _, item := range data.Data() {
	d := item.Data()
	fmt.Println(
		d.ID,
		d.ItemType,
		d.LeafNamePath.DecodedURL,
		d.DeletedByName,
		d.DeletedDate,
	)
}
```

Items in recycle bins are queryable collection, OData modifiers can be applied in a usual way.

Response is strongly typed, helps do not care about unmarshalling the structures. Items in recycle bin contains the following metadata:

```go
type RecycledItem struct {
	AuthorEmail               string
	AuthorName                string
	DeletedByEmail            string
	DeletedByName             string
	DeletedDate               time.Time
	DeletedDateLocalFormatted string
	DirName                   string
	ID                        string
	ItemState                 int
	ItemType                  int
	LeafName                  string
	Size                      int
	Title                     string
	LeafNamePath              *DecodedURL
	DirNamePath               *DecodedURL
}
```

Once you have Item ID \(which is a GUID in case of recycle bin\) you can not resore it.

### Restoring recycled items

```go
data, err := sp.Site().RecycleBin().Top(1).Get()
if err != nil {
	log.Fatal(err)
}

if len(data.Data()) > 0 {
	itemID := data.Data()[0].Data().ID
	if err := sp.Site().RecycleBin().GetByID(itemID).Restore(); err != nil {
		log.Fatal(err)
	}
}
```

