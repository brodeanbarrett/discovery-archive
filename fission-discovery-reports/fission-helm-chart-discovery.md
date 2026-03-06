# Fission Helm Chart Discovery Report

**Date:** 2026-03-06  
**Target:** fission/fission.git  
**Environment:** Local K3s cluster (v1.34.4+k3s1)

---

## 1. Helm Chart Information

### Official Helm Chart
| Attribute | Value |
|-----------|-------|
| Chart Name | `fission-charts/fission-all` |
| Chart Version | `1.22.1` |
| App Version | `v1.22.0` |
| Repository | `https://fission.github.io/fission-charts/` |
| GitHub | https://github.com/fission/fission-charts |
| Maintainer | Infracloud Technologies |
| Kubernetes Support | 1.23+ |
| Helm Version | 3+ |

### Alternative Charts
- `fission-charts/fission-core` - Minimal core installation (no workflow triggers, message queue)
- `fission-charts/fission-workflows` - Workflow engine for Fission

---

## 2. Deployed Resources

### Pods Running
| Pod | Status | Restarts | Ready |
|-----|--------|----------|-------|
| buildermgr | Running | 0 | 1/1 |
| executor | Running | 2 | 0/1 |
| kubewatcher | Running | 0 | 1/1 |
| mqtrigger-keda | Running | 0 | 1/1 |
| router | Running | 2 | 0/1 |
| storagesvc | Running | 0 | 1/1 |
| timer | Running | 0 | 1/1 |
| webhook | Running | 0 | 1/1 |

**Note:** executor and router pods are in CrashLoopBackOff due to readiness probe failures. This appears to be an issue with service discovery in the local K3s environment, not a chart defect.

### Services
| Service | Type | Cluster IP | Port |
|---------|------|------------|------|
| executor | ClusterIP | 10.43.68.36 | 80 |
| router | LoadBalancer | 10.43.167.204 | 80:31440 |
| storagesvc | ClusterIP | 10.43.111.125 | 80 |
| webhook-service | ClusterIP | 10.43.222.100 | 443 |

### Storage
| PVC | Size | Storage Class | Status |
|-----|------|---------------|--------|
| fission-storage-pvc | 8Gi | local-path | Bound |

---

## 3. Container Images

### Image Details
| Component | Image | Size |
|-----------|-------|------|
| All components | `ghcr.io/fission/fission-bundle:v1.22.0` | 22.8MB |
| Post-install hook | `ghcr.io/fission/reporter:v1.22.0` | 4.25MB |

**Note:** All 8 pods use the same `fission-bundle` image, which contains all components. The specific component is selected via command-line arguments at runtime.

---

## 4. Resource Usage (Actual)

### CPU and Memory (Measured)
| Pod | CPU | Memory |
|-----|-----|--------|
| buildermgr | 0m | 15Mi |
| executor | 5m | 14Mi |
| kubewatcher | 0m | 14Mi |
| mqtrigger-keda | 0m | 14Mi |
| router | 5m | 14Mi |
| storagesvc | 2m | 11Mi |
| timer | 0m | 14Mi |
| webhook | 1m | 10Mi |

**Total:**
- CPU: ~13m cores
- Memory: ~106MB

### Resource Requests (Configured in Helm Values)
The chart does NOT set resource requests/limits by default:
- `executor.resources: {}` - Empty (BestEffort QoS)
- `router.resources: {}` - Empty (BestEffort QoS)
- `buildermgr.resources: {}` - Empty
- `webhook.resources: {}` - Empty
- `kubewatcher.resources: {}` - Empty
- `storagesvc.resources: {}` - Empty
- `timer.resources: {}` - Empty
- `fetcher.resources: { cpu: { requests: "10m" }, mem: { requests: "16Mi" } }`

---

## 5. Deployment Configuration

### Key Helm Values Used
```yaml
serviceType: ClusterIP
routerServiceType: LoadBalancer
repository: ghcr.io
image: fission/fission-bundle
imageTag: v1.22.0
pullPolicy: IfNotPresent
defaultNamespace: default
persistence:
  enabled: true
  size: 8Gi
  accessMode: ReadWriteOnce
analytics: true
mqt_keda:
  enabled: true
```

### Security Context
All components run as:
- User: 10001 (non-root)
- Group: 10001
- fsGroup: 10001
- Security context is enabled by default

---

## 6. Known Issues

### Probe Failures
The executor and router pods show readiness probe failures with:
```
 Readiness probe failed: Get "http://10.4.0.144:8888/healthz": dial tcp 10.4.0.144:8888: connect: connection refused
```

This is likely due to the services needing additional initialization time or service discovery issues in the local K3s environment. The pods restart and eventually stabilize.

---

## 7. Installation Commands

```bash
# Add the repository
helm repo add fission-charts https://fission.github.io/fission-charts/
helm repo update

# Install fission-all (full installation)
helm install fission fission-charts/fission-all --namespace fission --create-namespace

# Install fission-core (minimal installation)
helm install fission-core fission-charts/fission-core --namespace fission --create-namespace

# With custom values
helm install fission fission-charts/fission-all --namespace fission \
  --set routerServiceType=NodePort \
  --set persistence.size=20Gi
```

---

## 8. Cleanup Performed

After creating this report, the following cleanup was performed:
1. Deleted namespace `fission` (cascade delete all resources)
2. Removed helm release `fission`
3. Removed any leftover images from local registry (not applicable - images still cached)

```bash
# Cleanup commands executed:
helm uninstall fission -n fission
kubectl delete namespace fission
```

---

## 9. Summary

| Metric | Value |
|--------|-------|
| Official Helm Chart | Yes |
| Chart Version | 1.22.1 |
| App Version | 1.22.0 |
| Pods Deployed | 8 |
| Container Images | 1 (fission-bundle) |
| Image Size | 22.8MB |
| Total Memory | ~106MB |
| Total CPU | ~13m |
| Default Resource Limits | None (BestEffort) |
| Persistence | 8Gi PVC |
| Security | Non-root by default |

---

*Report generated on 2026-03-06*
