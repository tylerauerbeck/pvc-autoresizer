Maintenance guide
=================

How to change the supported Kubernetes minor versions
-------------------------------------------

pvc-autoresizer depends on some Kubernetes repositories like `k8s.io/client-go` and should support 3 consecutive Kubernetes versions at a time.
Issues and PRs related to the last upgrade task also help you understand how to upgrade the supported versions,
so checking them together(e.g https://github.com/topolvm/pvc-autoresizer/pull/85) with this guide is recommended when you do this task.

### Check release notes

First of all, we should have a look at the release notes in the order below.

1. TopoLVM
    - Choose the [TopoLVM](https://github.com/topolvm/topolvm/releases) version that supported target Kubernetes version.
2. Kubernetes
    - Choose the next version and check the [release note](https://kubernetes.io/docs/setup/release/notes/). e.g. 1.17, 1.18, 1.19 -> 1.18, 1.19, 1.20
    - Read the [release note](https://github.com/kubernetes-sigs/controller-runtime/releases), and check whether there are serious security fixes and whether the new minor version is compatible with older versions from the pvc-autoresizer's point of view. If there are breaking changes, we should decide how to manage these changes.
    - Read the [kubebuilder go.mod](https://github.com/kubernetes-sigs/kubebuilder/blob/master/go.mod), and check the controller-tools version corresponding to controller-runtime.
    - Check the Kubernetes version which is supported by controller-runtime using the following command.
      ```
      bin/setup-envtest list
      ```
      This version will be used for envtest.
3. Depending tools
    - They does not depend on other software, use latest versions.
      - [helm](https://github.com/helm/helm/releases)
      - [helm-docs](github.com/norwoodj/helm-docs/releases)
      - [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus/releases)
4. Depending modules
    - Read [kubernetes go.mod](https://github.com/kubernetes/kubernetes/blob/master/go.mod), and update the `prometheus/*` modules.
5. Golang version
    - Consider whether the golang should be updated.

We should write down in the github issue of this task what are the important changes and the required actions to manage incompatibilities if exist.
The format is up to you.

Basically, we should pay attention to breaking changes and security fixes first.

### Update written versions

We should also update the following files.

- `README.md`: Documentation which indicates what versions are supported by pvc-autoresizer
- `Makefile`: Makefile for running envtest
- `e2e/Makefile`: Makefile for running e2e tests
- `.github/workflows`: Configuration files of github actions

`git grep <the kubernetes version which support will be dropped>, `git grep image:`, and `git grep -i VERSION` might help to avoid overlooking necessary changes.

### Update dependencies

Next, we should update `go.mod` by the following commands.

```bash
# If the new kubernetes version is v1.x.y", the `VERSION` will be v0.x.y.
$ VERSION=<upgrading Kubernetes release version>
$ go get k8s.io/api@v${VERSION} k8s.io/apimachinery@v${VERSION} k8s.io/client-go@v${VERSION}
```

If we need to upgrade the `controller-runtime` version, do the following as well.

```bash
$ VERSION=<upgrading controller-runtime version>
$ go get sigs.k8s.io/controller-runtime@v${VERSION}
```

Then, please tidy up the dependencies.

```bash
$ go mod tidy
```

These are minimal changes for the Kubernetes upgrade, but if there are some breaking changes found in the release notes, you have to handle them as well in this step.

### Release the changes

We should update [RELEASE.md](../RELEASE.md) to add the entry for the new pvc-autoresizer's version.

### Prepare for the next upgrade

We should create an issue for the next upgrade. Besides, Please update this document if we find something to be updated.
