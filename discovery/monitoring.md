# Monitoring Component Discovery

## Goal

Detect monitoring components in the cluster (Prometheus, Grafana, AlertManager, etc.) and get access information.

## Detection Flow

### 1. Detect Prometheus Operator

```bash
# Check if CRDs exist
kubectl get crd | grep -E "prometheus|servicemonitor|alertmanager"

# If exists, get Prometheus instances
kubectl get prometheus -A
kubectl get alertmanager -A
```

If CRD detected: `prometheuses.monitoring.coreos.com`, it's using Prometheus Operator.

### 2. Detect Prometheus Service

```bash
# Find by label
kubectl get svc -A -l "app.kubernetes.io/name=prometheus"
kubectl get svc -A -l "app=prometheus"

# Or by name
kubectl get svc -A | grep -i prometheus
```

Record:
- namespace
- service name
- port

### 3. Detect Grafana

```bash
# By label
kubectl get svc -A -l "app.kubernetes.io/name=grafana"

# By name
kubectl get svc -A | grep -i grafana
```

### 4. Get Grafana Credentials

```bash
# Common Secret name patterns
kubectl get secret -n <namespace> | grep -i grafana

# Get password (don't output actual value, only record the command)
# kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d
```

**Note**: Don't execute the password retrieval command, only record the command itself for later inclusion in Skill.

### 5. Detect AlertManager

```bash
kubectl get svc -A -l "app.kubernetes.io/name=alertmanager"
kubectl get svc -A | grep -i alertmanager
```

### 6. Detect Loki (Logging)

```bash
kubectl get svc -A -l "app.kubernetes.io/name=loki"
kubectl get svc -A | grep -i loki
```

### 7. Detect Ingress Exposure

```bash
# Check if monitoring services are exposed via Ingress
kubectl get ingress -A | grep -E "prometheus|grafana|alertmanager"
```

## Output Format

```yaml
monitoring:
  type: "prometheus-operator"  # prometheus-operator | standalone | none
  
  prometheus:
    found: true
    namespace: "monitoring"
    service: "prometheus-operated"
    port: 9090
    ingress: null  # or "https://prometheus.example.com"
  
  grafana:
    found: true
    namespace: "monitoring"
    service: "grafana"
    port: 80
    ingress: "https://grafana.example.com"
    credentials:
      user: "admin"
      password_cmd: "kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d"
  
  alertmanager:
    found: true
    namespace: "monitoring"
    service: "alertmanager-operated"
    port: 9093
  
  loki:
    found: true
    namespace: "logging"
    service: "loki"
    port: 3100

logging:
  type: "loki"  # loki | efk | none
  # ...
```

## Access Method Generation

Based on detection results, generate access methods:

### If Ingress Exists

````markdown
## Grafana Access

- URL: https://grafana.example.com
- Username: admin
- Password: `kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d`
````

### If Only Service Exists

````markdown
## Grafana Access

```bash
# Port forward
kubectl port-forward -n monitoring svc/grafana 3000:80

# Access http://localhost:3000
# Username: admin
# Password: kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d
```
````

## Show to User

```
Monitoring Components Detected:

✓ Prometheus Operator (monitoring/prometheus-operated:9090)
✓ Grafana (monitoring/grafana:80)
  - Ingress: https://grafana.example.com
  - Credentials: Secret monitoring/grafana
✓ AlertManager (monitoring/alertmanager-operated:9093)
✓ Loki (logging/loki:3100)

Any additions or modifications needed?
```
