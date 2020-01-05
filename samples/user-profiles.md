---
description: Dealing with user profiles API
---

# User Profiles

Working with UPS API is simple. UPS API is not something sophisticated as [Search](https://app.gitbook.com/@koltyakov/s/gosip/~/drafts/-LxpsgqFJPaz-EgS5dYZ/samples/search-api) for example and it has just a few methods which can be in demand in a server-side operation.

Don't get me wrong, there are not-covered social features from within user profiles API in Gosip but we rarely have seen demand and scenarios of their usage in SharePoint solutions. Anyways, if some additional feature coverage is required for you please let us know by posting an [issue](https://github.com/koltyakov/gosip/issues).

### Getting profiles

The most in-demand, I would say, feature when it comes to user profiles is getting all profiles. However, UPS API allows only dealing with a single profile, you can't request all of them.

Luckily, this is possible and recommended achieving with the search.

```go
res, err := sp.Search().PostQuery(&api.SearchQuery{
	QueryText: "*",
	SourceID:  "b09a7990-05ea-4af9-81ef-edfab16c4e31",
})

if err != nil {
	log.Fatal(err)
}

fmt.Printf("%+v\n", res.Results())
```

See a bit more [advanced sample](search-api.md#user-profiles-search-sample).

### Getting profile properties

```go
user, err := sp.Web().SiteUsers().
	GetByEmail("jane.doe@contoso.onmicrosoft.com").Get()

if err != nil {
	log.Fatal(err)
}

props, err := sp.Profiles().GetPropertiesFor(user.Data().LoginName)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("%+v\n", props.Data())
```

Gosip strongly types properties response using `.Data()` helper, here is the resulted struct:

```go
type ProfilePropsInto struct {
	AccountName           string
	DirectReports         []string
	DisplayName           string
	Email                 string
	ExtendedManagers      []string
	ExtendedReports       []string
	Peers                 []string
	IsFollowed            bool
	PersonalSiteHostURL   string
	PersonalURL           string
	PictureURL            string
	Title                 string
	UserURL               string
	UserProfileProperties []*TypedKeyValue
}
```

### Getting single property

Sometimes you have to be as effective as possible and trim down responses to a minimum. Let's say you only need a single property.

```go
rop, err := sp.Profiles().
	GetUserProfilePropertyFor(user.Data().LoginName, "AccountName")

if err != nil {
	log.Fatal(err)
}

fmt.Printf("%s\n", prop)
```

Seriously, I don't know the reason for existing of getting a single property, but not getting specific properties or multiple profiles or updating multiple properties. There are no excuses yet this part of SharePoint API is clunky and really old.

### Setting user profile property values

There are two methods for setting user profile property value. Yeah, you heard me the right property in a time.

#### Set single value profile property

```go
if err := sp.Profiles().SetSingleValueProfileProperty(
	user.Data().LoginName,
	"AboutMe",
	"Updated from Gosip",
); err != nil {
	log.Fatal(err)
}
```

#### Set multi valued profile property

```go
tags := []string{"#ci", "#demo", "#test"}
if err := sp.Profiles().SetMultiValuedProfileProperty(
	user.Data().LoginName,
	"SPS-HashTags",
	tags,
); err != nil {
	log.Fatal(err)
}
```

### Summary

User profiles API is limited due to its legacy nature. It is what it is. However, many SharePoint solutions, especially intranet portals and workflow processes can be heavily based on UPS. Go worker can be handy for custom synchronization scenarios and also in external workflow workers with UPS as a source for settings for detecting user dynamic roles.

