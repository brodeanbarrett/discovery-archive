# Prest (prest/prest) Kubernetes Helm Chart Discovery Report

## Executive Summary

This report documents the findings of a Kubernetes helm chart discovery for **Prest (prest/prest)**, a PostgreSQL REST API written in Go.

**Key Finding**: There is **no official or community helm chart** for Prest. The project provides raw Kubernetes manifests for deployment.

---

## 1. Helm Chart Availability

### Official Helm Chart
- **Status**: NOT AVAILABLE
- There is no official helm chart in the prest/prest repository or any associated organization.

### Community Helm Chart
- **Status**: NOT AVAILABLE
- No community-maintained helm charts found in Helm Hub or popular chart repositories.
- Search conducted via:
  - `helm search hub prest`
  - `helm search repo prest`
  - Web search for "prestd helm chart"

### Kubernetes Manifests
- **Source**: Official repository
- **Location**: `https://github.com/prest/prest/tree/main/install-manifests/kubernetes`
- **Files**:
  - `deployment.yaml` - Deployment manifest
  - `svc.yaml` - Service manifest
  - `README.md` - Deployment instructions

---

## 2. Docker Image Information

### Primary Image
| Tag | Size (uncompressed) | Size (compressed) | Architecture |
|-----|---------------------|-------------------|--------------|
| `prest/prest:v1` | 433.7 MB | ~455 MB | amd64 |
| `prest/prest:v1-noplugins` | 53.7 MB | ~54 MB | amd64 |
| `prest/prest:latest` | 433.7 MB | ~455 MB | amd64 |
| `prest/prest:latest-noplugins` | 53.7 MB | ~54 MB | amd64 |

### Image Variants
- **With plugins** (`v1`, `latest`): Includes all plugins, ~433 MB
- **No plugins** (`v1-noplugins`, `latest-noplugins`): Minimal image without plugins, ~54 MB (recommended for production)

---

## 3. Kubernetes Deployment Details

### Deployment Manifest (from official repo)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prest
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prest
  template:
    spec:
      containers:
        - image: prest/prest:v1
          imagePullPolicy: IfNotPresent
          name: prest
          ports:
            - containerPort: 3000
              protocol: TCP
          env:
            - name: DATABASE_URL
              value: "postgres://username:password@hostname:port/dbname"
            - name: PREST_DEBUG
              value: "true"
            - name: PREST_JWT_DEFAULT
              value: "true"
          resources: {}
```

### Service Manifest

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prest
  namespace: default
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  selector:
    app: prest
  type: LoadBalancer
```

---

## 4. Resource Usage Analysis

### Resource Requests/Limits
- **Status**: NOT DEFINED in official manifest
- The official deployment.yaml has `resources: {}` - no requests or limits are set

### Actual Resource Consumption (Measured)

| Metric | Value |
|--------|-------|
| CPU | 272 mCPU (~0.27 cores) |
| Memory | 12 MiB |
| QoS Class | BestEffort |

### Recommendations
For production deployments, add resource limits:

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

---

## 5. Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | Required |
| `PREST_DEBUG` | Enable debug mode | false |
| `PREST_JWT_DEFAULT` | Enable default JWT | false |
| `PREST_JWT_KEY` | JWT secret key | (none) |
| `PREST_PG_HOST` | PostgreSQL host | localhost |
| `PREST_PG_PORT` | PostgreSQL port | 5432 |
| `PREST_PG_USER` | PostgreSQL user | postgres |
| `PREST_PG_DATABASE` | PostgreSQL database | postgres |
| `PREST_PG_MAX_OPEN_CONNS` | Max open connections | 25 |
| `PREST_PG_MAX_IDLE_CONNS` | Max idle connections | 25 |
| `PREST_HTTP_PORT` | HTTP server port | 3000 |
| `PREST_LOG_LEVEL` | Log level | info |

---

## 6. Deployment Options

### Option 1: Raw Kubernetes Manifests (Official)
```bash
kubectl create -f deployment.yaml
kubectl create -f svc.yaml
```

### Option 2: Helm Template (Workaround)
Since no helm chart exists, you can convert the manifests to a helm chart template manually:

```bash
# Create a basic helm chart structure
helm create prest-chart
# Copy deployment.yaml and svc.yaml as templates
# Add values.yaml with configurable parameters
```

### Option 3: Kustomize
```bash
# In the kubernetes directory
kubectl kustomize .
```

---

## 7. Findings Summary

| Category | Finding |
|----------|---------|
| Official Helm Chart | None |
| Community Helm Chart | None |
| Kubernetes Manifests | Available (official) |
| Image Size (default) | ~433 MB |
| Image Size (noplugins) | ~54 MB |
| Resource Requests | Not set |
| Resource Limits | Not set |
| Measured CPU | ~272 mCPU |
| Measured Memory | ~12 MiB |

---

## 8. Recommendations

1. **Use noplugins image** for production: `prest/prest:v1-noplugins` (54 MB vs 433 MB)
2. **Add resource limits** to the deployment manifest
3. **Consider creating a community helm chart** if one doesn't exist
4. **Use Ingress** instead of LoadBalancer for better traffic management
5. **Enable TLS** for production deployments

---

## 9. References

- GitHub Repository: https://github.com/prest/prest
- Kubernetes Manifests: https://github.com/prest/prest/tree/main/install-manifests/kubernetes
- Docker Hub: https://hub.docker.com/r/prest/prest
- Documentation: https://docs.prestd.com/

---

*Report generated: 2026-03-06*
