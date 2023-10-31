# uds-capability-nexus
Bigbang [Nexus Repository Manager](https://repo1.dso.mil/big-bang/product/packages/nexus) deployed via flux by zarf

## Deployment Prerequisites

### Resources
- Minimum compute requirements for single node deployment are at LEAST 64 GB RAM and 32 virtual CPU threads (aws `m6i.8xlarge` instance type should do)
- k3d installed on machine

#### General

- Create `nexus` namespace
- Label `nexus` namespace with `istio-injection: enabled`

#### Database

- A Postgres database is running on port `5432` and accessible to the cluster
- This database can be logged into via the username configured with `ZARF_VAR_NEXUS_DB_USERNAME`. The default is `nexus`.
- This database instance has a psql database created matching the configuration of `ZARF_VAR_NEXUS_DB_NAME`. The default is `nexusdb`.
- The database user has read/write access to the above mentioned database
- Create `nexus-postgres` service in `nexus` namespace that points to the psql database
- Create `nexus-postgres` secret in `nexus` namespace with the key `password` that contains the password to for the configured user of the psql database

#### Pro License
- You must provide a valid Nexus license to use the external DB configuration. If a license is not provided Nexus will default to the OSS version and will use an internal H2 DB.
- Provide your license via the Zarf deploy time variable `nexus_license_key`.
- You can update the [zarf-config.yaml](zarf-config.yaml) in this project. The Makefile will copy that to the build directory to use at deploy time.
- In production, set the `nexus_license_key` in a way that is appropriate for your deployment.

## Deploy

### Use zarf to login to the needed registries i.e. registry1.dso.mil

```bash
# Download Zarf
make build/zarf

# Login to the registry
set +o history

# registry1.dso.mil (To access registry1 images needed during build time)
export REGISTRY1_USERNAME="YOUR-USERNAME-HERE"
export REGISTRY1_TOKEN="YOUR-TOKEN-HERE"
echo $REGISTRY1_TOKEN | build/zarf tools registry login registry1.dso.mil --username $REGISTRY1_USERNAME --password-stdin

set -o history
```

### Build and Deploy Everything via Makefile and local package

```bash
# This will run make build/all, make cluster/reset, and make deploy/all. Follow the breadcrumbs in the Makefile to see what and how its doing it.
make all
```

## Declare This Package In Your UDS Bundle
Below is an example of how to use this projects zarf package in your UDS Bundle

```yaml
kind: UDSBundle
metadata:
  name: example-bundle
  description: An Example UDS Bundle
  version: 0.0.1
  architecture: amd64

zarf-packages:
  # Nexus
  - name: nexus
    repository: ghcr.io/defenseunicorns/uds-capability/nexus
    ref: 0.0.2
```