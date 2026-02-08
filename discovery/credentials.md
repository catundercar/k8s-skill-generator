# Credential Handling

## Core Principle

**Never store plaintext passwords in generated Skills**

Only generate commands to retrieve credentials, let users get them when needed.

## Credential Discovery

### 1. Common Component Credential Locations

| Component | Secret Name Pattern | Key |
|-----------|---------------------|-----|
| Grafana | `grafana`, `{release}-grafana` | `admin-password` |
| ArgoCD | `argocd-initial-admin-secret` | `password` |
| Harbor | `harbor-core` | `HARBOR_ADMIN_PASSWORD` |
| MinIO | `minio` | `accesskey`, `secretkey` |
| PostgreSQL | `{name}-postgresql` | `postgres-password` |
| MySQL | `{name}-mysql` | `mysql-root-password` |
| Redis | `{name}-redis` | `redis-password` |

### 2. Discovery Flow

```bash
# List secrets in namespace
kubectl get secrets -n <namespace>

# View secret keys (not values)
kubectl get secret <name> -n <namespace> -o jsonpath='{.data}' | jq 'keys'
```

### 3. Recording Format

```yaml
credentials:
  - component: grafana
    namespace: monitoring
    secret_name: grafana
    key: admin-password
    command: "kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d"
  
  - component: argocd
    namespace: argocd
    secret_name: argocd-initial-admin-secret
    key: password
    command: "kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d"
```

## Generated Skill Format

### Method 1: Direct Command

````markdown
### Grafana Credentials

- Username: `admin`
- Password:
  ```bash
  kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d
  ```
````

### Method 2: Environment Variable Setup

````markdown
### Set Credential Environment Variables

```bash
# Grafana
export GRAFANA_PASSWORD=$(kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d)

# Usage
curl -u admin:$GRAFANA_PASSWORD https://grafana.example.com/api/health
```
````

### Method 3: Script Wrapper

````markdown
### Credential Retrieval Script

```bash
#!/bin/bash
# get-credentials.sh

case "$1" in
  grafana)
    kubectl get secret -n monitoring grafana -o jsonpath='{.data.admin-password}' | base64 -d
    ;;
  argocd)
    kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d
    ;;
  *)
    echo "Usage: $0 {grafana|argocd}"
    ;;
esac
```
````

## External Credential Handling

For credentials not in K8s Secrets (e.g., external service API keys), ask user:

```
Agent: Detected external service configuration, the following credentials cannot be auto-discovered:
       - External Webhook URL
       - Slack Token (if using Slack alerts)
       
       Are these credentials stored in environment variables?
       If yes, please provide the variable names, I'll use ${VAR_NAME} reference in the Skill.

User: Webhook URL is in ALERT_WEBHOOK_URL environment variable
```

Generated:
```markdown
### External Service Credentials

| Service | Credential Source |
|---------|-------------------|
| Alert Webhook | `${ALERT_WEBHOOK_URL}` |
```

## Security Checklist

Before generating Skill, verify:

- [ ] No plaintext passwords
- [ ] No API keys
- [ ] No tokens
- [ ] All credentials are retrieved via commands or environment variable references
