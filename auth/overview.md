---
description: "\U0001F510 SharePoint authentication strategies implemented in Gosip"
---

# Overview

## Authentication strategies

Auth strategy should be selected correspondin to your SharePoint environment and its configuration.

Import path `strategy "github.com/koltyakov/gosip/auth/{strategy}"`. Where `/{strategy}` stands for a strategy auth package.

| `/{strategy}` | SPO | On-Prem | Credentials sample\(s\) |
| :--- | :--- | :--- | :--- |
| `/saml` | ✅ | ❌ | [sample](strategies/addin.md#json) |
| `/addin` | ✅ | ❌ | [sample](strategies/addin.md#json) |
| `/ntlm` | ❌ | ✅ | [sample](strategies/ntlm.md#json) |
| `/adfs` | ✅ | ✅ | [spo](strategies/adfs.md#sharepoint-online-configuration), [on-prem](strategies/adfs.md#on-premises-configuration), [on-prem \(wap\)](strategies/adfs.md#on-premises-behing-wap-configuration) |
| `/fba` | ❌ | ✅ | [sample](strategies/fba.md#json) |
| `/tmg` | ❌ | ✅ | [sample](strategies/tmg.md#json) |

JSON and struct representations are different in terms of language notations. So credentials parameters names in `private.json` files and declared as structs initiators vary.

## Secrets encoding

When storing credential in local `private.json` files, which can be handy in local development scenarios, we strongly recommend to encode secrets such as `password` or `clientSecret` using [cpass](../utilits/cpass.md). Cpass converts a secret to an encrypted representation which can only be decrypted on the same machine where it was generated. This minimize incidental leaks, i.e. with git commits.
