# CSI Overlay Driver

This is a fork of the [CSI Hostpath Driver](https://github.com/kubernetes-csi/csi-driver-host-path) with overlay filesystem support.

## Fork Changes

This fork adds the following customizations:

- **Overlay filesystem support** - Extended node server to support overlay mounts
- **Modified driver identity** - Updated driver name and capabilities for overlay use case
- **Simplified deployment** - Streamlined deployment manifests and scripts

### Quick Deployment

```bash
# Deploy the driver
deploy/kubernetes-latest/deploy.sh

# Remove the driver
deploy/kubernetes-latest/destroy.sh
```

### Example Usage

Example pod manifests are available in the `examples/` directory:
- `examples/overlay-pod.yaml` - Pod using overlay volume
- `examples/image-pod.yaml` - Pod using image-backed volume

## How It Works

The CSI Overlay Driver provides **OverlayFS-based volumes** for pods, enabling a copy-on-write storage model where a read-only "golden image" layer is merged with a writable PVC layer.

1. **Golden Image Layer (Lower)**: A pre-cached read-only base image stored at `/var/lib/overlay-csi/golden-image` on each node. This is populated by a DaemonSet InitContainer at cluster startup.

2. **Writable Layer (Upper)**: A user-provided PVC that stores all modifications. Changes to files are written here while the original golden image remains untouched.

3. **OverlayFS Mount**: When a pod requests an overlay volume, the driver creates an OverlayFS mount combining:
   - `lowerdir` → golden image cache (read-only)
   - `upperdir` → PVC mount path (writable)
   - `workdir` → work directory on the PVC

### Key Features

- **Copy-on-Write**: Pods see a merged filesystem; writes go to the PVC while reads can come from either layer
- **Shared Base Image**: Multiple pods can share the same golden image, saving disk space
- **Persistent Changes**: Modifications persist in the PVC across pod restarts
- **Configurable Timeout**: Wait timeout for dependent PVC mounts is configurable via volume context

### Volume Context Parameters

| Parameter | Description |
|-----------|-------------|
| `overlay.csi.io/backside-pvc` | Name of the PVC to use as the writable upper layer (required) |
| `overlay.csi.io/wait-timeout` | Timeout for waiting on PVC mount (default: 120s) |

### Use Case

Ideal for workloads that need a consistent base environment (development tools, libraries, runtime) while allowing per-pod customizations that persist across restarts.

---

# CSI Hostpath Driver (Upstream)

This repository hosts the CSI Hostpath driver and all of its build and dependent configuration files to deploy the driver.

---
*WARNING: This driver is just a demo implementation and is used for CI testing. This has many fake implementations and other non-standard best practices, and should not be used as an example of how to write a real driver.
---

## Pre-requisite
- Kubernetes cluster
- Running version 1.17 or later
- Access to terminal with `kubectl` installed
- VolumeSnapshot CRDs and Snapshot Controller must be installed as part of the cluster deployment (see Kubernetes 1.17+ deployment instructions)

## Features

The driver can provide empty directories that are backed by the same filesystem as EmptyDir volumes. In addition, it can provide raw block volumes that are backed by a single file in that same filesystem and bound to a loop device.

[Various command line parameters](cmd/hostpathplugin/main.go) influence the behavior of the driver. This is relevant in particular for the end-to-end testing that this driver is used for in Kubernetes.

Usually, the driver implements all CSI operations itself. When deployed with the `-proxy-endpoint` parameter, it instead proxies all incoming connections for a CSI driver that is [embedded inside the Kubernetes E2E test suite](https://github.com/kubernetes/kubernetes/tree/master/test/e2e/storage/drivers/csi-test) and used for mocking a CSI driver [with callbacks provided by certain tests](https://github.com/kubernetes/kubernetes/blob/5ad79eae2dcbf33df3b35c48ec993d30fbda46dd/test/e2e/storage/csi_mock_volume.go#L110).

## Deployment
[Deployment for Kubernetes 1.17 and later](docs/deploy-1.17-and-later.md)

## Examples
The following examples assume that the CSI hostpath driver has been deployed and validated:
- [Volume snapshots](docs/example-snapshots-1.17-and-later.md)
- [Inline ephemeral volumes](docs/example-ephemeral.md)

## Building the binaries
If you want to build the driver yourself, you can do so with the following command from the root directory:

```shell
make
```

## Development

### Updating sidecar images
The `deploy/` directory contains manifests for deploying the CSI hostpath driver for different Kubernetes versions.

If you want to update the image versions used in these manifests, you can do so with the following command from the root directory:

```shell
hack/bump-image-versions.sh
```

## Community, discussion, contribution, and support

Learn how to engage with the Kubernetes community on the [community page](http://kubernetes.io/community/).

You can reach the maintainers of this project at:

- [Slack](http://slack.k8s.io/)
- [Mailing List](https://groups.google.com/forum/#!forum/kubernetes-dev)

### Code of conduct

Participation in the Kubernetes community is governed by the [Kubernetes Code of Conduct](code-of-conduct.md).

[owners]: https://git.k8s.io/community/contributors/guide/owners.md
[Creative Commons 4.0]: https://git.k8s.io/website/LICENSE
