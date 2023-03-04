---
description: 🔐 SharePoint authentication strategies implemented in Gosip
---

# Overview

### Authentication strategies

Auth strategy should be selected corresponding to your SharePoint environment and its configuration.

Import path `strategy "github.com/koltyakov/gosip/auth/{strategy}"`. Where `/{strategy}` stands for a strategy auth package.

| `/azurecert`  | ✅ | ❌ | [sample](strategies/azure-certificate-auth.md#json)                                                                                                                                          |
| ------------- | - | - | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/azurecreds` | ✅ | ❌ | [sample](strategies/azure-creds-auth.md#json)                                                                                                                                                |
| `/azureenv`   | ✅ | ❌ | [sample](strategies/azure-environment-auth.md)                                                                                                                                               |
| `/device`     | ✅ | ❌ | [sample](strategies/azure-device-flow.md#auth-configuration-and-usage)                                                                                                                       |
| `/saml`       | ✅ | ❌ | [sample](strategies/addin.md#json)                                                                                                                                                           |
| `/addin`      | ✅ | ❌ | [sample](strategies/addin.md#json)                                                                                                                                                           |
| `/ntlm`       | ❌ | ✅ | [sample](strategies/ntlm.md#json)                                                                                                                                                            |
| `/adfs`       | ✅ | ✅ | [spo](strategies/adfs.md#sharepoint-online-configuration), [on-prem](strategies/adfs.md#on-premises-configuration), [on-prem (wap)](strategies/adfs.md#on-premises-behing-wap-configuration) |
| `/fba`        | ❌ | ✅ | [sample](strategies/fba.md#json)                                                                                                                                                             |
| `/tmg`        | ❌ | ✅ | [sample](strategies/tmg.md#json)                                                                                                                                                             |

JSON and struct representations are different in terms of language notations. So credentials parameters names in `private.json` files and declared as structs initiators vary.

### Additional strategies

Gosip supports [custom](custom-auth/) (ad hoc) strategies. Some worthy are boiled in [the Sandbox](https://github.com/koltyakov/gosip-sandbox/tree/master/strategies) to be added later on to the main package in a case of the demand.

| Strategy name         | SPO | On-Prem | Credentials sample(s)                                         |
| --------------------- | --- | ------- | ------------------------------------------------------------- |
| On-Demand             | ✅   | ✅       | [sample](custom-auth/on-demand.md#configure-and-usage-sample) |
| Alternative NTLM      | ❌   | ✅       | [see more](custom-auth/alternative-ntlm.md)                   |
| Dynamic auth (helper) | ✅   | ✅       | [see more](custom-auth/dynamic-auth.md)                       |

### Secrets encoding

When storing credential in local `private.json` files, which can be handy in local development scenarios, we strongly recommend to encode secrets such as `password` or `clientSecret` using [cpass](../utilits/cpass.md). Cpass converts a secret to an encrypted representation which can only be decrypted on the same machine where it was generated. This minimize incidental leaks, i.e. with git commits.
