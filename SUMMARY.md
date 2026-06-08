# RAG Escalation Automation - Collection Summary

## Quick Stats

- **Collection**: `agnosticd.rag_escalation_automation`
- **Role**: `ocp4_workload_rag_escalation_workbench`
- **Required Variables**: 3
- **Optional Variables**: 7
- **Task Files**: 3 (main, workload, remove_workload)
- **Templates**: 2 (PVC, Notebook)

## Purpose

Deploy a Jupyter workbench in OpenShift AI with a pre-loaded git repository for RAG/AI labs.

## Minimal Configuration

```yaml
# In your catalog item (e.g., rag-escalation-lab-tenant/common.yaml)

requirements_content:
  collections:
  - name: https://github.com/treddy08/rag-escalation-automation.git
    type: git
    version: main

workloads:
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench

# Configuration (3 required variables)
ocp4_workload_rag_escalation_workbench_namespace: "user-abc123-dsproject"
ocp4_workload_rag_escalation_workbench_git_repo: "https://github.com/FrankLaVigne/EscalationLab"
ocp4_workload_rag_escalation_workbench_tenant_label: "user-abc123"
```

## What Gets Created

| Resource | Name | Details |
|----------|------|---------|
| **Notebook** | `rag-escalation-workbench` | JupyterLab workbench with GPU access |
| **PVC** | `rag-escalation-workbench-pvc` | 20Gi persistent storage |
| **Git Repo** | `/opt/app-root/src/EscalationLab` | Pre-cloned from specified URL |
| **Resources** | 8 CPU, 24Gi RAM, 1 GPU | Configurable via variables |
| **OAuth** | Enabled | OpenShift authentication |

## Design Principles

1. **Hard-code what never changes** - Fixed names, paths, OAuth settings
2. **Default what rarely changes** - Resources, storage size, image
3. **Require only what varies** - Namespace and git repo
4. **Single responsibility** - Deploy workbench with git repo, nothing more
5. **No pre/post hooks** - Linear flow in one file

## Files

```
rag-escalation-automation/
├── galaxy.yml
├── README.md
├── VARIABLES.md
├── INTEGRATION_GUIDE.md
├── SIMPLIFIED.md
├── SUMMARY.md (this file)
└── roles/ocp4_workload_rag_escalation_workbench/
    ├── defaults/main.yml
    ├── meta/main.yml
    ├── tasks/
    │   ├── main.yml           # Entry point
    │   ├── workload.yml       # Validate → Create → Wait → Export
    │   └── remove_workload.yml # Cleanup
    └── templates/
        ├── pvc.yml.j2         # Storage
        └── notebook.yml.j2    # Workbench with init container
```

## Key Features

### Init Container Pattern
- Runs **before** JupyterLab starts
- Clones git repository
- Installs optional pip packages
- Idempotent (won't re-clone if exists)

### GPU Scheduling
Always deploys on tenant's GPU node:
- GPU resource request: `nvidia.com/gpu: 1`
- Toleration: `nvidia.com/gpu=true:NoSchedule`
- Node selector: `tenant: <tenant_label>` (targets tenant's specific GPU node)
- Ensures multi-tenant isolation

### User Data Export
- `rag_escalation_workbench_url` - Direct link to workbench
- `rag_escalation_workbench_namespace` - Deployment namespace
- `rag_escalation_git_repo` - Repository URL

## Documentation

- **README.md** - Quick start and installation
- **VARIABLES.md** - Complete variable reference
- **INTEGRATION_GUIDE.md** - Step-by-step tenant CI integration
- **SIMPLIFIED.md** - Design decisions and rationale
- **SUMMARY.md** - This file

## Usage in Tenant CI

The typical use case is a multi-tenant GPU lab where:
1. Cluster component provisions shared OpenShift cluster with GPU nodes
2. Tenant component (using this role) gives each user:
   - Isolated Data Science Project namespace
   - Jupyter workbench with lab materials pre-loaded
   - GPU access for training/inference
   - Keycloak authentication

This role handles step #2 - the per-user workbench deployment.

## Next Steps

1. Push to GitHub: `https://github.com/treddy08/rag-escalation-automation`
2. Integrate into `rag-escalation-lab-tenant` catalog item
3. Test deployment on cluster
4. Update info message template with workbench URL

## License

Apache-2.0

## Author

Tyrell Reddy (treddy@redhat.com)
