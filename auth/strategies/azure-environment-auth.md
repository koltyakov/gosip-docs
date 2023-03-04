---
description: Azure AD  environment-based authentication
---

# Azure Env-based Auth

### Azure App registration

1\. Create or use existing app registration

2\. Make sure that the app is configured for a specific auth scenario:

* Client credentials (_might not work with SharePoint but require a Certificate-based auth_)
* **Certificate**
* **Username/Password** (public clients flows must be enabled)
* Managed identity

Follow instructions: [https://docs.microsoft.com/en-us/sharepoint/dev/solution-guidance/security-apponly-azuread](https://docs.microsoft.com/en-us/sharepoint/dev/solution-guidance/security-apponly-azuread)

* O365 Admin -> Azure Active Directory
* Generate self-signed certificate

```bash
# PowerShell, run on a Windows machine
$certName = "MyCert"
$password = "MyPassword"

$startDate = Get-Date
$endDate = (Get-Date).AddYears(5)
$securePass = (ConvertTo-SecureString -String $password -AsPlainText -Force)

.\Create-SelfSignedCertificate.ps1 -CommonName $certName -StartDate $startDate -EndDate $endDate -Password $securePass
```

or on a Linux or macOS client via `openssl`:

```bash
chmod +x ./Create-SelfSignedCertificate.sh
./Create-SelfSignedCertificate.sh
```

{% hint style="info" %}
Get [scripts](https://github.com/koltyakov/gosip/tree/master/auth/azurecert).
{% endhint %}

* New App Registration
  * Accounts in this organizational directory only
  * API Permissions -> SharePoint :: Application :: Sites.FullControl.All -> Grant Admin Consent
  * Certificates & Secrets -> Upload `.cer` file
* Use environment variables to provide creds bindings:
  * `AZURE_TENANT_ID` - Directory (tenant) ID in App Registration
  * `AZURE_CLIENT_ID` - Application (client) ID in App Registration
  * For certificate-base auth:
    * `AZURE_CERTIFICATE_PATH` - path to `.pfx` file
    * `AZURE_CERTIFICATE_PASSWORD` - password used for self-signed certificate
  * For username/password auth:
    * `AZURE_USERNAME`&#x20;
    * `AZURE_PASSWORD`&#x20;

### Auth configuration and usage

```go
package main

import (
    "fmt"
    "log"
    "os"

    "github.com/koltyakov/gosip"
    "github.com/koltyakov/gosip/api"
    strategy "github.com/koltyakov/gosip-sandbox/strategies/azureenv"
)

func main() {

    // os.Setenv("AZURE_TENANT_ID", "b1bacba7-c38a-414b-8c8b-65df26a15749")
    // os.Setenv("AZURE_CLIENT_ID", "8ca10ce6-c3d5-47c6-b803-0ef3b619f464")
    // os.Setenv("AZURE_CERTIFICATE_PATH", "/path/to/cert.pfx")
    // os.Setenv("AZURE_CERTIFICATE_PASSWORD", "cert-password")

    authCnfg := &strategy.AuthCnfg{
        SiteURL: os.Getenv("SPAUTH_SITEURL"),
    }

    client := &gosip.SPClient{AuthCnfg: authCnfg}
    sp := api.NewSP(client)

    res, err := sp.Web().Select("Title").Get()
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Site title: %s\n", res.Data().Title)

}
```

### Environment variables auto-injection

Environment variables can be automatically injected in a runtime for Azure AAD library. To use injection add correcponding environment variables in `private.json` into `env` JSON property:

```javascript
{
  "siteUrl": "https://contoso.sharepoint.com/sites/site",
  "env": {
    "AZURE_TENANT_ID": "1efde0dc-21f5-4d3d-a053-1da762c7838c",
    "AZURE_CLIENT_ID": "7278fe9b-acd5-4be5-b688-999603560d31",
    "AZURE_CERTIFICATE_PATH": "./certs/MyCert.pfx",
    "AZURE_CERTIFICATE_PASSWORD": "MyPassword"
  }
}
```
