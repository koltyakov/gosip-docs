---
description: 'Managing groups, requesting users'
---

# Groups & Users

Site groups and users can be requested via Web's `.SiteGroups()` and `.SiteUsers()` queriable collections correspondingly.

### Getting users sample

```go
users, err := sp.Web().SiteUsers().
	Select("Email,Id").
	Filter("Email ne ''").
	OrderBy("Id", true).
	Get()

if err != nil {
	log.Fatal(err)
}

for _, user := range users.Data() {
	fmt.Printf("%d: %s\n", user.Data().ID, user.Data().Email)
}
```

### Getting groups sample

```go
groups, err := sp.Web().SiteGroups().Get()
if err != nil {
	log.Fatal(err)
}

for _, group := range groups.Data() {
	fmt.Printf("%d: %s\n", user.Data().ID)
}
```

### Ensuring a user

You can't add new user via SharePoint API, but a user who exists in AD/AAD can be added to a site by ensuring him/her by a logon name.

```go
user, err := sp.Web().EnsureUser("jane.doe@contoso.onmicrosoft.com")
if err != nil {
	log.Fatal(err)
}

fmt.Printf("User: %+v\n", user)
```

### Associated groups

We love the "power of defaults". Each web by default has three predefined groups: Owners, Members and Visitors. But their IDs and names are different from web to web. Luckily there is a helper for getting associated groups.

```go
// .Members() and .Owners() correspondingly
group, err := sp.Web().AssociatedGroups().Visitors().Get()
if err != nil {
	log.Fatal(err)
}

fmt.Printf(
  "Visitors group ID %d, Title \"%s\"\n",
  group.Data().ID,
  group.Data().Title,
)
```

### Creating groups

```go
group, err := sp.Web().SiteGroups().Add("My group", nil)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("New group ID: %d\n", group.Data().ID)
```

### Deleting groups

```go
// or .RemoveByID(groupID)
if err := sp.Web().SiteGroups().RemoveByLoginName("My group"); err != nil {
	log.Fatal(err)
}
```

### Adding user to a group

```go
user, err := sp.Web().EnsureUser("jane.doe@contoso.onmicrosoft.com")
if err != nil {
	log.Fatal(err)
}

fmt.Printf("User login: %s\n", user.LoginName)
// User login: i:0#.f|membership|jane.doe@contoso.onmicrosoft.com

visitorGroup := sp.Web().AssociatedGroups().Visitors()
if err := visitorGroup.AddUser(user.LoginName); err != nil {
	log.Fatal(err)
}
```

In group's `.AddUser` method the argument should be full and valid login name including  security provider membership prefix. For instance, while you can ensure Jane using `jane.doe@contoso.onmicrosoft.com` the same as an `.AddUser` method will fail as Jane's login is actually different `i:0#.f|membership|jane.doe@contoso.onmicrosoft.com`.

If you already know UserID but not sure about LoginName `.AddUserByID` helper is at the disposal.

### Removing users from a group

Similarly as with adding users:

```go
user, err := sp.Web().EnsureUser("jane.doe@contoso.onmicrosoft.com")
if err != nil {
	log.Fatal(err)
}

memberGroup := sp.Web().AssociatedGroups().Members()
if err := memberGroup.RemoveUser(user.LoginName); err != nil {
	log.Fatal(err)
}

// or

if err := memberGroup.RemoveUserByID(user.ID); err != nil {
	log.Fatal(err)
}
```

### Managing group owner

```go
memberGroup := sp.Web().AssociatedGroups().Members()
if err := memberGroup.SetAsOwner(user.ID); err != nil {
	log.Fatal(err)
}
```

### Getting group's users

```go
users, err := sp.Web().AssociatedGroups().Visitors().
	Users().Select("Id,Title").Get()

if err != nil {
	log.Fatal(err)
}

for _, user := range users.Data() {
	fmt.Printf("%d: %s\n", user.Data().ID, user.Data().Title)
}
```

### Summary

That's it, most of the common actions with groups and users are covered.

You'd probably will be interested with the connected topics:

{% page-ref page="permissions.md" %}

{% page-ref page="search-api.md" %}

{% page-ref page="user-profiles.md" %}

