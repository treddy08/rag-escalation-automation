# Simplified Role Configuration

## What Changed

The role has been dramatically simplified based on the principle: **hard-code what never changes, expose only what varies**.

### File Structure Simplification

**Before**: 5 task files (main, pre_workload, workload, post_workload, remove_workload)  
**After**: 3 task files (main, workload, remove_workload)

- ❌ Removed `pre_workload.yml` - validation moved to workload.yml
- ❌ Removed `post_workload.yml` - user data export moved to workload.yml
- ✅ Single linear flow in workload.yml: validate → create → wait → export

### Hard-Coded Values

These are now fixed in the templates and cannot be overridden:

| What | Value | Why |
|------|-------|-----|
| **Workbench name** | `rag-escalation-workbench` | Fixed name, no need to vary |
| **PVC name** | `rag-escalation-workbench-pvc` | Matches workbench name |
| **Git target directory** | `EscalationLab` | Fixed directory name |
| **Working directory** | `/opt/app-root/src/EscalationLab` | Always the same |
| **OAuth enabled** | `true` | Always use OAuth |
| **OAuth logout URL** | `https://console-openshift-console.<cluster>/oauth/sign_out` | Standard OpenShift console URL |

### GPU Scheduling (Always On)

The workbench **always** deploys on the tenant's GPU node:
- **GPU resource**: `nvidia.com/gpu: 1` (requests 1 GPU)
- **GPU toleration**: `nvidia.com/gpu=true:NoSchedule` (allows scheduling on GPU-tainted nodes)
- **Node selector**: `tenant: <tenant_label>` (pins to tenant's specific GPU node)

This ensures multi-tenant isolation - each workbench lands on its tenant's dedicated GPU node.

### Required Variables (Only 2!)

```yaml
ocp4_workload_rag_escalation_workbench_namespace: "user-abc123-dsproject"
ocp4_workload_rag_escalation_workbench_git_repo: "https://github.com/FrankLaVigne/EscalationLab"
```

### Optional Variables (Rarely Need to Change)

```yaml
# Git
ocp4_workload_rag_escalation_workbench_git_branch: "main"

# Image (only change if you need PyTorch/TensorFlow)
ocp4_workload_rag_escalation_workbench_notebook_image: "quay.io/opendatahub/workbench-images:jupyter-datascience-ubi9-python-3.11-20250101"

# Resources (only change if you need more/less)
ocp4_workload_rag_escalation_workbench_storage_size: 20Gi
ocp4_workload_rag_escalation_workbench_cpu_limit: "8"
ocp4_workload_rag_escalation_workbench_memory_limit: 24Gi
ocp4_workload_rag_escalation_workbench_gpu_count: "1"

# Advanced
ocp4_workload_rag_escalation_workbench_pip_packages: []  # Additional packages
```

## GPU Scheduling

**Important**: The workbench does NOT need a node selector to target a specific GPU node.

Here's why:
1. Pod requests `nvidia.com/gpu: "1"` in resources
2. Pod has toleration for `nvidia.com/gpu=true:NoSchedule`
3. Kubernetes scheduler automatically places it on any GPU node with available GPUs
4. GPU is only used when user executes code in notebooks, not by JupyterLab itself

The tenant's GPU machineset ensures there's a GPU node available. The scheduler handles the rest.

## Tenant CI Integration

Minimal configuration needed:

```yaml
# In rag-escalation-lab-tenant/common.yaml

# Collection
requirements_content:
  collections:
  - name: https://github.com/treddy08/rag-escalation-automation.git
    type: git
    version: main

# Workload
workloads:
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench

# Configuration (just 2 required vars!)
ocp4_workload_rag_escalation_workbench_namespace: "{{ ocp4_workload_tenant_keycloak_username }}-dsproject"
ocp4_workload_rag_escalation_workbench_git_repo: "https://github.com/FrankLaVigne/EscalationLab"
```

That's it! Everything else uses sensible defaults.

## What Gets Created

Always the same:
- **Notebook**: `rag-escalation-workbench`
- **PVC**: `rag-escalation-workbench-pvc`
- **Git repo cloned to**: `/opt/app-root/src/EscalationLab`
- **Working directory**: `/opt/app-root/src/EscalationLab`
- **GPU access**: 1x NVIDIA GPU
- **Resources**: 8 CPU, 24Gi RAM, 20Gi storage
- **OAuth**: Enabled via OpenShift

## Variables Removed

These were removed because they never needed to vary:
- ❌ `ocp4_workload_rag_escalation_workbench_name`
- ❌ `ocp4_workload_rag_escalation_workbench_pvc_name`
- ❌ `ocp4_workload_rag_escalation_workbench_git_target_dir`
- ❌ `ocp4_workload_rag_escalation_workbench_git_clone_depth`
- ❌ `ocp4_workload_rag_escalation_workbench_storage_class`
- ❌ `ocp4_workload_rag_escalation_workbench_tolerations`
- ❌ `ocp4_workload_rag_escalation_workbench_node_selector`
- ❌ `ocp4_workload_rag_escalation_workbench_tenant_label`
- ❌ `ocp4_workload_rag_escalation_workbench_env_vars`
- ❌ `ocp4_workload_rag_escalation_workbench_oauth_enabled`
- ❌ `ocp4_workload_rag_escalation_workbench_oauth_logout_url`
- ❌ `ocp4_workload_rag_escalation_workbench_skip_workload`

## Result

**Before**: 25+ variables, many complex nested structures  
**After**: 2 required + 7 optional simple variables

The role does one thing well: deploy a Jupyter workbench with a git repo on a GPU-enabled cluster.
