---
description: Email notifications utility
---

# Sending Emails

There is nothing simpler than sending email notifications using SharePoint REST and Gosip Fluent API. However, when building workflow workers or custom subscription services email functionality is vital. And what can be better than embedded OOTB functionality? No extra knowledge of SMTP server and mail credentials, just a usual API call to `_api/SP.Utilities.Utility.SendEmail` utility endpoint.

There are some limitations to this method:

* recipients only from the site
* impossibility to change the sender
* no attachments

but as embedded solution it's OK. And probably sort of workflow notification service should stay in such margins anyways.

### Sending email notification

```go
user, err := sp.Web().SiteUsers().
	GetByEmail("jane.doe@contoso.onmicrosoft.com").Get()

if err != nil {
	log.Fatal(err)
}

if err := sp.Utility().SendEmail(&api.EmailProps{
	Subject: "Say hi to Gosip!",
	Body:    "Text or HTML body here...",
	To:      []string{user.Data().Email},
}); err != nil {
	log.Fatal(err)
}
```

Send email utility is not sophisticated but it works and enough for an embedded functionality.

