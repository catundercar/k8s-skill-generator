---
name: k8s-production
description: Production K8s cluster operations. Triggered when user mentions K8s, kubectl, Pod, Deployment, order system, user service, monitoring, alerts.
---

# production Cluster Operations

Production K8s cluster hosting core business systems.

## Security Rules (Must Follow)

**All create/update/delete operations require human confirmation:**

```
AI: I will perform the following operation:
    - Operation type: [Create/Update/Delete] [Resource type]
    - Namespace: [namespace]
    - Summary: [brief description]
    
    Confirm execution? [Y/n]
```

- **Requires confirmation**: `kubectl apply/create/delete/patch`, API write operations
- **No confirmation needed**: `kubectl get/describe/logs`, status checks

## Cluster Overview

| Item | Value |
|------|-------|
| Context | `production` |
| K8s Version | v1.28.3 |
| Nodes | 5 (3 master + 2 worker) |
| CNI | Cilium |
| Ingress | nginx |

## Cluster Access

```bash
# Switch context
kubectl config use-context production

# Verify connection
kubectl cluster-info
kubectl get nodes
```

## Namespaces

### kube-system

**Purpose**: Kubernetes system components

### monitoring

**Purpose**: Monitoring infrastructure (Prometheus + Grafana + AlertManager)
**Criticality**: high

**Services**:
| Service | Type | Description |
|---------|------|-------------|
| `prometheus-operated` | Monitoring | Metrics collection and storage |
| `grafana` | Visualization | Dashboards and alerting |
| `alertmanager-operated` | Alerting | Alert routing and notification |

### order-system

**Purpose**: Core order system, handles all order creation, payment, and status transitions
**Criticality**: critical

**Services**:
| Service | Type | Description |
|---------|------|-------------|
| `order-api` | REST API | Order service entry, port 8080 |
| `order-worker` | Async Task | Order status change processing |
| `order-scheduler` | Scheduled Task | Timeout order handling |
| `redis` | Cache/Queue | Order cache and task queue |

### user-system

**Purpose**: User center, manages user registration, login, profile maintenance
**Criticality**: critical

**Services**:
| Service | Type | Description |
|---------|------|-------------|
| `user-api` | REST API | User service entry |
| `postgres` | Database | User data storage |

### logging

**Purpose**: Logging system (Loki)
**Criticality**: medium

**Services**:
| Service | Type | Description |
|---------|------|-------------|
| `loki` | Log Storage | Log aggregation and query |
| `promtail` | Log Collection | Container log collection |

## Monitoring

### Components

| Component | Namespace | Service | Port |
|-----------|-----------|---------|------|
| Prometheus | monitoring | prometheus-operated | 9090 |
| Grafana | monitoring | grafana | 80 |
| AlertManager | monitoring | alertmanager-operated | 9093 |

### Grafana Access

```bash
# Port forward
kubectl port-forward -n monitoring svc/grafana 3000:80
# Access http://localhost:3000
```

- Username: `admin`
- Password:
  ```bash
  kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d
  ```

### Prometheus Query

```bash
# Port forward
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090

# Query example
curl -s "http://localhost:9090/api/v1/query" --data-urlencode 'query=up'
```

## Quick Command Reference

```bash
# View all pods
kubectl get pods -A

# View resources in namespace
kubectl get all -n <namespace>

# Pod logs (follow)
kubectl logs -f <pod> -n <namespace>

# Enter container
kubectl exec -it <pod> -n <namespace> -- /bin/sh

# Port forward
kubectl port-forward svc/<service> <local>:<remote> -n <namespace>
```

> For complete operations guide, see the Operation Docs below.

## Operation Docs

- [K8s Core Resource Operations](operations/core.md)
- [Monitoring Operations](operations/monitoring.md)
- [Logging Operations](operations/logging.md)
