# RAG Escalation Automation Collection

Ansible collection for automating RAG Escalation Lab workloads on OpenShift AI.

## Roles

### `ocp4_workload_rag_escalation_workbench`

Deploys a JupyterHub workbench (Notebook CR) in an OpenShift AI Data Science Project with a pre-loaded git repository.

**Features**:
- Creates Jupyter workbench with GPU access
- Clones specified git repository into workspace using init container
- Configures resource limits and GPU allocation
- Targets specific GPU nodes via node selectors and tolerations
- Exports workbench URL to user data

**Required Variables**:
- `ocp4_workload_rag_escalation_workbench_namespace` - Target namespace (Data Science Project)
- `ocp4_workload_rag_escalation_workbench_git_repo` - Git repository URL to clone
- `ocp4_workload_rag_escalation_workbench_tenant_label` - Tenant label for node selector (e.g., `user-abc123`)

**Optional Variables**:
- `ocp4_workload_rag_escalation_workbench_git_branch` - Git branch (default: `main`)
- `ocp4_workload_rag_escalation_workbench_notebook_image` - JupyterLab image
- `ocp4_workload_rag_escalation_workbench_storage_size` - PVC size (default: `20Gi`)
- `ocp4_workload_rag_escalation_workbench_cpu_limit` - CPU limit (default: `8`)
- `ocp4_workload_rag_escalation_workbench_memory_limit` - Memory limit (default: `24Gi`)
- `ocp4_workload_rag_escalation_workbench_gpu_count` - GPU count (default: `1`)

See `SIMPLIFIED.md` for design rationale and `VARIABLES.md` for complete reference.

## Installation

```yaml
# requirements.yml
collections:
- name: https://github.com/treddy08/rag-escalation-automation.git
  type: git
  version: main
```

## Usage Example

```yaml
- name: Deploy RAG Escalation workbench
  hosts: localhost
  tasks:
  - name: Include workbench role
    include_role:
      name: agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench
    vars:
      ocp4_workload_rag_escalation_workbench_namespace: "user-abc123-dsproject"
      ocp4_workload_rag_escalation_workbench_git_repo: "https://github.com/FrankLaVigne/EscalationLab"
      ocp4_workload_rag_escalation_workbench_tenant_label: "user-abc123"
      # Optional: override defaults
      # ocp4_workload_rag_escalation_workbench_git_branch: "main"
      # ocp4_workload_rag_escalation_workbench_storage_size: 20Gi
      # ocp4_workload_rag_escalation_workbench_gpu_count: "1"
```

## License

Apache-2.0

## Author

Tyrell Reddy (treddy@redhat.com)
