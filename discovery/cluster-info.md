# Cluster Basic Info Collection

## Goal

Collect basic K8s cluster information including version, nodes, CNI, etc.

## Commands to Execute

### 1. Cluster Version

```bash
kubectl version -o json
```

Extract: `serverVersion.gitVersion`

### 2. Node Information

```bash
kubectl get nodes -o wide
```

Collect:
- Node count
- Node roles (master/worker)
- Node IPs
- OS/kernel version

### 3. CNI Detection

```bash
# Check CNI components in kube-system
kubectl get pods -n kube-system -o custom-columns=NAME:.metadata.name | grep -E "cilium|calico|flannel|weave|canal"

# Or via DaemonSet
kubectl get daemonsets -n kube-system
```

Common CNI identification:
- `cilium` → Cilium
- `calico-node` → Calico
- `kube-flannel` → Flannel
- `weave-net` → Weave
- `aws-node` → AWS VPC CNI

### 4. Ingress Controller Detection

```bash
# Check ingress related namespaces
kubectl get namespaces | grep -E "ingress|traefik|istio"

# Check IngressClass
kubectl get ingressclass
```

### 5. Storage Classes

```bash
kubectl get storageclasses
```

## Output Format

After collection, organize into this structure:

```yaml
cluster:
  context: "production"
  version: "1.28.3"
  nodes:
    total: 5
    masters: 3
    workers: 2
    list:
      - name: "master-1"
        role: "master"
        ip: "10.0.0.11"
      - name: "worker-1"
        role: "worker"
        ip: "10.0.0.21"
  cni: "cilium"
  ingress: "nginx"
  storage_classes:
    - name: "standard"
      provisioner: "kubernetes.io/aws-ebs"
      default: true
```

## Show to User

After collection, show summary to user:

```
Cluster Info:
- Context: production
- Version: v1.28.3
- Nodes: 5 (3 master + 2 worker)
- CNI: Cilium
- Ingress: nginx
- Storage: standard (AWS EBS)

Is this correct?
```
