# Application Kubernetes Deployment Resources - Consolidated Report

**Date:** 2026-03-06  
**Environment:** K3s v1.34.4+k3s1 (honhon-thinkpad-t14s-gen-2a)

---

## Executive Summary

This report consolidates the findings from Kubernetes Helm chart discovery for six open-source projects:
- Prest (prest/prest)
- Pocketbase (pocketbase/pocketbase)
- Windmill (windmill-labs/windmill)
- Temporal (temporalio/temporal)
- Fission (fission/fission)
- Dagger (dagger/dagger)

---

## Quick Comparison Table

| Project | Official Helm Chart | Chart Version | Image Size | Total Memory (Idle) | Resource Limits Set |
|---------|---------------------|---------------|------------|---------------------|---------------------|
| Prest | No (manifests only) | N/A | 54-434 MB | ~12 MiB | No |
| Pocketbase | No (community) | 0.29.3 | 32.8 MB | ~9 MiB | No |
| Windmill | Yes | 4.0.96 | ~1.4 GB | ~1,250 MiB | Yes (2Gi) |
| Temporal | Yes (RC) | 1.0.0-rc.2 | ~174 MB | ~224 MiB | No* |
| Fission | Yes | 1.22.1 | 22.8 MB | ~106 MiB | No |
| Dagger | Yes | 0.20.0 | ~399 MB | ~166 MiB | No |

*Temporal PostgreSQL has limits set (128Mi-192Mi), but Temporal services do not.

---

## Detailed Findings

### 1. Prest (prest/prest)

**Project:** PostgreSQL REST API written in Go

**Helm Chart Status:**
- **Official Helm Chart:** NOT AVAILABLE
- **Kubernetes Manifests:** Available at `https://github.com/prest/prest/tree/main/install-manifests/kubernetes`
- **Community Helm Chart:** NOT AVAILABLE

**Container Images:**

| Tag | Size (uncompressed) | Size (compressed) |
|-----|---------------------|-------------------|
| `prest/prest:v1` | 433.7 MB | ~455 MB |
| `prest/prest:v1-noplugins` | 53.7 MB | ~54 MB |
| `prest/prest:latest` | 433.7 MB | ~455 MB |
| `prest/prest:latest-noplugins` | 53.7 MB | ~54 MB |

**Recommendation:** Use `prest/prest:v1-noplugins` for production (54 MB vs 433 MB)

**Resource Configuration:**
- Resource Requests: NOT SET
- Resource Limits: NOT SET

**Actual Measured Usage:**
| Metric | Value |
|--------|-------|
| CPU | 272 mCPU (~0.27 cores) |
| Memory | 12 MiB |
| QoS Class | BestEffort |

**Deployment Method:** Raw Kubernetes manifests (`kubectl apply -f`)

---

### 2. Pocketbase (pocketbase/pocketbase)

**Project:** Open-source backend-as-a-service platform

**Helm Chart Status:**
- **Official Helm Chart:** NOT AVAILABLE
- **Community Charts Available:**
  - `techwolf12/pocketbase` (recommended, v0.29.3)
  - `SeProprioDev/pocketbase-docker`
  - `litesql/pocketbase-ha` (HA variant)

**Container Image:**

| Image | Size |
|-------|------|
| `ghcr.io/techwolf12/pocketbase:0.29.3` | 32.8 MB |

**Resource Configuration:**
- Resource Requests: NOT SET
- Resource Limits: NOT SET

**Actual Measured Usage:**
| Metric | Value |
|--------|-------|
| CPU | 1 mCPU (0.001 cores) |
| Memory | 9 MiB |
| Storage | 2 Gi (PVC) |

**Persistence:** 2Gi PVC using local-path storage class

---

### 3. Windmill (windmill-labs/windmill)

**Project:** Open-source workflow automation platform

**Helm Chart Status:**
- **Official Helm Chart:** YES
- **Repository:** https://windmill-labs.github.io/windmill-helm-charts/
- **Chart Version:** 4.0.96
- **App Version:** 1.651.1

**Container Images:**

| Image | Size |
|-------|------|
| `ghcr.io/windmill-labs/windmill:1.651.1` | ~1.4 GB (1,469,499,416 bytes) |
| `ghcr.io/windmill-labs/windmill-extra:1.651.1` | - |
| `postgres:18` | ~155 MB |

**Resource Configuration (Default):**

| Component | Memory Limit | Memory Request |
|-----------|--------------|----------------|
| windmill-app | 2Gi | 2Gi |
| windmill-extra | 1Gi | 1Gi |
| windmill-workers | 2Gi | 2Gi |
| postgresql | 2Gi | 2Gi |

**Actual Measured Usage (Idle):**

| Pod | CPU | Memory |
|-----|-----|--------|
| windmill-app (x2) | 2m each | 374-474 MiB |
| windmill-extra | 1m | 100 MiB |
| windmill-workers-default (x2) | 17m | 29-68 MiB |
| windmill-workers-native | 25m | 34 MiB |
| postgresql | 65m | 171 MiB |

**Total Memory Usage (idle):** ~1,250 MiB (~1.2 GB)

**Notes:**
- Large image size (~1.4GB) causes long initial pull times
- PostgreSQL image took ~11 minutes to pull on first deployment
- CPU limits not set by default

---

### 4. Temporal (temporalio/temporal)

**Project:** Workflow orchestration engine

