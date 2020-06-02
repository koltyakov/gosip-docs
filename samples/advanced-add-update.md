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

<table>
  <thead>
    <tr>
      <th style="text-align:left">Field data type</th>
      <th style="text-align:left">Value sample</th>
      <th style="text-align:left">Comment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Text (single line and note)</td>
      <td style="text-align:left">&quot;text&quot;</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Number</td>
      <td style="text-align:left">&quot;123&quot;</td>
      <td style="text-align:left">as a string</td>
    </tr>
    <tr>
      <td style="text-align:left">Yes/No</td>
      <td style="text-align:left">&quot;1&quot;</td>
      <td style="text-align:left">&quot;1&quot; - Yes, &quot;2&quot; - No</td>
    </tr>
    <tr>
      <td style="text-align:left">Person or group, single and multiple</td>
      <td style="text-align:left">`[{ &quot;Key&quot;: &quot;LoginName&quot;, &quot;IsResolved&quot;: true
        }]`</td>
      <td style="text-align:left">
        <p>&quot;LoginName&quot; is a valid login name, including provider prefix</p>
        <p>&quot;IsResolved&quot; is optional</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Date time</td>
      <td style="text-align:left">&quot;6/23/2018 10:15 PM&quot;</td>
      <td style="text-align:left">for different web locales is different</td>
    </tr>
    <tr>
      <td style="text-align:left">Date only</td>
      <td style="text-align:left">&quot;6/23/2018&apos;</td>
      <td style="text-align:left">for different web locales is different</td>
    </tr>
    <tr>
      <td style="text-align:left">Choice (single)</td>
      <td style="text-align:left">&quot;Choice 1&quot;</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Choice (multi)</td>
      <td style="text-align:left">&quot;Choice 1;#Choice 2&quot;</td>
      <td style="text-align:left">&quot;;#&quot; separated list</td>
    </tr>
    <tr>
      <td style="text-align:left">Hyperlink or picture</td>
      <td style="text-align:left">&quot;https://go.spflow.com, Gosip&quot;</td>
      <td style="text-align:left">a description can go after URL and &quot;, &quot; delimiter</td>
    </tr>
    <tr>
      <td style="text-align:left">Lookup (single)</td>
      <td style="text-align:left">&quot;2&quot;</td>
      <td style="text-align:left">item ID as string</td>
    </tr>
    <tr>
      <td style="text-align:left">Lookup (multi)</td>
      <td style="text-align:left">&quot;1;#;#2;#;#3;#&quot;</td>
      <td style="text-align:left">&quot;;#&quot; separated list, after each ID goes additional &quot;;#&quot;</td>
    </tr>
    <tr>
      <td style="text-align:left">Managed metadata (single)</td>
      <td style="text-align:left">&quot;Department 2|220a3627-4cd3-453d-ac54-34e71483bb8a;&quot;</td>
      <td
      style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">Managed metadata (multi)</td>
      <td style="text-align:left">&quot;Department 2|220a3627-4cd3-453d-ac54-34e71483bb8a;Department 3|700a1bc3-3ef6-41ba-8a10-d3054f58db4b;&quot;</td>
      <td
      style="text-align:left"></td>
    </tr>
  </tbody>
</table>

