# K8s Core Resource Operations

## Deployment Management

### View Deployments

```bash
# List all
kubectl get deployments -A

# Specific namespace
kubectl get deployments -n <namespace>

# Detailed info
kubectl describe deployment <name> -n <namespace>

# Export YAML
kubectl get deployment <name> -n <namespace> -o yaml
```

### Create/Update Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <name>
  namespace: <namespace>
spec:
  replicas: 3
  selector:
    matchLabels:
      app: <name>
  template:
    metadata:
      labels:
        app: <name>
    spec:
      containers:
      - name: <name>
        image: <image>:<tag>
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Common Operations

```bash
# Scale
kubectl scale deployment <name> -n <namespace> --replicas=5

# Update image
kubectl set image deployment/<name> <container>=<image>:<tag> -n <namespace>

# Check rollout status
kubectl rollout status deployment/<name> -n <namespace>

# View history
kubectl rollout history deployment/<name> -n <namespace>

# Rollback to previous version
kubectl rollout undo deployment/<name> -n <namespace>

# Rollback to specific revision
kubectl rollout undo deployment/<name> -n <namespace> --to-revision=2
```

## Service Management

### View Services

```bash
kubectl get svc -A
kubectl get svc -n <namespace>
kubectl describe svc <name> -n <namespace>
```

### Create Service

```yaml
# ClusterIP (internal access)
apiVersion: v1
kind: Service
metadata:
  name: <name>
  namespace: <namespace>
spec:
  type: ClusterIP
  selector:
    app: <app-label>
  ports:
  - port: 80
    targetPort: 8080
```

## ConfigMap Management

### View ConfigMaps

```bash
kubectl get configmaps -n <namespace>
kubectl describe configmap <name> -n <namespace>
kubectl get configmap <name> -n <namespace> -o yaml
```

### Create ConfigMap

```bash
# From file
kubectl create configmap <name> --from-file=<path> -n <namespace>

# From literal values
kubectl create configmap <name> --from-literal=key1=value1 -n <namespace>
```

## Secret Management

### View Secrets

```bash
kubectl get secrets -n <namespace>
kubectl describe secret <name> -n <namespace>

# Decode and view (use with caution)
kubectl get secret <name> -n <namespace> -o jsonpath='{.data.<key>}' | base64 -d
```

### Create Secret

```bash
kubectl create secret generic <name> --from-literal=password=secret -n <namespace>
```

## Pod Debugging

### Logs

```bash
# View logs
kubectl logs <pod> -n <namespace>

# Follow logs
kubectl logs -f <pod> -n <namespace>

# Previous container logs
kubectl logs <pod> -n <namespace> --previous
```

### Enter Container

```bash
# Interactive shell
kubectl exec -it <pod> -n <namespace> -- /bin/sh
```

### Port Forward

```bash
# Pod port forward
kubectl port-forward <pod> <local>:<remote> -n <namespace>

# Service port forward
kubectl port-forward svc/<service> <local>:<remote> -n <namespace>
```

### Temporary Debug Pod

```bash
# Start temporary pod
kubectl run debug --rm -it --image=busybox -n <namespace> -- /bin/sh

# Debug pod with network tools
kubectl run debug --rm -it --image=nicolaka/netshoot -n <namespace> -- /bin/bash
```
