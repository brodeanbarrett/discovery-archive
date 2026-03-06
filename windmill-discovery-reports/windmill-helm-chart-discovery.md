# Windmill Helm Chart Discovery Report

## Date: 2026-03-06

## 1. Helm Chart Overview

### Official Helm Chart
- **Repository**: https://github.com/windmill-labs/windmill-helm-charts
- **Helm Repo**: https://windmill-labs.github.io/windmill-helm-charts/
- **Chart Version**: 4.0.96
- **App Version**: 1.651.1

### Community Helm Charts
- Official chart is well-maintained with 800+ commits
- Additional EE charts available at: https://github.com/windmill-labs/windmillhub-helm-charts

## 2. Helm Chart Details

### Installation
```bash
helm repo add windmill https://windmill-labs.github.io/windmill-helm-charts/
helm install windmill windmill/windmill -n windmill --create-namespace
```

### Default Components
| Component | Description |
|-----------|-------------|
| windmill-app | Main Windmill application (frontend + backend) |
| windmill-extra | LSP, Multiplayer, Debugger containers |
| windmill-workers | Job execution workers |
| windmill-postgresql | Included PostgreSQL for demo (can be disabled) |

### Default Configuration
- **appReplicas**: 2
- **extraReplicas**: 1
- **workerReplicas**: 3 (default group) + 1 (native group)
- **baseDomain**: windmill
- **baseProtocol**: http

## 3. Container Images

| Pod | Image | Image Size |
|-----|-------|------------|
| windmill-app | ghcr.io/windmill-labs/windmill:1.651.1 | ~1.4GB (1,469,499,416 bytes) |
| windmill-extra | ghcr.io/windmill-labs/windmill-extra:1.651.1 | - |
| windmill-workers | ghcr.io/windmill-labs/windmill:1.651.1 | ~1.4GB |
| postgresql | postgres:18 | ~155MB (162,312,823 bytes) |

## 4. Resource Specifications (Default)

### Memory Limits/Requests
| Component | Memory Limit | Memory Request |
|-----------|-------------|----------------|
| windmill-app | 2Gi | 2Gi |
| windmill-extra | 1Gi | 1Gi |
| windmill-workers | 2Gi | 2Gi |
| postgresql | 2Gi | 2Gi |

### CPU
- No CPU limits set by default
- No CPU requests set by default

## 5. Actual Resource Usage (Idle)

Measured after successful deployment with pods running:

| Pod | CPU | Memory |
|-----|-----|--------|
| windmill-app-58cf668486-n9wft | 2m | 374Mi |
| windmill-app-58cf668486-rw5jm | 2m | 474Mi |
| windmill-extra-69cc67f97c-9759v | 1m | 100Mi |
| windmill-postgresql-demo-app | 65m | 171Mi |
| windmill-workers-default (x2) | 17m | 29-68Mi |
| windmill-workers-native | 25m | 34Mi |

**Total Memory Usage (idle)**: ~1,250Mi (~1.2GB)

## 6. Kubernetes Services

| Service | Type | Ports |
|---------|------|-------|
| windmill-app | ClusterIP | 8000/TCP |
| windmill-app-metrics | ClusterIP | 8001/TCP |
| windmill-extra | ClusterIP | 3001/TCP, 3002/TCP, 3003/TCP |
| windmill-postgresql | ClusterIP | 5432/TCP |

## 7. Deployment Notes

### Issues Encountered
1. Initial deployment timeout due to large image sizes (~1.4GB per Windmill image)
2. PostgreSQL image pull took ~11 minutes (postgres:18)
3. App pods crashed initially due to PostgreSQL not being ready (CrashLoopBackOff)
4. After PostgreSQL became ready, all pods stabilized

### Image Pull Time
- Windmill images: ~2 seconds (cached after first pull)
- PostgreSQL: ~11 minutes (first pull)

### Recommendations for Production
1. Pre-pull images on cluster nodes
2. Use external PostgreSQL instead of embedded demo version
3. Configure ingress for external access
4. Adjust memory limits based on workload
5. Consider disabling embedded MinIO for production

## 8. Helm Values Customization

### Key Configurable Options
```yaml
windmill:
  baseDomain: "windmill.example.com"
  baseProtocol: "https"
  databaseUrl: "postgres://user:pass@host:5432/windmill"
  appReplicas: 2
  workerGroups:
    - name: "default"
      replicas: 3
      resources:
        limits:
          memory: "2Gi"

postgresql:
  enabled: false  # Use external database

enterprise:
  enabled: false
  licenseKey: ""
  s3CacheBucket: ""
```

## 9. Files Generated
- This report: windmill-discovery/reports/

## 10. Appendix: Worker Groups Configuration

The Helm chart supports multiple worker groups with different configurations:

### Default Worker Group
- **Name**: default
- **Controller**: Deployment
- **Replicas**: 3
- **Privileged**: true (required for PID namespace isolation)
- **Memory Limit**: 2Gi
- **Termination Grace Period**: 604800 seconds (7 days)
  - Allows jobs to complete before pod termination

### Native Worker Group
- **Name**: native
- **Replicas**: 1
- **Privileged**: false
- Uses different memory management and isolation mechanism

### Key Worker Options
- `workerGroups[].name`: Group identifier
- `workerGroups[].replicas`: Number of pods
- `workerGroups[].resources`: Memory/CPU limits
- `workerGroups[].privileged`: Security context
- `workerGroups[].disableUnsharePid`: For nodes with disabled user namespaces
- `workerGroups[].terminationGracePeriodSeconds`: Job completion time

---
*Report generated: 2026-03-06*