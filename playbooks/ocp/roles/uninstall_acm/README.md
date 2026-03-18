# Uninstall ACM

`uninstall_acm` uninstalls Advanced Cluster Management (ACM) from an OpenShift cluster by:
1. Deleting MultiClusterObservability (if present), then MultiClusterHub
2. Deleting the ACM operator Subscription and CSV
3. Deleting the **Multicluster Engine** operator Subscription and CSV (default namespace `multicluster-engine`)
4. Removing **ACM/MCE console plugins** (`ConsolePlugin` `acm` / `mce` and their entries under `consoles.operator.openshift.io` `cluster` `spec.plugins`) so the web console does not keep calling `/api/plugins/acm/` after uninstall
5. Optionally cleaning up ManagedCluster and KlusterletAddonConfig resources
6. Optionally deleting the operator namespace

## Usage

### Basic Usage

```yaml
- hosts: localhost
  roles:
    - role: uninstall_acm
```

### With Custom Configuration

```yaml
- hosts: localhost
  roles:
    - role: uninstall_acm
      vars:
        acm_multiclusterhub_name: "my-multiclusterhub"
        acm_multiclusterhub_namespace: "open-cluster-management"
        acm_operator_subscription_name: "advanced-cluster-management"
        acm_operator_subscription_namespace: "open-cluster-management"
        cleanup_managed_clusters: true
        delete_operator_namespace: false
```

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `acm_multiclusterhub_name` | `multiclusterhub` | Name of the MultiClusterHub CR to delete |
| `acm_multiclusterhub_namespace` | `open-cluster-management` | Namespace where MultiClusterHub is installed |
| `acm_operator_subscription_name` | `advanced-cluster-management` | Name of the ACM operator Subscription |
| `acm_operator_subscription_namespace` | `open-cluster-management` | Namespace where the operator Subscription is installed |
| `wait_for_multiclusterhub_deletion` | `true` | Wait for MultiClusterHub to be fully deleted |
| `multiclusterhub_deletion_retries` | `60` | Maximum retries when waiting for deletion |
| `multiclusterhub_deletion_delay` | `10` | Delay between retries (seconds) |
| `delete_operator_csv` | `true` | Delete the ACM operator CSV after uninstalling |
| `mce_operator_subscription_name` | `multicluster-engine` | MCE Subscription name to remove |
| `mce_operator_subscription_namespace` | `multicluster-engine` | Namespace where MCE Subscription is installed |
| `delete_mce_operator` | `true` | Remove MCE Subscription and CSV (set `false` if MCE not installed) |
| `delete_mce_operator_csv` | `true` | Delete the MCE operator CSV after removing its Subscription |
| `delete_acm_console_plugins` | `true` | Delete `ConsolePlugin` acm/mce and drop them from Console `spec.plugins` |
| `acm_console_plugin_names` | `["acm","mce"]` | Plugin names to remove from cluster console |
| `delete_operator_namespace` | `false` | Delete the operator namespace after uninstalling |
| `preclean_ocm_namespace_before_delete` | `true` | Before namespace delete, run `oc delete <each namespaced kind> --all` in passes (needs `oc`) |
| `ocm_namespace_preclean_passes` | `3` | How many full passes over API resources to delete (handles ordering) |
| `cleanup_managed_clusters` | `false` | Delete ManagedCluster and KlusterletAddonConfig resources |

## Notes

- The role will gracefully handle cases where resources don't exist
- If `cleanup_managed_clusters` is set to `true`, all ManagedCluster and KlusterletAddonConfig resources will be deleted
- Setting `delete_operator_namespace` to `true` removes the namespace but the role **does not wait** for it to finish (namespaces often stay `Terminating`); clear finalizers manually if needed
- The uninstall process may take several minutes depending on cluster resources
