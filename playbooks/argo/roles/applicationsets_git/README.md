# applicationsets_git

Creates a single Argo CD **`ApplicationSet`** using a **Git directory generator** in a **`matrix`** with either:

- **`clusterDecisionResource`** (default) — follows OCM **`PlacementDecision`** resources produced by **[`acm_gitops_clustersets`](../acm_gitops_clustersets/)** **Placement**, or  
- **`clusters`** — Argo CD cluster secrets only (`applicationset_cluster_selection: argocd`).

## Prerequisites

1. **OpenShift GitOps** with the ApplicationSet controller (CRD `applicationsets.argoproj.io`).
2. **[`acm_gitops_clustersets`](../acm_gitops_clustersets/)** when using **`placement`**: **ManagedClusterSet**, **Binding**, **Placement**, and the **PlacementDecision ConfigMap**. Set **`acm_gitops_clustersets_enabled: false`** only if you do not use ACM / placement mode.
3. **ApplicationSet controller RBAC** to **get/list** `placementdecisions` in **`applicationset_namespace`** when using **`placement`** (see **`acm_gitops_clustersets`** README).
4. **[`configure_argo`](../configure_argo/)**: **AppProject** allows **`gitops_repo_url`** in **`sourceRepos`**; repo credentials if private.
5. **Clusters in `PlacementDecision.status.decisions`** must exist as **Argo CD cluster** registrations; **`clusterName`** matches the Argo cluster name (often the **ManagedCluster** name).
6. **`spec.goTemplate: true`**. See [Git](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Git/), [cluster decision resource](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Cluster-Decision-Resource/), [Matrix](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Matrix/), [GoTemplate](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/GoTemplate/).

## Cluster targeting

| Setting | Behavior |
|---------|----------|
| **`applicationset_cluster_generator_enabled: true`**, **`applicationset_cluster_selection: placement`** (default) | **`matrix(git × clusterDecisionResource)`**. Selects every **PlacementDecision** labeled **`cluster.open-cluster-management.io/placement=<applicationset_placement_name>`**. Template uses **`{{ .clusterName }}`**, **`{{ .path.basenameNormalized }}`**, **`{{ .server }}`** (Argo resolves API URL from the registered cluster). |
| **`applicationset_cluster_generator_enabled: true`**, **`applicationset_cluster_selection: argocd`** | **`matrix(git × clusters)`** — same as before: **`applicationset_clusters_selector`**, **`{{ .name }}`** + **`{{ .path.basenameNormalized }}`**, **`{{ .server }}`**. |
| **`applicationset_cluster_generator_enabled: false`** | **Git only**; **`applicationset_destination_server`** for every Application. |

**AppProject `destinations`:** extend allowed **`server`** values for spokes (or `*`) when deploying off the hub. See [`configure_argo`](../configure_argo/).

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `applicationset_name` | `gitops-generated-apps` | `ApplicationSet` metadata name |
| `applicationset_namespace` | `openshift-gitops` | Namespace for **`ApplicationSet`** / **`Application`** metadata |
| `applicationset_destination_namespace` | same as **`applicationset_namespace`** | **`spec.destination.namespace`** (must equal **`applicationset_namespace`**; role asserts) |
| `applicationset_project` | `acm` | Argo **`AppProject`** |
| `applicationset_repo_revision` | `main` | Git ref |
| `applicationset_git_directories` | see defaults | Git directory rules |
| `applicationset_cluster_generator_enabled` | `true` | Enable **matrix** with a cluster source |
| `applicationset_cluster_selection` | `placement` | **`placement`** or **`argocd`** |
| `applicationset_placement_decision_configmap` | `argocd-placementdecision-config` | **clusterDecisionResource.configMapRef** (must match **`acm_placement_decision_configmap_name`**) |
| `applicationset_placement_name` | `argo-gitops-placement` | **PlacementDecision** label value (match **`acm_placement_name`**) |
| `applicationset_placement_requeue_after_seconds` | `60` | **clusterDecisionResource.requeueAfterSeconds** |
| `applicationset_clusters_selector` | `{}` | Used only when **`cluster_selection: argocd`** |
| `applicationset_destination_server` | `https://kubernetes.default.svc` | Used when **`cluster_generator_enabled: false`** |
| `applicationset_directory_recurse` | `true` | `spec.source.directory.recurse` |
| `applicationset_sync_policy` | `Automatic` | `Automatic` or `Manual` |
| `applicationset_sync_options` / `applicationset_sync_retry_*` | see defaults | Sync policy |

`gitops_repo_url` comes from **`arg_gitops_repo`**.

### `applicationset_git_directories`

Each item: **`path`**; optional **`exclude: true`**.

### `applicationset_clusters_selector` (**`argocd`** mode only)

Non-empty dict becomes **`clusters.selector`** on the second matrix generator.

**Sync destination namespace:** **`spec.destination.namespace`** is **`applicationset_destination_namespace`** (Argo CD namespace).

## Playbook

[`applicationsets.yaml`](../../applicationsets.yaml) sets **`applicationset_placement_name`** and **`applicationset_placement_decision_configmap`** from **`acm_placement_name`** / **`acm_placement_decision_configmap_name`** when you define them on the play.

```bash
ansible-playbook playbooks/argo/applicationsets.yaml
```

**Argo-only clusters** (no PlacementDecision):

```bash
ansible-playbook playbooks/argo/applicationsets.yaml \
  -e '{"applicationset_cluster_selection":"argocd"}'
```

**Hub-only** (no matrix):

```bash
ansible-playbook playbooks/argo/applicationsets.yaml \
  -e '{"applicationset_cluster_generator_enabled":false}'
```
