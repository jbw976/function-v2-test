# function-v2-test

A function for testing function-sdk-go v2 compatibility as part of https://github.com/crossplane/function-sdk-go/pull/226.

The function takes a few pieces of input:

* From the XR:
  * a list of names of `ConfigMap` resources to compose
  * a data value to save in each `ConfigMap`
* From the Composition pipeline step `.input`
  * the name of the data key to use in the each `ConfigMap`

Given the following example XR from the [example folder](./example/):
```yaml
apiVersion: v2.test.crossplane.io/v1
kind: CoolXR
metadata:
  name: example-xr
spec:
  dataValue: "cool-value"
  names:
  - "cool-cm-1"
  - "cool-cm-2"
```

And `Composition` snippet:
```yaml
...
    input:
      apiVersion: template.fn.crossplane.io/v1beta1
      kind: Input
      keyName: "cool-key"
```

This function will compose the following desired resources:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    crossplane.io/composite: example-xr
  name: cool-cm-1
  namespace: default
data:
  cool-key: cool-value
---
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    crossplane.io/composite: example-xr
  name: cool-cm-2
  namespace: default
data:
  cool-key: cool-value
```

## Development

```shell
# Run code generation - see input/generate.go
$ go generate ./...

# Run tests - see fn_test.go
$ go test ./...
```

```shell
# Build the function's runtime image - see Dockerfile
docker build . --quiet --platform=linux/amd64 --tag runtime-amd64
docker build . --quiet --platform=linux/arm64 --tag runtime-arm64
```

```shell
# Build a function package - see package/crossplane.yaml
crossplane xpkg build \
    --package-root=package \
    --embed-runtime-image=runtime-amd64 \
    --package-file=function-amd64.xpkg
crossplane xpkg build \
    --package-root=package \
    --embed-runtime-image=runtime-arm64 \
    --package-file=function-arm64.xpkg
```

```shell
# Push the function package to a registry
crossplane xpkg push \
  --package-files=function-amd64.xpkg,function-arm64.xpkg \
  xpkg.upbound.io/jaredorg/function-v2-test:v0.0.2
```

[functions]: https://docs.crossplane.io/latest/concepts/composition-functions
[go]: https://go.dev
[function guide]: https://docs.crossplane.io/knowledge-base/guides/write-a-composition-function-in-go
[package docs]: https://pkg.go.dev/github.com/crossplane/function-sdk-go
[docker]: https://www.docker.com
[cli]: https://docs.crossplane.io/latest/cli
