---
name: k8s-skill-generator
version: 1.0.0
description: Generate customized operations Skill for K8s clusters. Triggered when user says "generate K8s Skill", "create cluster operations docs", or "generate K8s operations guide".
---

# K8s Skill Generator

Help users generate customized operations Skills for their K8s clusters.

## Trigger Conditions

When user requests:
- "Help me generate a K8s Skill"
- "Create operations docs for this cluster"
- "Generate K8s operations guide"
- "Create K8s Skill"

## Execution Flow

### Step 1: Verify Cluster Access

First, verify kubectl can access the cluster:

```bash
# Check if kubectl is available
kubectl cluster-info

# Get current context
kubectl config current-context

# List available contexts
kubectl config get-contexts
```

**If failed**: Ask the user:
- kubeconfig file location
- Which context to use
- Whether SSH jump is needed

### Step 2: Collect Cluster Information

Follow these guides to collect data:

1. **[cluster-info.md](discovery/cluster-info.md)** - Basic cluster info (version, CNI, nodes)
2. **[namespaces.md](discovery/namespaces.md)** - Namespace content analysis
3. **[monitoring.md](discovery/monitoring.md)** - Monitoring component discovery (Prometheus/Grafana)
4. **[credentials.md](discovery/credentials.md)** - Credential handling

### Step 3: Analyze Business Context

Based on collected data, analyze and confirm with user:

1. **Business purpose of each namespace**
   - Infer from name, pods, images
   - Ask user for uncertain ones

2. **Service criticality levels**
   - critical: Core business, cannot be interrupted
   - high: Important services
   - medium: Supporting services
   - low: Test/development

3. **Monitoring recommendations**
   - Recommend metrics based on business characteristics
   - Suggest alert thresholds

**Example dialogue**:
```
Agent: Found namespace `order-system` with these services:
       - order-api (image: company/order-api:v2.1)
       - order-worker (image: company/order-worker:v2.1)
       - redis (image: redis:7)
       
       Is this the order processing system? What's its criticality?

User: Yes, it's the core order system, criticality is critical
```

### Step 4: Determine Output Scope

Ask user which modules to include:

- **core**: K8s core resource operations (Deployment, StatefulSet, DaemonSet, CronJob, Service, ConfigMap, HPA...)
- **monitoring**: Monitoring operations (Prometheus queries, Grafana alerts...)
- **logging**: Logging operations (if Loki/EFK detected)
- **network**: Network policies *(planned, not yet available)*
- **storage**: Storage management *(planned, not yet available)*
- **rbac**: Permission management *(planned, not yet available)*

Default: Enable core + modules for detected components

### Step 5: Generate Skill

Use templates from [templates/](templates/) to generate files.

**Template syntax rules**:
- `{{variable}}` — Replace with actual collected value
- `{{#if condition}}...{{/if}}` — Include content only if the condition/component exists, otherwise remove the entire block
- `{{#each list}}...{{/each}}` — Expand into multiple entries, one for each item in the list
- Content between ```` ``` ```` and ```` ``` ```` inside templates is the actual output content; the outer ```` ```` ```` wrapper is just for delimiting the template

**Output structure**:
```
.cursor/skills/k8s-{context}/
├── SKILL.md                    # Main entry
└── operations/                 # Operation docs
    ├── core.md                 # Core resource operations
    ├── monitoring.md           # Monitoring operations (if enabled)
    └── ...                     # Other modules
```

### Step 6: Confirm Output Location

Default location: `.cursor/skills/k8s-{context}/`

Ask user:
- Use default location?
- Need to change Skill name?

## Security Rules

**Must follow**:

1. **No plaintext passwords**
   - Only generate commands to retrieve credentials (e.g., `kubectl get secret ...`)
   - Never write actual password values in Skill

2. **Sensitive info confirmation**
   - Show credential-related content to user for confirmation
   - Only write after user confirms

3. **Write operation confirmation**
   - Show list of files to be created before generation
   - Only create after user confirms

## Quick Reference

### One-click generation (standard cluster)

If cluster uses standard config (kubeconfig already configured), quick generate:

```bash
# Agent will execute these steps:
# 1. kubectl cluster-info
# 2. Collect info
# 3. Ask for confirmation
# 4. Generate Skill
```

### Specify context

```
User: Generate K8s Skill for production context
```

### Specify output location

```
User: Generate K8s Skill, output to ./my-skills/k8s/
```

## Related Docs

- [discovery/](discovery/) - Data collection guides
- [templates/](templates/) - Output templates
- [examples/](examples/) - Example output
