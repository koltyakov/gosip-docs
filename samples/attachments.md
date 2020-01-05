---
description: Dealing with items attachments
---

# Attachments

Our common recommendation with attachments is to use them in moderation with a preference to documents in libraries and linking business objects items to that document using metadata and other logical relationship. But sometimes you need nothing more than just a simple binary addition to an item.

Working with attachments is mostly straightforward as you can only get a list of item's attachment, get a specific attachment by its name, add and delete an attachment.

### Getting attachments

```go
list := sp.Web().GetList("Lists/MyList")
item := list.Items().GetByID(1)

attachments, err := item.Attachments().Get()
if err != nil {
	log.Fatal(err)
}

for _, attachment := range attachments.Data() {
	data := attachment.Data()
	fmt.Printf("%s (%s)\n", data.FileName, data.ServerRelativeURL)
}
```

Attachments API provides little information, actually only FileName and ServerRelativeURL.

### Items have attachments

To detect which items have attachments the corresponding `Attachments` property can be requested within an ordinary get items request:

```go
items, err := list.Items().Select("Id,Attachments").Get()
if err != nil {
	log.Fatal(err)
}

for _, item := range items.Data() {
	data := item.Data()
	hasAttachments := "has no"
	if data.Attachments {
		hasAttachments = "has"
	}
	fmt.Printf("Item ID %d %s attachments\n", data.ID, hasAttachments)
}
```

### Adding attachments

Adding attachments is almost identical to [adding documents](documents.md#adding-uploading-a-file) to a document library.

```go
list := sp.Web().GetList("Lists/MyList")
item := list.Items().GetByID(1)

content := []byte("Get content in a usual Go way you like")

item.Attachments().GetByName("MyAttachment.txt").Delete()

resp, err := item.Attachments().Add("MyAttachment.txt", content)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Attachment is added: %s\n", resp.Data().ServerRelativeURL)
```

### Getting attachment by name

```go
list := sp.Web().GetList("Lists/MyList")
item := list.Items().GetByID(1)

content, err := item.Attachments().GetByName("MyAttachment.txt").Download()
if err != nil {
	log.Fatal(err)
}

fmt.Printf(
	"Do whatever needed with this content of %d bytes\n",
	len(content),
)
```

### Attachment actions

With an attachment you can:

* Download
* Get reader \(download in a stream way\)
* Delete
* or Recycle

These actions are rather obvious with the help of the Fluent API.

