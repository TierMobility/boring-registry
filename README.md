# boring-registry

Boring Registry is an open source Terraform Module Registry.

The registry is designed to be simple and only implements the "Module Registry Protocol" and apart from the registry storage backend (currently only S3 is supported), there are no external dependencies, it also does not ship with a UI. The endpoints provided are:

* GET /v1/modules/{namespace}/{name}/{provider}/versions
* GET /v1/modules/{namespace}/{name}/{provider}/{version}/download

The storage backend expects a clear path structure to know where modules live.
Example structure:
```
namespace=tier/name=s3/provider=aws/version=1.0.0
```

An example bucket looks like this when all modules have been uploaded:

```
$> tree .
.
└── namespace=tier
    ├── name=s3
    │   └── provider=aws
    │       └── version=1.0.0
    │           └── tier-s3-aws-1.0.0.tar.gz
    └── name=dynamodb
        └── provider=aws
            ├── version=1.0.0
            │   └── tier-dynamodb-aws-1.0.0.tar.gz
            └── version=1.0.1
                └── tier-dynamodb-aws-1.0.1.tar.gz

```

Example Terraform configuration file referencing the registry:
```hcl
module "main-s3" {
  source = "boring-registry/tier/s3/aws"
  version = "~> 1"
}
```

## Getting Started

The registry supports two modes:
  * Server - The server runs the registry API
  * Upload - Uploads modules to the configured registry

To run the server you need to specify which registry to use (currently only S3 is supported):

**Example using the S3 registry:**
```bash
$ boring-registry server \
  -type=s3 \
  -s3-bucket=terraform-registry-test
```

**Example using the registry with GCS:**
```bash
$ boring-registry server \
  -type=gcs \
  -gcs-bucket=terraform-registry-test
```

Make sure the server has GCP credentials context set properly (e.g. `GOOGLE_CLOUD_PROJECT`). 

To upload modules to the registry you need to specify which registry to use (currently only S3 is supported) and which local directory to work from.

**Example using the S3 registry:**
```bash
$ boring-registry upload \
  -type=s3 \
  -s3-bucket=terraform-registry-test terraform/modules
```

**Example using the registry with GCS:**
```bash
$ boring-registry upload \
  -type=gcs \
  -gcs-bucket=terraform-registry-test terraform/modules
```

Make sure the server has GCP credentials context set properly (e.g. `GOOGLE_CLOUD_PROJECT`, `GOOGLE_APPLICATION_CREDENTIALS`).

## Configuration

The Boring Registry does not rely on any configuration files. Instead, everything can be configured using flags or environment variables.
**Important Note**: Flags have higher priority than environment variables. Environment variables are always prefixed with `BORING_REGISTRY`.

**Example:**
To enable debug logging you can either pass the flag: `-debug` or set the environment variable: `BORING_REGISTRY_DEBUG=true`.

### Authentication
The Boring Registry can be configured with a set of API keys to match for by using the `-api-key="very-secure-token"` flag or by providing it as an environment variable `BORING_REGISTRY_API_KEY="very-secure-token"`

This can then be configured inside `~/.terraformrc` like this:
```
credentials "boring-registry" {
  token = “very-secure-token”
}
```

## Uploading modules

When uploading modules the `upload` command expects a directory. This directory is then walked recursively and looks for files called: `boring-registry.hcl`.

The `boring-registry.hcl` file expects a `metadata` block like this:
```hcl
metadata {
  namespace = "tier"
  name      = "s3"
  provider  = "aws"
  version   = "1.0.0"
}
```

When running the upload command, the module is then packaged up and stored inside the registry. 

## Help output

```
USAGE
  boring-registry [flags] <subcommand> [flags] [<arg>...]

SUBCOMMANDS
  server   Runs the server component
  upload   Uploads modules to a registry.
  version  Prints the version
```

### Server help output 

