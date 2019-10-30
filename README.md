### Buildpack User Documentation
This extension buildpack can be used to authenticate each AI to an Active Directory protected service, for example using integrated auth in your app to connect to SQL Server.

### Using the buildpack
Create and bind a service to your application to provide the credentials used by each AI to remotely connect to the network service. This could be done via a user provided service or via the CredHub service broker. You must tag the service with `windows-vault` and `network_address` must use a fully qualified domain name or IP address _with_ the service's port. For SQL Server this is typically port `1433`. Here's an example using a user provided service:

```sh
$ cf cups sql01 -t "windows-vault" -p '{"user":"sqluser","password":"secret","network_address":"sql01.example.com:1433"}'
```

In your app manifest specify the extension buildpack `windows_vault_buildpack` and the service which will provide the credentials to the buildpack, in this case `sql01`. Here's an example app manifest:

```yaml
---
applications:
- name: contoso
  stack: windows
  buildpacks:
  - windows_vault_buildpack
  - hwc_buildpack
  health-check-type: http
  services:
  - sql01
```

To ensure the app will use the credentials when connecting to SQL Server ensure you have a proper connection string which is using integrated with and a matching FQDN of the SQL Server that was specified in the user provided serivce. Here's an example connection string associated with the user provided service:

```
Server=sql01.example.com;Database=contoso;Trusted_Connection=True;
```

With all that in place and your DB already properly created all that's left is to `cf push` the app. Once pushed you should see a message in the push output similar to this:

```
[APP/PROC/WEB/0] OUT Populating the local Windows vault
[APP/PROC/WEB/0] OUT Storing encrypted credentials for sql01.example.com:1433
```

### Building the Buildpack
To build this buildpack, run the following command from the buildpack's directory:

1. Source the .envrc file in the buildpack directory.
```bash
source .envrc
```
To simplify the process in the future, install [direnv](https://direnv.net/) which will automatically source .envrc when you change directories.

1. Install buildpack-packager
```bash
./scripts/install_tools.sh
```

1. Build the buildpack
```bash
buildpack-packager build
```

1. Use in Cloud Foundry
Upload the buildpack to your Cloud Foundry and optionally specify it by name

```bash
cf create-buildpack [BUILDPACK_NAME] [BUILDPACK_ZIP_FILE_PATH] 1
cf push my_app [-b BUILDPACK_NAME]
```

### Testing
Buildpacks use the [Cutlass](https://github.com/cloudfoundry/libbuildpack/cutlass) framework for running integration tests.

To test this buildpack, run the following command from the buildpack's directory:

1. Source the .envrc file in the buildpack directory.

```bash
source .envrc
```
To simplify the process in the future, install [direnv](https://direnv.net/) which will automatically source .envrc when you change directories.

1. Run unit tests

```bash
./scripts/unit.sh
```

1. Run integration tests

```bash
./scripts/integration.sh
```

More information can be found on Github [cutlass](https://github.com/cloudfoundry/libbuildpack/cutlass).

### Reporting Issues
Open an issue on this project

## Disclaimer
This buildpack is experimental and not yet intended for production use.
