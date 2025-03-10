# Development

{{< hint "info" >}}

This document provides details on how to build and run `kwok` and `kwokctl` locally.

{{< /hint >}}

## Directory Structure

- cmd
  - kwok - Main entry point for `kwok`
  - kwokctl - Main entry point for `kwokctl`
- pkg
  - apis - API definitions
    - internalversion - For all internal use only
    - v1alpha1 - For parsing and converting configurations only
  - config - Configuration utilities
  - kwok - `kwok` implementation
  - kwokctl - `kwokctl` implementation

## Building

### Building `kwok` and `kwokctl`

``` bash
IMAGE_PREFIX=local make build
```

On a successful build, the binaries will be located in `./bin/$(go env GOOS)/$(go env GOARCH)`

### Building `kwok` image

```bash
IMAGE_PREFIX=local make build-image
```

The image will be tagged as `local/kwok:${tag}` and can be found in `docker images`

### Starting a local cluster with locally built `kwokctl` and `kwok` using Docker

``` bash
./bin/$(go env GOOS)/$(go env GOARCH)/kwokctl create cluster
```
