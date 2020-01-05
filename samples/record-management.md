---
description: In place record management helpers
---

# Record Management

In enterprise content management \(ECM\) scenarios, records are an important element within a document lifecycle and a company's compliance policies. Not only documents but actually items.

When declared as a record a document or item can't be later changed or moved without undeclaring. This guarantees integrity.

REST API in terms of managing classic in-place records is limited, you can only retrieve record status from item metadata. Item's `OData__vti_ItemDeclaredRecord` property tells was an item declared as a record and when. The field is a date-time value, empty means item is not a record.

We would not be we if didn't add an extension with allows Gosip to declare, undeclare and declare with a declaration date. To workaround REST's limitation, these functionality is achieved by corresponding CSOM calls. What's great is that Gosip abstracts for you such a nuance providing methods which under-the-hood details are interesting but not necessarily should be known by a library consumer.

It's important to notice that in-place record management requires some [configuration](https://support.office.com/en-us/article/configuring-in-place-records-management-7707a878-780c-4be6-9cb0-9718ecde050a) which is not the topic of this article. So let's observe API methods instead.

### Is item declared as record

```go
list := sp.Web().GetList("Lists/MyList")
item := list.Items().GetByID(1)

isRecord, err := item.Records().IsRecord()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Item is record? %t\n", isRecord)
```

### Is document declared as a record

Declaration as records is more common for documents \(files\), yet it's actually a file's item that keeps record status. And therefore holds API actions.

```go
file := sp.Web().GetFile("MyLibrary/Contract.docx")
item, err := file.GetItem()
if err != nil {
	log.Fatal(err)
}

isRecord, err := item.Records().IsRecord()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Document is record? %t\n", isRecord)
```

### Declare as a record

```go
	if err := item.Records().Declare(); err != nil {
		log.Fatal(err)
	}
```

### Undeclare as a record

```go
if err := item.Records().Undeclare(); err != nil {
	log.Fatal(err)
}
```

### Declare with declaration date

{% hint style="warning" %}
The support of that method is not presented in old SharePoint versions.
{% endhint %}

```go
date, _ := time.Parse(time.RFC3339, "2019-01-01T08:00:00.000Z")
if err := item.Records().DeclareWithDate(date); err != nil {
	log.Fatal(err)
}
```

### Summary

There is no methods for declaring and undeclaring items as records in SharePoint REST API \(yet\) and there won't be ever, IMO, for classic record management, but maybe to the [modern](https://regarding365.com/modern-vs-classic-in-place-records-management-in-sharepoint-8424ebab29f1), however, in Gosip, we mimic some CSOM calls and providing corresponding methods.

