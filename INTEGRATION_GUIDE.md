# Integration Guide: RAG Escalation Lab Tenant CI

## Overview

This guide shows how to integrate the `ocp4_workload_rag_escalation_workbench` role into the RAG Escalation Lab tenant catalog item.

## Step 1: Update Collection Requirements

In `agd_v2/rag-escalation-lab-tenant/common.yaml`, update the `requirements_content` section:

```yaml
requirements_content:
  collections:
  # Tenant roles (keycloak_user, namespace, gpu_machineset)
  - name: https://github.com/treddy08/namespaced_workloads.git
    type: git
    version: "{{ tag }}"

  # RAG Escalation workbench (NEW)
  - name: https://github.com/treddy08/rag-escalation-automation.git
    type: git
    version: "{{ tag }}"

  # Showroom
  - name: https://github.com/agnosticd/showroom.git
    type: git
    version: v1.6.11
```

## Step 2: Update Workload Order

Add the workbench role to the deployment sequence:

```yaml
workloads:
- agnosticd.namespaced_workloads.ocp4_workload_tenant_namespace
- agnosticd.namespaced_workloads.ocp4_workload_tenant_keycloak_user
- agnosticd.namespaced_workloads.ocp4_workload_tenant_machineset
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench  # NEW
- agnosticd.showroom.ocp4_workload_showroom

remove_workloads:
- agnosticd.showroom.ocp4_workload_showroom
- agnosticd.rag_escalation_automation.ocp4_workload_rag_escalation_workbench  # NEW
- agnosticd.namespaced_workloads.ocp4_workload_tenant_machineset
- agnosticd.namespaced_workloads.ocp4_workload_tenant_keycloak_user
- agnosticd.namespaced_workloads.ocp4_workload_tenant_namespace
```

## Step 3: Configure Workbench Variables

Add these configuration variables (only 2 required!):

```yaml
# -------------------------------------------------------------------
# Workload: ocp4_workload_rag_escalation_workbench
# Deploys JupyterHub workbench with EscalationLab repository pre-loaded
# -------------------------------------------------------------------
ocp4_workload_rag_escalation_workbench_namespace: "{{ ocp4_workload_tenant_keycloak_username }}-dsproject"
ocp4_workload_rag_escalation_workbench_git_repo: "https://github.com/FrankLaVigne/EscalationLab"

# Optional: override defaults if needed
# ocp4_workload_rag_escalation_workbench_git_branch: "main"
# ocp4_workload_rag_escalation_workbench_storage_size: 20Gi
# ocp4_workload_rag_escalation_workbench_cpu_limit: "8"
# ocp4_workload_rag_escalation_workbench_memory_limit: 24Gi
```

## Step 4: Update Info Message Template (Optional)

In `agd_v2/rag-escalation-lab-tenant/info-message-template.adoc`, add workbench access info:

```asciidoc
== Jupyter Workbench

Your personal Jupyter workbench is ready with the EscalationLab repository pre-loaded.

* Workbench URL: {{ lookup('agnosticd_user_data', 'rag_escalation_workbench_url') }}
* Repository location: `/opt/app-root/src/EscalationLab`
* GPU access: 1x NVIDIA GPU on dedicated node

Access via OpenShift AI dashboard or the direct URL above.
```

## What Gets Deployed

When a user orders the tenant CI, they receive:

1. **Namespace**: `user-<guid>-dsproject` (Data Science Project)
2. **Keycloak User**: `user-<guid>` with password
3. **GPU Node**: Dedicated worker node (provisioned by machineset)
4. **Jupyter Workbench**: 
   - Name: `rag-escalation-workbench`
   - Git repo: `https://github.com/FrankLaVigne/EscalationLab` (pre-cloned)
   - Working directory: `/opt/app-root/src/EscalationLab`
   - Resources: 8 CPU, 24Gi RAM, 1 GPU, 20Gi storage
   - OAuth authentication enabled
   - Scheduled on: any available GPU node (via GPU resource request)
5. **Showroom**: Lab guide UI

## Deployment Flow

```
Order Tenant CI
    ↓
Create namespace (user-<guid>-dsproject)
    ↓
Create Keycloak user (user-<guid>)
    ↓
Provision GPU machineset (1 node with tenant label)
    ↓
Deploy workbench (with git clone init container)
    ↓
Deploy Showroom lab guide
    ↓
Export URLs to user
```

## Verification

After deployment, verify:

```bash
# Check namespace
oc get namespace user-<guid>-dsproject

# Check notebook
oc get notebook -n user-<guid>-dsproject

# Check PVC
oc get pvc -n user-<guid>-dsproject

# Check GPU node assignment
oc get pod -n user-<guid>-dsproject -o wide

# Verify git repo is cloned
oc exec -n user-<guid>-dsproject <notebook-pod> -- ls -la /opt/app-root/src/EscalationLab
```

## Cleanup

When the tenant CI is destroyed, the removal order ensures clean teardown:

1. Showroom removed
2. Workbench deleted (Notebook CR)
3. PVC deleted
4. GPU machineset deleted (node deprovisioned)
5. Keycloak user removed
6. Namespace deleted

## Troubleshooting

### Notebook not starting

Check notebook status:
```bash
oc get notebook -n user-<guid>-dsproject -o yaml
```

Look for events:
```bash
oc get events -n user-<guid>-dsproject --sort-by='.lastTimestamp'
```

### Git clone failed

Check init container logs:
```bash
oc logs -n user-<guid>-dsproject <notebook-pod> -c git-clone-repo
```

### GPU not available

Verify GPU node exists:
```bash
oc get machinesets -n openshift-machine-api | grep gpu-<guid>
oc get nodes -l tenant=user-<guid>
```

Check GPU allocation:
```bash
oc describe node <gpu-node-name> | grep -A 5 "Allocated resources"
```
