---
description: Azure AD  Certificate authentication
---

# Azure Certificate Auth

This article is the sample showing Gosip custom auth with [AAD Certificate Authorization](https://docs.microsoft.com/en-us/azure/developer/go/azure-sdk-authorization).

### Azure App registration

1\. Create or use existing app registration

2\. Make sure that the app is configured for a specific auth scenario:

* Certificate

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

### JSON

`private.json` sample:

```javascript
{
	"siteUrl": "https://contoso.sharepoint.com/sites/test",
	"tenantId": "e4d43069-8ecb-49c4-8178-5bec83c53e9d",
	"clientId": "628cc712-c9a4-48f0-a059-af64bdbb4be5",
	"certPath": "cert.pfx",
	"certPass": "password"
}
```

### Usage sample

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/koltyakov/gosip"
	"github.com/koltyakov/gosip/api"
	strategy "github.com/koltyakov/gosip/auth/azurecert"
)

func main() {

	// authCnfg := &strategy.AuthCnfg{
	// 	SiteURL:  os.Getenv("SPAUTH_SITEURL"),
	// 	TenantID: os.Getenv("AZURE_TENANT_ID"),
	// 	ClientID: os.Getenv("AZURE_CLIENT_ID"),
	// 	CertPath: os.Getenv("AZURE_CERTIFICATE_PATH"),
	// 	CertPass: os.Getenv("AZURE_CERTIFICATE_PASSWORD"),
	// }
	// or using `private.json` creds source

	authCnfg := &strategy.AuthCnfg{}
	configPath := "./config/private.json"
	if err := authCnfg.ReadConfig(configPath); err != nil {
		log.Fatalf("unable to get config: %v", err)
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

