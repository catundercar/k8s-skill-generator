# Namespace Analysis

## Goal

Collect all namespaces and their contents, analyze business purpose.

## Commands to Execute

### 1. Get All Namespaces

```bash
kubectl get namespaces -o json
```

### 2. Collect Resources for Each Namespace

```bash
# Get main resources
kubectl get deployments,statefulsets,daemonsets,services -n <namespace> -o wide

# Get pods and images
kubectl get pods -n <namespace> -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[*].image

# Get ConfigMaps and Secrets (names only)
kubectl get configmaps,secrets -n <namespace> -o name
```

### 3. Get Resource Labels

```bash
kubectl get deployments -n <namespace> -o jsonpath='{range .items[*]}{.metadata.name}: {.metadata.labels}{"\n"}{end}'
```

## Analysis Rules

### System Namespaces (Skip detailed analysis)

| Namespace | Purpose |
|-----------|---------|
| `kube-system` | K8s system components |
| `kube-public` | Public resources |
| `kube-node-lease` | Node leases |
| `default` | Default namespace |

### Infrastructure Namespaces (Auto-identify)

| Keyword | Inferred Purpose |
|---------|------------------|
| `monitoring` / `observability` | Monitoring system |
| `logging` / `elastic` / `efk` | Logging system |
| `ingress` / `traefik` | Traffic ingress |
| `istio` / `linkerd` | Service mesh |
| `cert-manager` | Certificate management |
| `argocd` / `flux` | GitOps |

### Business Namespaces (Need user confirmation)

For other namespaces, analyze and ask user:

1. **Infer from name**
   - `order-system` → Likely order system
   - `user-service` → Likely user service
   - `payment` → Likely payment service

2. **Infer from content**
   - Deployments with `api` suffix → API services
   - Contains `worker` / `consumer` → Async task processing
   - Contains `redis` / `mysql` / `postgres` → Data storage

3. **Confirm with user**
   ```
   Agent: Found namespace `order-system` containing:
          - Deployments: order-api, order-worker, order-scheduler
          - Services: order-api (ClusterIP:8080)
          - StatefulSets: redis
          
          Inference: This is an order processing system?
          Please confirm or provide more accurate description:
   
   User: Yes, it's the core order system handling all order creation and status transitions
   ```

## Output Format

```yaml
namespaces:
  # System level
  - name: kube-system
    type: system
    purpose: "Kubernetes system components"
    skip_detail: true
  
  # Infrastructure
  - name: monitoring
    type: infrastructure
    purpose: "Monitoring system (Prometheus + Grafana)"
    components:
      - prometheus
      - grafana
      - alertmanager
  
  # Business
  - name: order-system
    type: business
    purpose: "Core order system, handles order creation and status transitions"
    criticality: critical
    services:
      - name: order-api
        type: "REST API"
        port: 8080
      - name: order-worker
        type: "Async task processing"
      - name: redis
        type: "Cache/Queue"
    user_confirmed: true
```

## Criticality Assessment

Ask user to assess criticality for each business namespace:

| Level | Description | Examples |
|-------|-------------|----------|
| critical | Core business, interruption directly affects revenue | Orders, Payments |
| high | Important services, affects user experience | User center, Notifications |
| medium | Supporting services, has alternatives | Reports, Admin dashboard |
| low | Non-critical services | Test environments, Tools |
