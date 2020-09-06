---
description: Azure AD Certificate (Environment-Based) authentication (sample)
---

# Azure Certificate Auth

The sample shows Gosip [custom auth](https://go.spflow.com/auth/custom-auth) with [AAD Environment-Based Authorization](https://docs.microsoft.com/en-us/azure/developer/go/azure-sdk-authorization#use-environment-based-authentication) using Certificates in particular.

### Custom auth implementation

{% hint style="success" %}
Checkout [the code](https://github.com/koltyakov/gosip-sandbox/blob/master/strategies/azureenv/azureenv.go).
{% endhint %}

### Azure App registration

1. Create or use existing app registration

2. Make sure that the app is configured for a specific auth scenario:

* Client credentials \(might not work with SharePoint but require a Certificate-based auth\)
* **Certificate**
* Username/Password
* Managed identity

Follow instructions: [https://docs.microsoft.com/en-us/sharepoint/dev/solution-guidance/security-apponly-azuread](https://docs.microsoft.com/en-us/sharepoint/dev/solution-guidance/security-apponly-azuread)

* O365 Admin -&gt; Azure Active Directory
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

* New App Registration
  * Accounts in this organizational directory only
  * API Permissions -&gt; SharePoint :: Application :: Sites.FullControl.All -&gt; Grant Admin Consent
  * Certificates & Secrets -&gt; Upload `.cer` file
* Use environment variables to provide creds bindings:
  * `AZURE_TENANT_ID` - Directory \(tenant\) ID in App Registration
  * `AZURE_CLIENT_ID` - Application \(client\) ID in App Registration
  * `AZURE_CERTIFICATE_PATH` - path to `.pfx` file
  * `AZURE_CERTIFICATE_PASSWORD` - password used for self-signed certificate

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

