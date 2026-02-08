# Logging Operations

## Loki Log Query

### Access Method

```bash
# Port forward
kubectl port-forward -n logging svc/loki 3100:3100
```

### LogQL Query

#### Basic Queries

```bash
# Query logs for specific namespace
curl -s "http://localhost:3100/loki/api/v1/query_range" \
  --data-urlencode 'query={namespace="order-system"}' \
  --data-urlencode 'limit=100'

# Query specific pod
curl -s "http://localhost:3100/loki/api/v1/query_range" \
  --data-urlencode 'query={namespace="order-system", pod=~"order-api.*"}' \
  --data-urlencode 'limit=100'
```

#### Filter Queries

```bash
# Contains keyword
curl -s "http://localhost:3100/loki/api/v1/query_range" \
  --data-urlencode 'query={namespace="order-system"} |= "error"' \
  --data-urlencode 'limit=100'

# Regex match
curl -s "http://localhost:3100/loki/api/v1/query_range" \
  --data-urlencode 'query={namespace="order-system"} |~ "error|warn"' \
  --data-urlencode 'limit=100'
```

### Common Query Examples

| Purpose | LogQL |
|---------|-------|
| All error logs | `{namespace="order-system"} \|= "error"` |
| Order API logs | `{namespace="order-system", app="order-api"}` |
| OOM events | `{namespace="order-system"} \|= "OOMKilled"` |

## kubectl Log Query

```bash
# View pod logs
kubectl logs <pod> -n <namespace>

# Follow logs
kubectl logs -f <pod> -n <namespace>

# Last 100 lines
kubectl logs --tail=100 <pod> -n <namespace>

# Last 30 minutes
kubectl logs --since=30m <pod> -n <namespace>

# Previous container logs
kubectl logs --previous <pod> -n <namespace>

# Select by label
kubectl logs -l app=order-api -n order-system
```

## Log Troubleshooting

### Pod Startup Failure

```bash
# View pod status
kubectl describe pod <pod> -n <namespace>

# View events
kubectl get events -n <namespace> --sort-by='.lastTimestamp' | tail -20

# View previous container logs
kubectl logs --previous <pod> -n <namespace>
```

### Application Errors

```bash
# Search for error keyword
kubectl logs <pod> -n <namespace> | grep -i error

# Monitor errors in real-time
kubectl logs -f <pod> -n <namespace> | grep -i error
```
