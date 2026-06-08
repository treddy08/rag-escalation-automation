# Role Variable Reference

## ocp4_workload_rag_escalation_workbench

### Required Variables (MUST be passed in)

| Variable | Description | Example |
|----------|-------------|---------|
| `ocp4_workload_rag_escalation_workbench_namespace` | Target Data Science Project namespace | `user-abc123-dsproject` |
| `ocp4_workload_rag_escalation_workbench_git_repo` | Git repository URL to clone | `https://github.com/FrankLaVigne/EscalationLab` |
| `ocp4_workload_rag_escalation_workbench_tenant_label` | Tenant label value for node selector | `user-abc123` |

**Note**: The `tenant_label` is used for the node selector: `tenant: <tenant_label>`. This pins the workbench to the tenant's specific GPU node.

### Optional Variables (Rarely Need to Change)

| Variable | Description | Default |
|----------|-------------|---------|
| `ocp4_workload_rag_escalation_workbench_git_branch` | Git branch to clone | `main` |
| `ocp4_workload_rag_escalation_workbench_notebook_image` | JupyterLab container image | `quay.io/opendatahub/workbench-images:jupyter-datascience-ubi9-python-3.11-20250101` |
| `ocp4_workload_rag_escalation_workbench_storage_size` | PVC storage size | `20Gi` |
| `ocp4_workload_rag_escalation_workbench_cpu_request` | CPU request | `2` |
| `ocp4_workload_rag_escalation_workbench_cpu_limit` | CPU limit | `8` |
| `ocp4_workload_rag_escalation_workbench_memory_request` | Memory request | `16Gi` |
| `ocp4_workload_rag_escalation_workbench_memory_limit` | Memory limit | `24Gi` |
| `ocp4_workload_rag_escalation_workbench_gpu_count` | Number of GPUs to request | `1` |
| `ocp4_workload_rag_escalation_workbench_pip_packages` | Additional pip packages to install | `[]` |
| `ocp4_workload_rag_escalation_workbench_ready_timeout` | Timeout for notebook ready (seconds) | `600` |
| `ocp4_workload_rag_escalation_workbench_export_user_data` | Export URL to agnosticd_user_data | `true` |

### Hard-Coded Values (Not Configurable)

The following are fixed in the role and cannot be overridden:

| What | Value | Reason |
|------|-------|--------|
| Workbench name | `rag-escalation-workbench` | Fixed name, no variation needed |
| PVC name | `rag-escalation-workbench-pvc` | Matches workbench name |
| Git target directory | `EscalationLab` | Fixed repository name |
| Working directory | `/opt/app-root/src/EscalationLab` | Always the same |
| OAuth enabled | `true` | Always use OpenShift OAuth |
| GPU toleration | `nvidia.com/gpu=true:NoSchedule` | Always applied |
| Node selector | `tenant: <tenant_label>` | Always applied - targets tenant's GPU node |

### Variables Exported to agnosticd_user_data

When `ocp4_workload_rag_escalation_workbench_export_user_data: true`:

- `rag_escalation_workbench_url` - Workbench access URL
- `rag_escalation_workbench_namespace` - Namespace where deployed
- `rag_escalation_git_repo` - Git repository URL

## Example: Tenant CI Integration

```yaml
# In rag-escalation-lab-tenant/common.yaml
workloads:
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench

# Required configuration (3 variables)
ocp4_workload_rag_escalation_workbench_namespace: "{{ ocp4_workload_tenant_keycloak_username }}-dsproject"
ocp4_workload_rag_escalation_workbench_git_repo: "https://github.com/FrankLaVigne/EscalationLab"
ocp4_workload_rag_escalation_workbench_tenant_label: "{{ ocp4_workload_tenant_keycloak_username }}"

# Optional: customize if needed (these are the defaults)
# ocp4_workload_rag_escalation_workbench_git_branch: "main"
# ocp4_workload_rag_escalation_workbench_storage_size: 20Gi
# ocp4_workload_rag_escalation_workbench_cpu_limit: "8"
# ocp4_workload_rag_escalation_workbench_memory_limit: 24Gi
# ocp4_workload_rag_escalation_workbench_gpu_count: "1"
```