**Helm Chart Status:**
- **Official Helm Chart:** YES (RC - Release Candidate)
- **Repository:** https://github.com/temporalio/helm-charts
- **Chart Version:** 1.0.0-rc.2
- **Server Version:** 1.29.2

**Container Images:**

| Image | Size |
|-------|------|
| `temporalio/server:1.29.2` | ~174 MB |
| `temporalio/ui:2.42.1` | - |
| `temporalio/admin-tools:1.29.1-tctl-1.18.4-cli-1.5.0` | ~210 MB |
| `bitnami/postgresql:latest` | ~123 MB |

**Resource Configuration:**

| Component | CPU Request | CPU Limit | Memory Request | Memory Limit |
|-----------|-------------|-----------|----------------|--------------|
| Temporal Services | Not set | Not set | Not set | Not set |
| PostgreSQL | 100m | 150m | 128Mi | 192Mi |

**Actual Measured Usage:**

| Pod | CPU | Memory |
|-----|-----|--------|
| temporal-frontend | 10m | 34 MiB |
| temporal-history | 11m | 38 MiB |
| temporal-matching | 11m | 39 MiB |
| temporal-worker | 10m | 30 MiB |
| temporal-web | 1m | 6 MiB |
| postgresql | 12m | 77 MiB |

**Total Memory Usage:** ~224 MiB

**Persistence:** PostgreSQL databases (temporal, temporal_visibility)

**Notes:**
- Requires external PostgreSQL, MySQL, or Cassandra
- Official Helm repository was inaccessible during deployment
- Chart is still in RC status

---

### 5. Fission (fission/fission)

**Project:** Serverless framework for Kubernetes

**Helm Chart Status:**
- **Official Helm Chart:** YES
- **Repository:** https://fission.github.io/fission-charts/
- **Chart Version:** 1.22.1
- **App Version:** v1.22.0

**Container Image:**

| Image | Size |
|-------|------|
| `ghcr.io/fission/fission-bundle:v1.22.0` | 22.8 MB |
| `ghcr.io/fission/reporter:v1.22.0` | 4.25 MB |

**Note:** All 8 pods use the same `fission-bundle` image with different runtime arguments

**Resource Configuration:**
- Resource Requests: NOT SET (BestEffort QoS)
- Resource Limits: NOT SET
- Exception: `fetcher` has small requests (10m CPU, 16Mi memory)

**Actual Measured Usage:**

| Pod | CPU | Memory |
|-----|-----|--------|
| buildermgr | 0m | 15 MiB |
| executor | 5m | 14 MiB |
| kubewatcher | 0m | 14 MiB |
| mqtrigger-keda | 0m | 14 MiB |
| router | 5m | 14 MiB |
| storagesvc | 2m | 11 MiB |
| timer | 0m | 14 MiB |
| webhook | 1m | 10 MiB |

**Total:**
- CPU: ~13m cores
- Memory: ~106 MiB

**Persistence:** 8Gi PVC for function storage

**Notes:**
- All components run as non-root (user 10001)
- executor and router pods had readiness probe failures in local K3s (service discovery issue)

---

---

### 6. Dagger (dagger/dagger)

**Project:** CI/CD Engine that runs pipelines in containers

**Helm Chart Status:**
- **Official Helm Chart:** YES
- **Registry:** `oci://ghcr.io/dagger/dagger-helm`
- **Chart Version:** 0.20.0
- **Source:** https://github.com/dagger/dagger

**Community Alternatives:**
- `kubiya-runner` - Available on Artifact Hub

**Container Image:**

| Image | Size |
|-------|------|
| `registry.dagger.io/engine:v0.20.0` | ~399 MB |

**Resource Configuration:**
- Resource Requests: NOT SET
- Resource Limits: NOT SET
- Default deployment runs as privileged container (required for container builds)

**Actual Measured Usage:**
| Metric | Value |
|--------|-------|
| CPU | 103 mCPU |
| Memory | 166 MiB |
| QoS Class | BestEffort |

**Volume Mounts:**
- `/var/lib/dagger` - Data volume
- `/run/dagger` - Runtime volume

**Security Note:** Dagger Engine runs as a privileged container with root access - required for Docker-in-Docker and container builds.

**Deployment Type:** DaemonSet (default) or StatefulSet (optional)

---

## Recommendations Summary

### For Minimum Resource Usage
1. **Fission** - ~106 MB idle memory, smallest image (22.8 MB)
2. **Pocketbase** - ~9 MB idle memory, small image (32.8 MB)
3. **Prest** - ~12 MB idle memory, but no Helm chart

### For Production Deployments
1. **Add resource limits** to all deployments (currently none set for most)
2. **Use noplugins variants** where available (Prest: 54 MB vs 434 MB)
3. **Pre-pull large images** on cluster nodes (Windmill: 1.4 GB)
4. **Configure persistence** appropriately for each application

### For Helm Chart Availability
- **Official charts:** Windmill, Temporal, Fission
- **Community charts only:** Pocketbase
- **No chart (manifests only):** Prest

---

## Cleanup Status

All deployments have been cleaned up:
- Namespaces deleted: prest, pocketbase, windmill, temporal, fission, dagger
- Helm releases removed
- Container images removed from local registry

---

## References

- Prest: https://github.com/prest/prest
- Pocketbase: https://pocketbase.io/
- Windmill: https://windmill.dev/
- Temporal: https://temporal.io/
- Fission: https://fission.io/

- Dagger: https://dagger.io/
---

*Report generated: 2026-03-06*
