# PostgreSQL 16 – middleware namespace

## Overview

This directory contains all Kubernetes manifests to deploy a 3-replica PostgreSQL 16 StatefulSet in the `middleware` namespace.

## StorageClass Detection

| Cluster | StorageClass | Provisioner | Binding Mode |
|---------|-------------|-------------|------------------|
| rancher-desktop | `local-path` (default) | `rancher.io/local-path` | `WaitForFirstConsumer` |

The `storageClassName: local-path` in `05-statefulset.yaml` was set after auto-detecting the available StorageClass.

## Files

| File | Resource |
|------|----------|
| `00-namespace.yaml` | Namespace `middleware` |
| `01-configmap.yaml` | ConfigMap `postgres-config` (DB name, user, PGDATA) |
| `02-secret.yaml` | Secret `postgres-secret` (password) |
| `03-service-headless.yaml` | Headless Service for STS DNS (`postgres-headless`) |
| `04-service-clusterip.yaml` | ClusterIP Service for client access (`postgres`) |
| `05-statefulset.yaml` | StatefulSet `postgres` – 3 replicas, 5Gi PVC each |

## Apply

```bash
kubectl apply -f generated-files/postgres/
```

## Connection

```
Host:     postgres.middleware.svc.cluster.local
Port:     5432
Database: appdb
User:     appuser
Password: (from secret postgres-secret)
```

## Probe Fix

The readiness/liveness probes use `pg_isready` with **explicit args** (not shell env expansion).
Using `exec pg_isready -U $(POSTGRES_USER)` inside a shell caused `role "-d" does not exist`
because the shell split the substituted string as multiple args. The fix passes each flag as a
separate array element directly to the exec probe — no shell involved.
