# Pocketbase Helm Chart Discovery Report

**Date:** 2026-03-06  
**Author:** Automated Discovery  
**Cluster:** K3s v1.34.4+k3s1 (honhon-thinkpad-t14s-gen-2a)

---

## 1. Executive Summary

**Finding:** No official Helm chart exists for Pocketbase. Multiple community-maintained Helm charts are available.

| Chart | Version | Status |
|-------|---------|--------|
| techwolf12/pocketbase | 0.29.3 | **Deployed** |
| SeProprioDev/pocketbase-docker | Multiple | Community |
| litesql/pocketbase-ha | 0.0.3 | HA variant |

---

## 2. Official vs Community Charts

### Official Chart
- **Status:** Does NOT exist
- Pocketbase is an open-source project without an official Kubernetes Helm chart
- Official deployment guidance is limited to Docker Compose and binary deployment

### Community Charts Available

#### a) Techwolf12/pocketbase (Recommended)
- **Repository:** https://helm.techwolf12.nl
- **Artifact Hub:** https://artifacthub.io/packages/helm/techwolf12/pocketbase
- **GitHub:** https://github.com/techwolf12/charts
- **Latest Version:** 0.29.3
- **App Version:** v0.29.3
- **Maintainer:** Christian de Die le Clercq (contact@techwolf12.nl)
- **Type:** Application
- **Active:** Yes - updated September 2025

#### b) SeProprioDev/pocketbase-docker
- **Repository:** https://github.com/SeProprioDev/pocketbase-docker
- **Type:** Community helm chart with Docker setup
- **Versions:** Multiple (aligned with Pocketbase releases)

#### c) litesql/pocketbase-ha
- **Artifact Hub:** https://artifacthub.io/packages/helm/litesql/pocketbase-ha
- **Purpose:** High-availability deployment variant
- **Version:** 0.0.3

---

## 3. Deployed Chart Details

### Installation
```bash
helm repo add techwolf12 https://helm.techwolf12.nl
helm install pocketbase techwolf12/pocketbase --version 0.29.3 --namespace pocketbase --create-namespace
```

### Resources Deployed

| Resource | Name | Status |
|----------|------|--------|
| Deployment | pocketbase | Available |
| ReplicaSet | pocketbase-5dc77f7f98 | Ready (1/1) |
| Pod | pocketbase-5dc77f7f98-hzggl | Running |
| PVC | pocketbase-pvc | Bound (2Gi) |
| PV | pvc-c084390e-6207-4a87-985c-8cd9f7a8563a | Bound |

### Container Image

| Property | Value |
|----------|-------|
| Image | ghcr.io/techwolf12/pocketbase:0.29.3 |
| Image Size | 32.8MB |
| Image Pull Policy | Always |
| Container Port | 8090 (HTTP) |
| Command | /pocketbase/pocketbase serve --http=0.0.0.0:8090 |

### Resource Requests/Limits

| Metric | Value |
|--------|-------|
| CPU Request | Not set |
| Memory Request | Not set |
| CPU Limit | Not set |
| Memory Limit | Not set |

**Note:** The helm chart does NOT set resource requests or limits by default. Users must configure these in their values file.

### Actual Resource Usage

| Metric | Value |
|--------|-------|
| CPU | 1m (0.001 cores) |
| Memory | 9 MiB |

### Persistence

| Property | Value |
|----------|-------|
| Storage Requested | 2Gi |
| Storage Class | local-path (K3s default) |
| Mount Path | /pocketbase/pb_data |

### Configuration Options (values.yaml)

```yaml
replicaCount: 1
image:
  repository: ghcr.io/techwolf12/pocketbase
  pullPolicy: Always
  tag: "0.29.3"
persistence:
  storage: 2Gi
  storageClass: ""
ingress:
  enabled: false
  host: ""
  tlsSecret: ""
resources: {}  # No defaults - user must configure
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
flags:
  encryptionEnv: ""  # PB_ENCRYPTION_KEY
```

### Known Issues

1. **Security Vulnerabilities:** Chart scan identified 56 vulnerabilities (including critical and high severity)
2. **Empty imagePullSecrets:** Warning during deployment - chart has empty pull secret configuration
3. **No resource defaults:** Requires manual configuration for production use

---

## 4. Recommendations

### For Development
- Use the Techwolf12 chart directly with default values
- Ensure encryption key is set via `flags.encryptionEnv`

### For Production
1. **Set resource limits:**
   ```yaml
   resources:
     requests:
       memory: "256Mi"
       cpu: "100m"
     limits:
       memory: "512Mi"
       cpu: "500m"
   ```

2. **Configure encryption key:**
   ```yaml
   flags:
     encryptionEnv: "your-32-character-encryption-key"
   ```

3. **Enable ingress** for external access

4. **Consider backup strategy** for the persistent volume

5. **Review security vulnerabilities** before production use

---

## 5. Files Created

- **Report Location:** `pocketbase-discovery/reports/pocketbase-helm-chart-report.md`

---

## 6. References

- [Artifact Hub - Techwolf12/pocketbase](https://artifacthub.io/packages/helm/techwolf12/pocketbase)
- [GitHub - Techwolf12/charts](https://github.com/techwolf12/charts)
- [Pocketbase Official Docs](https://pocketbase.io/docs/going-to-production/)
- [GitHub - SeProprioDev/pocketbase-docker](https://github.com/SeProprioDev/pocketbase-docker)
