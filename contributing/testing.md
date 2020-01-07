---
description: "\U0001F6A6Gosip automated testing"
---

# Testing

In a clone or fork.

### Authentication testing

Create auth credentials store files in `./config` folder for corresponding strategies:

* [private.onprem-adfs.json](../auth/strategies/adfs.md#on-premises-configuration)
* [private.onprem-fba.json](../auth/strategies/fba.md#json)
* [private.onprem-ntlm.json](../auth/strategies/ntlm.md#json)
* [private.onprem-tmg.json](../auth/strategies/tmg.md#json)
* [private.onprem-wap-adfs.json](../auth/strategies/adfs.md#on-premises-behing-wap-configuration)
* [private.onprem-wap.json](../auth/strategies/adfs.md#on-premises-behing-wap-configuration)
* [private.spo-addin.json](../auth/strategies/addin.md#json)
* [private.spo-user.json](../auth/strategies/saml.md#json)
* [private.spo-adfs.json](../auth/strategies/adfs.md#sharepoint-online-configuration)

See [samples](./config/samples).

```bash
go test ./... -v -race -count=1
```

Not provided auth configs and therefore strategies are ignored and not skipped in tests.

### API integration tests

Create auth credentials store files in `./config/integration` folder for corresponding environments:

* private.spo.json
* private.2013.json // 2013 has its nuances with OData mod and not supported methods

```bash
SPAUTH_ENVCODE=spo go test ./api/... -v -count=1
SPAUTH_ENVCODE=2013 go test ./api/... -v -count=1
```

**Environment variables**

* `SPAUTH_ENVCODE=code` environment variable switches target environments. `spo` is a default one.
* `SPAPI_HEAVY_TESTS=true` turns on "heavy" methods, e.g. web creation.

### Run manual tests

Modify `cmd/test/main.go` to include required scenarios and run:

```bash
go run ./cmd/test
```

Optionally, you can provide a strategy to use with a corresponding flag:

```bash
go run ./cmd/test -strategy adfs
```

### Run CI tests

Configure environment variables:

* SPAUTH\_SITEURL
* SPAUTH\_CLIENTID
* SPAUTH\_CLIENTSECRET
* SPAUTH\_USERNAME
* SPAUTH\_PASSWORD

```bash
go test ./... -v -race -count=1
SPAUTH_ENVCODE=spo go test ./api/... -v -count=1
```

