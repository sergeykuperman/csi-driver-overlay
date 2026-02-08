# CSI Hostpath Driver

> **Warning**: This is a demo/testing CSI driver implementation. It contains fake implementations and non-standard practices. Do NOT use as a reference for production CSI drivers.

## Project Overview

A Container Storage Interface (CSI) driver for Kubernetes that provides simple local storage backed by the host filesystem. Used primarily for CI testing and CSI feature development.

**Core capabilities:**
- Filesystem volumes (empty directories via bind mounts)
- Block volumes (single files bound to loop devices)
- Snapshots via tar archives
- Group snapshots, volume cloning, expansion
- Proxy mode for Kubernetes E2E testing

## Tech Stack

| Component | Technology |
|-----------|------------|
| Language | Go 1.25.5 |
| CSI Spec | v1.12.0 (`github.com/container-storage-interface/spec`) |
| gRPC | v1.78.0 |
| Kubernetes | v1.35.0 dependencies |
| Logging | `k8s.io/klog/v2` |
| Build | Makefile + release-tools |
| Container | Alpine Linux base |

## Project Structure

```
cmd/hostpathplugin/     Main entry point
pkg/hostpath/           CSI service implementations (Controller, Node, Identity, etc.)
pkg/state/              Persistent state management (volumes, snapshots)
internal/endpoint/      Unix/TCP socket handling
internal/proxy/         E2E test proxy
deploy/                 Kubernetes manifests (1.30+, distributed, test configs)
release-tools/          Shared build/release tooling
```

### Key Files

| File | Purpose |
|------|---------|
| `cmd/hostpathplugin/main.go:41-139` | Entry point, flag parsing, driver init |
| `pkg/hostpath/hostpath.go:52-92` | Core driver struct and config |
| `pkg/hostpath/controllerserver.go` | Volume/snapshot lifecycle (CreateVolume, CreateSnapshot) |
| `pkg/hostpath/nodeserver.go` | Mount operations (NodePublishVolume, NodeStageVolume) |
| `pkg/hostpath/identityserver.go` | Driver info and capabilities |
| `pkg/state/state.go:84-149` | State interface definition |

## Build Commands

```bash
make                    # Build binary to ./bin/hostpathplugin
make container          # Build Docker image
make test               # Run unit tests
```

## Deployment

```bash
deploy/kubernetes-latest/deploy.sh      # Deploy to cluster
deploy/kubernetes-latest/destroy.sh     # Remove from cluster
```

## Manual Testing

```bash
# Start driver
sudo ./bin/hostpathplugin --endpoint tcp://127.0.0.1:10000 --nodeid CSINode -v=5

# Test with csc tool
csc identity plugin-info --endpoint tcp://127.0.0.1:10000
csc controller new --endpoint tcp://127.0.0.1:10000 --cap 1,block CSIVolumeName
```

## Node-Level Debugging

To investigate storage or mount issues at the node level:

```bash
kubectl node-shell <node-name>
```

## Key Configuration Flags

| Flag | Purpose |
|------|---------|
| `--endpoint` | CSI socket (unix:// or tcp://) |
| `--nodeid` | Node identifier (required) |
| `--statedir` | State persistence directory |
| `--capacity` | Storage simulation (e.g., `fast=100Gi`) |
| `--enable-attach` | Enable controller publish/unpublish |
| `--proxy-endpoint` | Enable E2E proxy mode |

Full list: `cmd/hostpathplugin/main.go:42-92`

## CSI Services Implemented

| Service | File | Key Methods |
|---------|------|-------------|
| Identity | `identityserver.go` | GetPluginInfo, Probe |
| Controller | `controllerserver.go` | CreateVolume, DeleteVolume, CreateSnapshot |
| Node | `nodeserver.go` | NodePublishVolume, NodeStageVolume |
| GroupController | `groupcontrollerserver.go` | CreateVolumeGroupSnapshot |
| SnapshotMetadata | `snapshotmetadataserver.go` | GetMetadataAllocated, GetMetadataDelta |

## Additional Documentation

When working on specific areas, consult these files:

| Topic | File |
|-------|------|
| Architectural patterns, error handling, logging conventions | `.claude/docs/architectural_patterns.md` |
| Manual testing procedures | `pkg/hostpath/README.md` |
| Deployment instructions | `docs/deploy-1.17-and-later.md` |
| Project overview and warnings | `README.md` |
