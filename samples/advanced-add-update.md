---
description: Advanced creating and updating items
---

# Advanced add/update

In addition to standard OData [add](basic-crud.md#adding-an-item) and [update](basic-crud.md#updating-an-item) items operations REST provides such useful methods as `AddValidateUsingPath` and `ValidateUpdateListItem`. The first is only presented in modern SharePoint, it not only allows adding items right in a sub folder but also operate with form data payloads and control check in process. `ValidateUpdateListItem` is handy for operations requiring [system-like-update](https://www.linkedin.com/pulse/list-items-system-update-options-sharepoint-online-andrew-koltyakov/) logic via pure REST.

In Gosip, `AddValidateUsingPath` and `ValidateUpdateListItem` are represented with `Items().AddValidate()` and `Item.UpdateValidate()` methods correspondingly:

### Add validate

```go
list := sp.Web().GetList("Lists/MyList")

// Method options
options := &api.ValidateAddOptions{
  NewDocumentUpdate: true,
  CheckInComment: "test",
  DecodedPath: "Lists/MyList/subfolder" // is optional
}

// Form data payload
data := map[string]string{
  "Title": "New item",
}

if _, err := list.Items().AddValidate(data, options); err != nil {
	log.Fatal(err)
}
```

As `DecodedPath` option the relative path to folder can be provided. It's optional. The path should be relative to a web without trailing slash in the beginning. Gosip adds web relative URL automatically.

`ValidateAddOptions` are also optional, when no new document update or check-in comment or folder path are ever required, a `nil` value should be passed.

```go
list := sp.Web().GetList("Lists/MyList")

// Form data payload
data := map[string]string{
  "Title": "New item",
}

if _, err := list.Items().AddValidate(data, nil); err != nil {
	log.Fatal(err)
}
```

### Update validate

Using update validate is almost the same:

```go
options := &ValidateUpdateOptions{
  NewDocumentUpdate: true,
  CheckInComment: "test",
}

data := map[string]string{
  "Title": "New item",
}

if _, err := list.Items().GetByID(3).UpdateValidate(data, options); err != nil {
	log.Fatal(err)
}
```

### Form values fingerprints

Form values passed to the methods should stand for an array of `{ FieldName: "", FieldValue: "" }` objects where field value is a string of specific format depending on field's data type.

Gosip simplifies this payload operating with map of strings. In payload, map key should stand for a valid `FieldName`, a value, obviously, is the one mapped to `FieldValue`.

The fingerprints for the data types are following:

| Field data type | Value sample | Comment |
| :--- | :--- | :--- |
| Text \(single line and note\) | "text" |  |
| Number  | "123" |  |
| Yes/No | "1" | "1" - Yes, "2" - No |
| Person or group, single and multiple | \`\[{ "Key": "LoginName", "IsResolved": true }\]\` | "IsResolved" is optional |
| Date time | "6/23/2018 10:15 PM" | for different web locales is different  |
| Date only | "6/23/2018' | for different web locales is different  |
| Choice  \(single\) | "Choice 1" |  |
| Choice  \(multi\) | "Choice 1;\#Choice 2" | ";\#" separated list |
| Hyperlink or picture | "https://go.spflow.com, Gosip" | a description can go after URL and ", " delimiter |
| Lookup \(single\) | "2" | item ID as string |
| Lookup \(multi\) | "1;\#;\#2;\#;\#3;\#" | ";\#" separated list, after each ID goes additional ";\#"  |
| Managed metadata \(single\) | "Department 2\|220a3627-4cd3-453d-ac54-34e71483bb8a;" |  |
| Managed metadata \(multi\) | "Department 2\|220a3627-4cd3-453d-ac54-34e71483bb8a;Department 3\|700a1bc3-3ef6-41ba-8a10-d3054f58db4b;" |  |

