# Uninstall ACM Multicluster Observability

`uninstall_acm_observability` uninstalls the multicluster observability feature from Red Hat Advanced Cluster Management (ACM) by:

1. Optionally removing the ArgoCD Application for MCO (so GitOps does not recreate it)
2. Deleting the `MultiClusterObservability` custom resource
3. Waiting for the CR to be fully removed (operator tears down Thanos, Grafana, Alertmanager, etc.)
4. Optionally deleting the `open-cluster-management-observability` namespace

Removing the `MultiClusterObservability` CR disables and uninstalls the observability service; the operator removes components from the hub and managed clusters.

## Requirements

- ACM (Advanced Cluster Management) must be installed.
- `kubernetes.core` Ansible collection.
- Cluster access via `KUBECONFIG` or in-cluster auth.

## Usage

### Basic usage

```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: uninstall_acm_observability
```

### With custom configuration

```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: uninstall_acm_observability
      vars:
        mco_cr_name: "observability"
        mco_cr_namespace: "open-cluster-management"
        remove_argocd_application: true
        delete_observability_namespace: false
```

### Remove observability and delete the namespace

```yaml
- hosts: localhost
  roles:
    - role: uninstall_acm_observability
      vars:
        delete_observability_namespace: true
```

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `mco_cr_name` | `observability` | Name of the MultiClusterObservability CR to delete |
| `mco_cr_namespace` | `open-cluster-management` | Namespace where the MultiClusterObservability CR is installed |
| `wait_for_mco_deletion` | `true` | Wait for the CR to be fully deleted |
| `mco_deletion_retries` | `60` | Maximum retries when waiting for deletion |
| `mco_deletion_delay` | `10` | Delay between retries (seconds) |
| `remove_argocd_application` | `true` | Remove the ArgoCD Application for MCO if present |
| `mco_argocd_app_name` | `mco-observability` | ArgoCD Application name for MCO |
| `mco_argocd_namespace` | `openshift-gitops` | ArgoCD Application namespace |
| `delete_observability_namespace` | `false` | Delete the `open-cluster-management-observability` namespace after uninstall |

## Notes

- The role handles missing resources; it is safe to run when observability is not installed.
- If MCO was installed via GitOps, keep `remove_argocd_application: true` so the Application is removed first.
- If your MultiClusterObservability CR is in `open-cluster-management-observability`, set `mco_cr_namespace: "open-cluster-management-observability"`.
- Setting `delete_observability_namespace: true` removes the entire observability namespace and any remaining resources (e.g. PVCs may need manual cleanup).
- Uninstall can take several minutes while the operator removes components on the hub and managed clusters.
