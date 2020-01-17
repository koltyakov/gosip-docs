---
description: Uploading files in chunks
---

# Chunk upload

For large documents it's better using files Chunk API which allows splitting upload into separate REST API requests reducing memory consumption and providing a more manageable upload mechanism.

Files Chunk API appeared in SharePoint 2016 and should not be expected to work in the previous version.

{% hint style="warning" %}
Chunk upload can't be used in SharePoint 2013
{% endhint %}

### Basic chunk upload example

```go
file, err := os.Open("/path/to/large/file.zip")
if err != nil {
	log.Fatalf("unable to read a file: %v\n", err)
}
defer file.Close()

fileAddResp, err := foler.Files().AddChunked("My File.zip", file, nil)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("New file URL: %s\n", fileAddResp.Data().ServerRelativeURL)
```

### Upload flow

Fluent API abstracts some aspects of the chunked upload. Internally, `/StartUpload`, `/ContinueUpload`, `/CancelUpload`, and `/FinishUpload` endpoints are used. However, these are not exposed for the simplicity. Nonetheless, there is a way for canceling upload with the help of  `api.AddChunkedOptions`.

### Add chunked options

There are a few options which are optional. The third method parameter is a configuration structure.

```go
type AddChunkedOptions struct {
	Overwrite bool // should overwrite existing file
	// on progress callback, execute custom logic on each chunk
	// if the Progress is used it should return "true" to continue upload
	// otherwise upload is canceled
	Progress  func(data *FileUploadProgressData) bool
	ChunkSize int // chunk size in bytes
}
```

Defaults are: `Overwrite` is true and `ChunkSize` equals to 10485760 bytes.

`Progress` callback allows not only trigger a progress logic, for example upload percentage update, but also to cancel an upload. To cancel an upload the progress callback should return `false`.

