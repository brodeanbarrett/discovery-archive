# Discovery Report: Gitea Kubernetes Deployment

## Overview

This discovery investigated how to deploy Gitea to Kubernetes, specifically:
1. Using an official Helm chart
2. Supporting CI/CD pipelines triggered by repository pushes
3. Enabling Gitea to trigger Kubernetes deployments/pod management

## Key Findings

### 1. Official Gitea Helm Chart

Gitea provides an **official Helm chart** maintained at `gitea/helm-chart` (also known as `gitea-charts`).

**Installation:**
```bash
helm repo add gitea-charts https://dl.gitea.com/charts/
helm install gitea gitea-charts/gitea
```

The chart manages:
- Gitea deployment
- Built-in PostgreSQL or MySQL database (optional)
- Memcached for caching
- Ingress configuration
- Persistent storage
- Health checks (`/api/healthz`)

### 2. Gitea Actions (CI/CD)

Gitea includes **Gitea Actions** (similar to GitHub Actions), which provides CI/CD pipeline functionality. This requires:

1. **Enable Actions in Gitea config** - Via the `actions` section in values.yaml
2. **Deploy act_runner** - Gitea's action runner that executes workflows

The `gitea/helm-actions` chart exists specifically for deploying Gitea Actions runners on Kubernetes.

### 3. Kubernetes as CI/CD Runner Target

Gitea Actions can target Kubernetes pods as runners. The `gitea/act_runner` project provides Kubernetes examples:

- **dind-docker.yaml** - Docker-in-Docker deployment (requires privileged security context)
- **rootless-docker.yaml** - Rootless Docker deployment

These allow CI/CD jobs to run inside Kubernetes pods, enabling:
- Building container images
- Deploying to the same Kubernetes cluster
- Running any Kubernetes-native CI/CD workflows

### 4. Triggering Kubernetes Deployments from Gitea

The architecture for Gitea triggering Kubernetes deployments works as follows:

```
User Push → Gitea Repository → Gitea Actions → act_runner (K8s pod) → kubectl apply/delete
```

**Implementation options:**

1. **Via Gitea Actions workflow** - Use `kubectl` in action workflows to interact with Kubernetes API
2. **Via kubeconfig stored in Gitea** - Actions can mount a kubeconfig secret to authenticate with the Kubernetes API
3. **Via service account tokens** - Runner pods can use Kubernetes service accounts with appropriate RBAC

Example workflow snippet:
```yaml
- name: Deploy to Kubernetes
  run: |
    kubectl set image deployment/myapp myapp=myregistry/myapp:${{ github.sha }}
    kubectl rollout status deployment/myapp
```

## Repository Structure

The Gitea repository (`gitea/go-gitea`) does not contain Kubernetes deployment files directly. Instead:

- **Official Helm chart**: Hosted in separate repo `gitea/helm-gitea` (or `gitea-charts`)
- **Actions runner**: Hosted in `gitea/act_runner` with Kubernetes examples
- **Documentation**: Available at `docs.gitea.com/installation/install-on-kubernetes`

## Recommendations

### Deployment Architecture

1. **Use the official Helm chart** for Gitea installation:
   ```bash
   helm repo add gitea-charts https://dl.gitea.com/charts/
   helm install gitea gitea-charts/gitea -n gitea --create-namespace
   ```

2. **Enable Gitea Actions** via values.yaml:
   ```yaml
   gitea:
     config:
       actions:
         ENABLED: true
   ```

3. **Deploy act_runner** using either:
   - The `gitea/helm-actions` chart
   - The provided Kubernetes manifests from `gitea/act_runner/examples/kubernetes`

4. **For Kubernetes deployment triggers**:
   - Store kubeconfig as a Kubernetes Secret
   - Mount it in runner pods via the runner's Helm chart
   - Use RBAC to restrict runner permissions to specific namespaces
   - Create Actions workflows that use `kubectl` to apply/delete deployments

### Security Considerations

- **Docker-in-Docker (dind)** requires privileged containers - consider rootless mode for better security
- **RBAC**: Create dedicated service accounts for runners with minimal required permissions
- **Network policies**: Restrict runner pods from accessing unrelated services
- **Secrets management**: Use Kubernetes Secrets or external secret operators for sensitive data

### High Availability

The Gitea Helm chart supports:
- External database (PostgreSQL/MySQL)
- External Memcached
- Horizontal scaling with multiple replicas (requires external session storage)
- Persistent storage with PVCs

## Uncertainties

- **Specific Helm values for Actions**: While the `actions` config section exists, the exact values.yaml structure should be verified against the latest Helm chart documentation
- **Rootless Docker in Kubernetes**: While supported, some features may have limitations
- **Runner scaling**: How auto-scaling works with the Kubernetes runner executor requires further investigation

## Additional Resources

- Gitea Helm Chart: https://gitea.com/gitea/helm-gitea
- Gitea Actions: https://docs.gitea.com/actions/overview
- act_runner Kubernetes Examples: https://gitea.com/gitea/act_runner/src/branch/main/examples/kubernetes
- helm-actions Chart: https://gitea.com/gitea/helm-actions
