---
description: Download & upload files from/to SharePoint is simple
---

# Documents

SharePoint is ECM \(Enterprise Content Management\) system and it's common to expect files being uploaded, downloaded, migrated, processes, and managed in a variety ways.

Gosip provides an easy way of dealing with SharePoint document listaries, files and folders.

### Getting library object

Document library in SharePoint is almost the same as a List, but with intention of being a container for files.

```go
// The recommended way of getting lists is by using their relative URIs
// can be a short form without full web relative URL prefix
list := sp.Web().GetList("MyLibrary")

// other common but less recommended way of getting a list is
// list := sp.Web().Lists().GetByTitle("My Library")
```

### Getting folder object

```go
foler := sp.Web().GetFolder("MyLibrary/Folder01")
```

### Getting file object

```go
file := sp.Web().GetFile("MyLibrary/Folder01/File01.txt")
```

### Adding new folder

```go
// folderResp is a byte array read from response body with extra methods
folderResp, err := foler.Folders().Add("New Folder Name")
if err != nil {
	log.Fatal(err)
}

fmt.Printf("New folder URL: %s\n", folderResp.Data().ServerRelativeURL)
```

### Deleting folders

```go
folderRelativeURL := "MyLibrary/Folder01/New Folder Name"
if _, err := sp.Web().GetFolder(folderRelativeURL).Delete(); err != nil {
	log.Fatal(err)
}
```

### Adding/uploading a file

```go
	// fileAddResp is a byte array read from response body with extra methods
	fileAddResp, err := foler.Files().
		Add("My File.txt", []byte("File content"), true)
	
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("New file URL: %s\n", fileAddResp.Data().ServerRelativeURL)
```

Obviously, file content can be a result of reading a file from disk, e.g.:

```go
content, err := ioutil.ReadFile("/path/to/file.txt")
if err != nil {
	log.Fatal(err)
}

fileAddResp, err := foler.Files().Add("My File.txt", content, true)
if err != nil {
	log.Fatal(err)
}
```

For the large files it's better using AddChunked API, hovewer, it was not available in SharePoint 2013.

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

[More details](chunk-upload.md).

### Downloading files

```go
fileRelativeURL := "MyLibrary/Folder01/File01.txt"
data, err := sp.Web().GetFile(fileRelativeURL).Download()
if err != nil {
	log.Fatal(err)
}

file, err := os.Create("/path/toLocal/file.txt")
if err != nil {
	log.Fatalf("unable to create a file: %v\n", err)
}
defer file.Close()

_, err = file.Write(data)
if err != nil {
	log.Fatalf("unable to write to file: %v\n", err)
}

file.Sync()
```

For a large files it's better getting file reader thought:

```go
fileRelativeURL := "MyLibrary/Folder01/File01.txt"
fileReader, err := sp.Web().GetFile(fileRelativeURL).GetReader()
if err != nil {
	log.Fatal(err)
}
defer fileReader.Close()

file, err := os.Create("/path/toLocal/file.txt")
if err != nil {
	log.Fatalf("unable to create a file: %v\n", err)
}
defer file.Close()

if _, err := io.Copy(file, fileReader); err != nil {
	log.Fatalf("unable to save a file: %v\n", err)
}
```

### Summary

Using Gosip you can concentrate on business logic and Go language aspects while processing documents actions in SharePoint seemlessly.

With use of IntelliSense and Fluent syntax other supported actions can be consumed based on a specific requirement.

