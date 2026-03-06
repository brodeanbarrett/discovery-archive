# Temporal Helm Chart Discovery Report

## Date
2026-03-06

## Executive Summary
This report documents the discovery, deployment, and resource analysis of the Temporal workflow orchestration engine using Helm charts on a local K3s cluster.

## 1. Helm Chart Sources

### Official Helm Chart
- **Repository**: https://github.com/temporalio/helm-charts
- **Chart Name**: temporal
- **Latest Version**: 1.0.0-rc.2 (Temporal Server 1.29.2)
- **Status**: RC (Release Candidate)
- **License**: MIT

### Community Alternatives
1. **alexandermarston/temporal-helm-charts** - Community-maintained fork
2. **discord/temporalio-helm-charts** - Discord's fork for production use
3. **ComposioHQ/temporalio-helm-charts** - Alternative community chart

### Note on Helm Repository
The official Helm repository (https://helm.temporal.io) was not accessible during this deployment. The chart was deployed by cloning the GitHub repository directly.

## 2. Deployment Configuration

### Prerequisites
- K3s v1.34.4+k3s1
- Helm v3.20.0
- kubectl

### Infrastructure Components
| Component | Chart Version | App Version |
|-----------|---------------|-------------|
| Temporal Server | 1.0.0-rc.2 | 1.29.2 |
| PostgreSQL | 18.5.2 | 18.3.0 |

### Persistence Configuration
- **Default Store**: PostgreSQL (plugin: postgres12)
  - Database: temporal
  - Host: postgresql.temporal.svc.cluster.local:5432
- **Visibility Store**: PostgreSQL (plugin: postgres12)
  - Database: temporal_visibility
  - Host: postgresql.temporal.svc.cluster.local:5432

### Services Deployed
| Service | Type | Ports |
|---------|------|-------|
| temporal-frontend | ClusterIP | 7233/TCP, 7243/TCP |
| temporal-internal-frontend | ClusterIP | 7236/TCP, 7246/TCP |
| temporal-web | ClusterIP | 8080/TCP |
| postgresql | ClusterIP | 5432/TCP |

### Pods Deployed
| Pod Name | Container Image | Replicas |
|----------|-----------------|----------|
| temporal-frontend | temporalio/server:1.29.2 | 1 |
| temporal-history | temporalio/server:1.29.2 | 1 |
| temporal-matching | temporalio/server:1.29.2 | 1 |
| temporal-worker | temporalio/server:1.29.2 | 1 |
| temporal-web | temporalio/ui:2.42.1 | 1 |
| temporal-admin-tools | temporalio/admin-tools:1.29.1-tctl-1.18.4-cli-1.5.0 | 1 |
| postgresql | bitnami/postgresql:latest | 1 |

## 3. Resource Analysis

### Container Images Used
| Image | Tag | Purpose |
|-------|-----|---------|
| temporalio/server | 1.29.2 | Core Temporal services |
| temporalio/ui | 2.42.1 | Web UI |
| temporalio/admin-tools | 1.29.1-tctl-1.18.4-cli-1.5.0 | Admin tools & schema init |
| bitnami/postgresql | latest | Database |

### Resource Limits and Requests

#### PostgreSQL
| Resource | Requests | Limits |
|----------|----------|--------|
| CPU | 100m | 150m |
| Memory | 128Mi | 192Mi |
| Ephemeral Storage | 50Mi | 2Gi |

#### Temporal Services (Frontend, History, Matching, Worker)
- **No resource limits or requests configured** (default empty `{}`)
- This is consistent with the chart's default behavior which leaves resource configuration to the user

#### Temporal Web UI
- **No resource limits or requests configured**

#### Admin Tools
- **No resource limits or requests configured**

### Actual Resource Usage (Metrics)

| Pod | CPU | Memory |
|-----|-----|--------|
| temporal-frontend | 10m | 34Mi |
| temporal-history | 11m | 38Mi |
| temporal-matching | 11m | 39Mi |
| temporal-worker | 10m | 30Mi |
| temporal-web | 1m | 6Mi |
| temporal-admin-tools | 3m | 0Mi |
| postgresql | 12m | 77Mi |

**Total Memory Usage**: ~224Mi (excluding schema job which completed)

## 4. Helm Chart Observations

### Pros
1. Comprehensive configuration options for persistence (SQL, Cassandra, Elasticsearch)
2. Supports TLS configuration
3. Built-in metrics with Prometheus annotations
4. ServiceMonitor support for Prometheus Operator
5. Configurable security contexts

### Cons
1. **No embedded database** - Requires external PostgreSQL, MySQL, or Cassandra
2. **Elasticsearch required for advanced visibility** - SQL visibility is limited
3. No default resource limits - requires explicit configuration
4. Chart version is RC (1.0.0-rc.2) - not yet stable
5. Official Helm repository was inaccessible during this deployment

### Configuration Complexity
- Medium-High: Requires understanding of Temporal architecture
- Must configure persistence layer explicitly
- Schema initialization handled automatically via jobs

## 5. Recommendations for Production

1. **Resources**: Explicitly set CPU and memory limits for all Temporal services
2. **Persistence**: Use managed database services (e.g., AWS RDS, Cloud SQL) for production
3. **Elasticsearch**: Deploy Elasticsearch for advanced visibility features
4. **Security**: Configure TLS and authentication
5. **Monitoring**: Enable Prometheus ServiceMonitor for observability
6. **High Availability**: Increase replicaCount to 3+ for production clusters
7. **Storage**: Use persistent volume claims with appropriate storage class

## 6. Cleanup Performed

After deployment and analysis, the following cleanup was performed:
- Helm releases uninstalled: temporal, postgresql
- Namespace deleted: temporal
- All associated pods, services, and persistent volume claims removed

## 7. References

- Official Chart Repository: https://github.com/temporalio/helm-charts
- Temporal Documentation: https://docs.temporal.io/
- Chart Values Reference: https://github.com/temporalio/helm-charts/blob/main/charts/temporal/values.yaml
