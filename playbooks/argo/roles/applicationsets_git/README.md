# applicationsets_git

Creates one Argo CD **`ApplicationSet`** using **`clusterDecisionResource`**, following OCM **`PlacementDecision`** objects from **[`acm_gitops_clustersets`](../acm_gitops_clustersets/)** (**Placement** + ConfigMap).

Cluster targeting is exclusively via **PlacementDecision**; **`spec.destination.server`** uses **`{{ .server }}`** from the generator. **`spec.source.path`** is a single fixed path (**`applicationset_source_path`**) — create a second ApplicationSet invocation for each additional repo path.

## Prerequisites

1. **OpenShift GitOps** with the ApplicationSet controller (CRD `applicationsets.argoproj.io`).
2. **[`acm_gitops_clustersets`](../acm_gitops_clustersets/)**: **ManagedClusterSet**, **Binding**, **Placement**, and the **PlacementDecision ConfigMap**.
3. **ApplicationSet controller RBAC** to **get/list** `placementdecisions` in **`applicationset_namespace`** (see **`acm_gitops_clustersets`** README).
4. **[`configure_argo`](../configure_argo/)**: **AppProject** allows **`gitops_repo_url`** in `sourceRepos`; repo credentials if private.
5. **Clusters in `PlacementDecision.status.decisions`** must exist as Argo CD cluster registrations; `clusterName` matches the Argo cluster name.
6. **`spec.goTemplate: true`**. See [cluster decision resource](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Cluster-Decision-Resource/).

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `applicationset_name` | `gitops-generated-apps` | `ApplicationSet` metadata name |
| `applicationset_namespace` | `openshift-gitops` | Namespace for `ApplicationSet` / `Application` metadata and `spec.destination.namespace` |
| `applicationset_project` | `acm` | Argo **AppProject** |
| `applicationset_repo_revision` | `main` | Git ref |
| `applicationset_source_path` | `mco` | `spec.source.path` under `gitops_repo_url` |
| `applicationset_application_suffix` | `mco` | App name suffix: `<clusterName>-<suffix>` |
| `applicationset_placement_decision_configmap` | `argocd-placementdecision-config` | `clusterDecisionResource.configMapRef` (must match `acm_placement_decision_configmap_name`) |
| `applicationset_placement_name` | `argo-gitops-placement` | `PlacementDecision` label value (must match `acm_placement_name`) |
| `applicationset_placement_requeue_after_seconds` | `60` | `clusterDecisionResource.requeueAfterSeconds` |
| `applicationset_directory_recurse` | `true` | `spec.source.directory.recurse` |
| `applicationset_sync_policy` | `Automatic` | `Automatic` or `Manual` |
| `applicationset_sync_options` / `applicationset_sync_retry_*` | see defaults | Sync policy |

`gitops_repo_url` comes from **`arg_gitops_repo`**.

## Playbook

[`applicationsets.yaml`](../../applicationsets.yaml) maps `applicationset_placement_name` and `applicationset_placement_decision_configmap` from `acm_placement_name` / `acm_placement_decision_configmap_name`.

```bash
ansible-playbook playbooks/argo/applicationsets.yaml
```
