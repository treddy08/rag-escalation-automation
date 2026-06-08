# Quick Start Guide

## 1. Add Collection to Your Catalog Item

In your catalog item's `common.yaml`:

```yaml
requirements_content:
  collections:
  - name: https://github.com/treddy08/rag-escalation-automation.git
    type: git
    version: main
```

## 2. Add to Workload List

```yaml
workloads:
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench

remove_workloads:
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench
```

## 3. Configure (3 Required Variables)

```yaml
ocp4_workload_rag_escalation_workbench_namespace: "{{ your_namespace_var }}"
ocp4_workload_rag_escalation_workbench_git_repo: "https://github.com/FrankLaVigne/EscalationLab"
ocp4_workload_rag_escalation_workbench_tenant_label: "{{ your_tenant_username }}"
```

**Important**: The `tenant_label` is used for the node selector (`tenant: <value>`) to target the tenant's specific GPU node.

## 4. Done!

That's it. The role will:
- ✅ Create PVC (20Gi storage)
- ✅ Clone git repo to `/opt/app-root/src/EscalationLab`
- ✅ Deploy Jupyter workbench with GPU access
- ✅ Enable OAuth authentication
- ✅ Export workbench URL to user data

## Optional: Customize Resources

```yaml
# Only if you need different sizes
ocp4_workload_rag_escalation_workbench_storage_size: 50Gi
ocp4_workload_rag_escalation_workbench_cpu_limit: "16"
ocp4_workload_rag_escalation_workbench_memory_limit: 32Gi
```

## Optional: Use Different Image

```yaml
# For PyTorch workloads
ocp4_workload_rag_escalation_workbench_notebook_image: "quay.io/opendatahub/workbench-images:jupyter-pytorch-ubi9-python-3.11-20250101"

# For TensorFlow workloads
ocp4_workload_rag_escalation_workbench_notebook_image: "quay.io/opendatahub/workbench-images:jupyter-tensorflow-ubi9-python-3.11-20250101"
```

## Example: RAG Escalation Lab Tenant

```yaml
---
# Collection requirement
requirements_content:
  collections:
  - name: https://github.com/treddy08/namespaced_workloads.git
    type: git
    version: main
  - name: https://github.com/treddy08/rag-escalation-automation.git
    type: git
    version: main
  - name: https://github.com/agnosticd/showroom.git
    type: git
    version: v1.6.11

# Workload deployment order
workloads:
- agnosticd.namespaced_workloads.ocp4_workload_tenant_namespace
- agnosticd.namespaced_workloads.ocp4_workload_tenant_keycloak_user
- agnosticd.namespaced_workloads.ocp4_workload_tenant_machineset
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench
- agnosticd.showroom.ocp4_workload_showroom

# Cleanup order (reverse)
remove_workloads:
- agnosticd.showroom.ocp4_workload_showroom
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench
- agnosticd.namespaced_workloads.ocp4_workload_tenant_machineset
- agnosticd.namespaced_workloads.ocp4_workload_tenant_keycloak_user
- agnosticd.namespaced_workloads.ocp4_workload_tenant_namespace

# Workbench configuration
ocp4_workload_rag_escalation_workbench_namespace: "{{ ocp4_workload_tenant_keycloak_username }}-dsproject"
ocp4_workload_rag_escalation_workbench_git_repo: "https://github.com/FrankLaVigne/EscalationLab"
ocp4_workload_rag_escalation_workbench_tenant_label: "{{ ocp4_workload_tenant_keycloak_username }}"
```

## Accessing User Data

The role exports these variables to `agnosticd_user_data`:

```yaml
rag_escalation_workbench_url: "https://rag-escalation-workbench-<namespace>.<cluster-domain>"
rag_escalation_workbench_namespace: "<namespace>"
rag_escalation_git_repo: "https://github.com/FrankLaVigne/EscalationLab"
```

Access in info message template:

```asciidoc
== Jupyter Workbench

Your workbench is ready:

* URL: {{ lookup('agnosticd_user_data', 'rag_escalation_workbench_url') }}
* Repository: /opt/app-root/src/EscalationLab
* GPU: 1x NVIDIA GPU
```

## Troubleshooting

### Notebook not starting

```bash
oc get notebook -n <namespace>
oc describe notebook rag-escalation-workbench -n <namespace>
oc get events -n <namespace> --sort-by='.lastTimestamp'
```

### Git clone failed

```bash
oc logs -n <namespace> <notebook-pod> -c git-clone-repo
```

### No GPU available

```bash
oc get nodes -l nvidia.com/gpu.present=true
oc describe node <gpu-node-name> | grep -A 5 "Allocated resources"
```

## Learn More

- **VARIABLES.md** - All configurable options
- **INTEGRATION_GUIDE.md** - Detailed integration steps
- **SIMPLIFIED.md** - Design decisions
