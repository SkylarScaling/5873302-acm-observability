# acm_gitops_clustersets

Creates hub **Open Cluster Management** resources so **Placement** / **PlacementDecision** drive Argo CD **ApplicationSet** scheduling (via **clusterDecisionResource**):

1. **`ManagedClusterSet`** (cluster-scoped)
2. **`ManagedClusterSetBinding`** in **`acm_gitops_namespace`** (default **`openshift-gitops`**) — exposes the set to that namespace
3. **`Placement`** in the same namespace — **`spec.clusterSets`** references the set; the placement controller creates **`PlacementDecision`** objects
4. **`ConfigMap`** — duck-typing metadata so the ApplicationSet controller can read **`PlacementDecision`** (`apiVersion` / `kind` / `statusListKey` / `matchKey`)

Requires **Red Hat ACM** (or OCM) APIs on the hub: `managedclustersets.cluster.open-cluster-management.io`, `placementdecisions.cluster.open-cluster-management.io`, etc.

## Cluster membership

With default **`ExclusiveClusterSetLabel`**, add this label to each **`ManagedCluster`** you want in the set (value must match **`acm_managedclusterset_name`**):

```yaml
metadata:
  labels:
    cluster.open-cluster-management.io/clusterset: argo-gitops-clusters
```

A cluster may belong to **only one** `ManagedClusterSet`.

To use a **`LabelSelector`** instead, override **`acm_managedclusterset_spec`** (see OCM / RHACM docs).

## Argo CD RBAC

The **ApplicationSet controller** service account must be able to **get/list** `placementdecisions` in **`acm_gitops_namespace`**. If sync does not generate Applications, add a **ClusterRole** / **Role** + binding per your OpenShift GitOps version (see [Argo CD cluster decision resource generator](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Cluster-Decision-Resource/) and [OCM + Argo CD](https://open-cluster-management.io/docs/scenarios/integration-with-argocd/)).

**Registered clusters:** `status.decisions[].clusterName` must match **Argo CD cluster secret names** (clusters added to Argo for each spoke).

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `acm_gitops_clustersets_enabled` | `true` | Set **`false`** to skip all resources in this role |
| `acm_gitops_namespace` | `openshift-gitops` | Namespace for binding, **Placement**, **ConfigMap** (align with **`applicationset_namespace`**) |
| `acm_managedclusterset_name` | `argo-gitops-clusters` | **ManagedClusterSet** name |
| `acm_managedclusterset_spec` | see `defaults/main.yaml` | **ManagedClusterSet** `spec` |
| `acm_managedclustersetbinding_name` | same as clusterset | **ManagedClusterSetBinding** metadata name |
| `acm_placement_decision_configmap_name` | `argocd-placementdecision-config` | **ConfigMap** name; **`applicationsets_git`** must use the same **`applicationset_placement_decision_configmap`** |
| `acm_placement_name` | `argo-gitops-placement` | **Placement** name; **PlacementDecision** label **`cluster.open-cluster-management.io/placement`** uses this value |
| `acm_placement_spec` | `{}` | Extra **Placement** `spec` fields (e.g. **`predicates`**) after **`clusterSets`** |

## Playbook

Included from [`applicationsets.yaml`](../applicationsets.yaml) **after** **`configure_argo`** and **before** **`applicationsets_git`**.
