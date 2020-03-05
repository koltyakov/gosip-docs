---
description: API methods compatibility
---

# Compatibility matrix

When it comes to a code base which should support multiple platform versions, which APIs obviously changes with time and not aligned together, it can be a challenge maintaining-wise.

Luckily, authentication methods are isolated and pretty static and HTTP client is generic for any SharePoint API consumption, no-matter REST, CSOM, or legacy SOAP are intended to be used.

On the other side, the most of the help for a Go developer might come from Fluent API as it self-documented or intuitive usage-wise, covers mostly demanded use-cases, and abstracts most of the API nuances and complexity under the hood so a Go developer should not know lots of SharePoint since start. In most of the cases, when it come to a method which only supported let's say in SharePoint Online we put a special comment spotlighting versions support nuances. But a comment can be easily missed out. We plan to introduce sort of a tool for analysis unsupported methods in a custom code using Gosip Fluent API which verify that only supported methods are used for a targeted platform, this might happen in future in case of a reasonable demand. Until then we recommend manual verification of the methods and API entities support across the platform versions using [sp-metadata](https://github.com/koltyakov/sp-metadata) project.

