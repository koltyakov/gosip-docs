---
description: Managing roles and objects permissions
---

# Permissions

Permissions management an important part of business logic which can be met in worker processes and workflows.

Let's explore how Gosip Fluent API wraps up SharePoint securable object and permissions.

First of all, by securable objects mean any SharePoint artifacts which can be configured with unique permissions: webs, lists, libraries, items, etc.

Fluent API scopes role operations under `.Roles()` method.

### Checking is an object has unique permissions

```go
item := list.Items().GetByID(1)
hasUniqueAssignments, err := item.Roles().HasUniqueAssignments()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Has unique permissions: %t\n", hasUniqueAssignments)
```

### Breaking role inheritance

To assign unique permissions to an object its role inheritence must be broken.

```go
item := list.Items().GetByID(1)
if err := item.Roles().BreakInheritance(true, false); err != nil {
	log.Fatal(err)
}

// where the first argument stands for `copyRoleAssigments`
//   - if true the permissions are copied from the current parent scope
// second argument is `clearSubScopes`
//  - true to make all child securable objects inherit role assignments 
//  from the current object
```

Be aware that abusing breaking role inheritence and having too many unique permissions can have a detrimental effect on SharePoint performance. The architecture of a solution which involves many unique permissions has too be planned carefully. See some valuable [recommendations](https://docs.microsoft.com/en-us/sharepoint/sites/best-practices-for-using-fine-grained-permissions-in-sharepoint-server).

### Role definitions

Before assigning permissions it's important to know how to get role definitions.

> A role definition is a collection of rights bound to a specific object. Role definitions \(for example, Full Control, Read, Contribute, Design, or Limited Access\) are scoped to the Web site and mean the same thing everywhere within the Web site, but their meanings can differ between sites within the same site collection. Role definitions can also be inherited from the parent Web site, just as permissions can be inherited.

Definitions are csoped as Web level in `.RoleDefinitions()` quariable collection.

```go
def, err := sp.Web().RoleDefinitions().GetByType(api.RoleTypeKinds.Contributor)
if err != nil {
	log.Fatal(err)
}
```

Definitions can be gotten by Definition ID, Name or Type. We recommend using OOTB definitions and getting them by types. There is an enumerator-like helper for default types `api.RoleTypeKinds`.

### Getting principals

Principal is site user or group, their IDs are used in roles assignments. Principal ID can be received by requesting UIL \(User Information List\), getting site user/group by name/email, etc.

```go
roup, err := sp.Web().SiteGroups().GetByName("Process Users").Get()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Principal ID: %d\n", group.Data().ID)

// or

user, err := sp.Web().SiteUsers().GetByEmail("jonh.doe@contoso.com").Get()
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Principal ID: %d\n", user.Data().ID)
```

It's in preference to operate on groups level and grant permissions to a specific users as little as possible.

### Adding role assignments

> The role assignment is the relationship among the role definition, the users and groups, and the scope \(for example, one user may be a reader on list 1, while another user is a reader on list 2\). The relationship expressed through the role assignment is the key to making SharePoint  security management role-based.

At last, now we are ready for roles assigment. 😜 Who told permissions is simple?

```go
if err := item.Roles().AddAssigment(group.Data().ID, def.ID); err != nil {
	log.Fatal(err)
}
```

### Removing role assignments

Removing role assignments is just the same as adding but in opposite. 

```go
if err := item.Roles().RemoveAssigment(group.Data().ID, def.ID); err != nil {
	log.Fatal(err)
}
```

### Reseting roles inheritance

 To reset permissions inheritance:

```go
item := list.Items().GetByID(1)
if err := item.Roles().ResetInheritance(); err != nil {
	log.Fatal(err)
}
```

After reseting object roles assigments its permissions are again inherited from the parent object.

### Summary

Now we know how to treat SharePoint permissions using Gosip and Fluent API. Before wrapping up, we want to stress on importance of careful planning of permissions model, as less unique permissions is better.