```
USAGE
  boring-registry server -type=<type> [flags]

  Runs the server component.

  This command requires some configuration, such as which registry type to use.

  The server starts two servers (one for serving the API and one for Telemetry).

  Example Usage: boring-registry server -type=s3 -s3-bucket=example-bucket

  For more options see the available options below.

FLAGS
  -api-key=...
  BORING_REGISTRY_API_KEY=...
  Comma-separated string of static API keys to protect the server with.

  -debug=false
  BORING_REGISTRY_DEBUG=false
  Enable debug output.

  -gcs-bucket=...
  BORING_REGISTRY_GCS_BUCKET=...
  Bucket to use when using the GCS registry type.

  -gcs-prefix=...
  BORING_REGISTRY_GCS_PREFIX=...
  Prefix to use when using the GCS registry type.

  -json=false
  BORING_REGISTRY_JSON=false
  Output logs in JSON format.

  -listen-address=:5601
  BORING_REGISTRY_LISTEN_ADDRESS=:5601
  Listen address for the registry api.

  -no-color=false
  BORING_REGISTRY_NO_COLOR=false
  Disables colored output.

  -s3-bucket=...
  BORING_REGISTRY_S3_BUCKET=...
  Bucket to use when using the S3 registry type.

  -s3-prefix=...
  BORING_REGISTRY_S3_PREFIX=...
  Prefix to use when using the S3 registry type.

  -s3-region=...
  BORING_REGISTRY_S3_REGION=...
  Region of the S3 bucket when using the S3 registry type.

  -telemetry-listen-address=:7801
  BORING_REGISTRY_TELEMETRY_LISTEN_ADDRESS=:7801
  Listen address for telemetry.

  -type=...
  BORING_REGISTRY_TYPE=...
  Registry type to use (currently only "s3" and "gcs" is supported).
```

### Upload help output

```
USAGE
  boring-registry upload [flags] <dir>

  Uploads modules to a registry.

  This command requires some configuration,
  such as which registry type to use and a directory to search for modules.

  The upload command walks the directory recursively and looks
  for modules with a boring-registry.hcl file in it. The file is then parsed
  to get the module metadata the module is then archived and uploaded to the given registry.

  Example Usage: boring-registry upload -type=s3 -s3-bucket=example-bucket modules/

  For more options see the available options below.

FLAGS
  -debug=false
  BORING_REGISTRY_DEBUG=false
  Enable debug output.

  -gcs-bucket=...
  BORING_REGISTRY_GCS_BUCKET=...
  Bucket to use when using the GCS registry type.

  -gcs-prefix=...
  BORING_REGISTRY_GCS_PREFIX=...
  Prefix to use when using the GCS registry type.

  -json=false
  BORING_REGISTRY_JSON=false
  Output logs in JSON format.

  -no-color=false
  BORING_REGISTRY_NO_COLOR=false
  Disables colored output.

  -s3-bucket=...
  BORING_REGISTRY_S3_BUCKET=...
  Bucket to use when using the S3 registry type.

  -s3-prefix=...
  BORING_REGISTRY_S3_PREFIX=...
  Prefix to use when using the S3 registry type.

  -s3-region=...
  BORING_REGISTRY_S3_REGION=...
  Region of the S3 bucket when using the S3 registry type.

  -type=...
  BORING_REGISTRY_TYPE=...
  Registry type to use (currently only "s3" and "gcs" is supported).
```

# Roadmap

The project is in its very early stages and there is a lot of things we want to tackle. This may mean some breaking changes in the future, but once the project is stable enough we will put quite heavy focus on keeping changes backwards compatible. This project started out as a single server (just serving the Module Registry Protocol), but is now becoming a single binary that can host the server and allow operators to manage the registry using a streamlined interface.

* Module maintenance - The CLI should be able to inspect/view, modify and delete existing modules.
* Migration helpers - We want the CLI to be able to provide some automation when migrating to the boring-registry. This is currently a manual task and is quite time consuming.
* Extensive metrics - We want to add proper metrics for the server component so we can monitor the internal health of the Boring Registry.
