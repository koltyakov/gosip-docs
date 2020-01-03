# Overview

## Authentication strategies

Auth strategy should be selected correspondin to your SharePoint environment and its configuration.

Import path `strategy "github.com/koltyakov/gosip/auth/{strategy}"`. Where `/{strategy}` stands for a strategy auth package.

| `/{strategy}` | SPO | On-Prem | Credentials sample\(s\) |
| :--- | :--- | :--- | :--- |
| `/saml` | ✅ | ❌ | [sample](config/samples/private.spo-user.json) |
| `/addin` | ✅ | ❌ | [sample](config/samples/private.spo-addin.json) |
| `/ntlm` | ❌ | ✅ | [sample](config/samples/private.onprem-ntlm.json) |
| `/adfs` | ✅ | ✅ | [spo](config/samples/private.spo-adfs.json), [on-prem](config/samples/private.onprem-adfs.json), [on-prem \(wap\)](config/samples/private.onprem-wap.json) |
| `/fba` | ❌ | ✅ | [sample](config/samples/private.onprem-fba.json) |
| `/tmg` | ❌ | ✅ | [sample](config/samples/private.onprem-tmg.json) |

JSON and struct representations are different in terms of language notations. So credentials parameters names in `private.json` files and declared as structs initiators vary.

## Secrets encoding

When storing credential in local `private.json` files, which can be handy in local development scenarios, we strongly recommend to encode secrets such as `password` or `clientSecret` using [cpass](cmd/cpass/README.md). Cpass converts a secret to an encrypted representation which can only be decrypted on the same machine where it was generated. This minimize incidental leaks, i.e. with git commits.

