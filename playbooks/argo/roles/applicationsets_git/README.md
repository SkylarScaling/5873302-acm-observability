# applicationsets_git

Creates a single Argo CD **`ApplicationSet`** using a **Git directory generator**, optionally combined with a **`clusters` generator** via a **`matrix`** so you get one **`Application`** per **(matching Git path × matching Argo cluster)**.

## Prerequisites

1. **OpenShift GitOps** installed with the ApplicationSet controller (CRD `applicationsets.argoproj.io`).
2. Run after **[`configure_argo`](../configure_argo/)** (or equivalent): the **`AppProject`** must allow **`gitops_repo_url`** in **`spec.sourceRepos`**, and a **repository credential** secret must exist if the repo is private.
3. **Cluster generator:** target clusters must be **registered in Argo CD** (e.g. `argocd cluster add` / cluster secrets). The default **`in-cluster`** entry counts as one cluster.
4. **`spec.goTemplate: true`**. See [Git generator](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Git/), [Cluster generator](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Cluster/), [Matrix generator](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Matrix/), and [GoTemplate](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/GoTemplate/).

## Cluster targeting

| Mode | Behavior |
|------|----------|
| **`applicationset_cluster_generator_enabled: true`** (default) | **`matrix`** of **Git directories × clusters**. **`spec.destination.server`** is **`{{ .server }}`** from each cluster. Application **`metadata.name`** is **`{{ .name }}-{{ .path.basenameNormalized }}`** so names stay unique in the GitOps namespace. Restrict clusters with **`applicationset_clusters_selector`** (optional); empty **`{}`** means **all** clusters Argo knows. |
| **`applicationset_cluster_generator_enabled: false`** | **Git only** (hub-style). Every Application uses **`applicationset_destination_server`** (default in-cluster URL). |

**AppProject `destinations`:** [`configure_argo`](../configure_argo/)’s default project only allows **`https://kubernetes.default.svc`**. For spoke clusters, extend **`spec.destinations`** (e.g. add each API server URL, or a pattern your policy allows). Otherwise sync will be denied by the project.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `applicationset_name` | `gitops-generated-apps` | `ApplicationSet` metadata name |
| `applicationset_namespace` | `openshift-gitops` | Namespace for the ApplicationSet and generated Applications |
| `applicationset_project` | `acm` | Argo CD `AppProject` for generated Applications |
| `applicationset_repo_revision` | `main` | Git ref for generator and `spec.source.targetRevision` |
| `applicationset_git_directories` | see `defaults/main.yaml` | Git directory rules (see below) |
| `applicationset_cluster_generator_enabled` | `true` | Enable **matrix(git × clusters)** |
| `applicationset_clusters_selector` | `{}` | Optional **`clusters.selector`** (e.g. `matchLabels`). Empty = all registered clusters |
| `applicationset_destination_server` | `https://kubernetes.default.svc` | Used only when **`cluster_generator_enabled`** is **`false`** |
| `applicationset_destination_namespace_mode` | `basename` | `basename` or `fixed` (see below) |
| `applicationset_destination_namespace` | `""` | Required when **`namespace_mode`** is **`fixed`** |
| `applicationset_directory_recurse` | `true` | `spec.source.directory.recurse` |
| `applicationset_sync_policy` | `Automatic` | `Automatic` or `Manual` |
| `applicationset_sync_options` | see defaults | `syncPolicy.syncOptions` |
| `applicationset_sync_retry_*` | see defaults | `syncPolicy.retry` backoff |

`gitops_repo_url` is provided by the **`arg_gitops_repo`** role dependency.

### `applicationset_git_directories`

Each item must include **`path`**. Optional **`exclude`**: `true` to exclude matching paths.

### `applicationset_clusters_selector`

Non-empty dict becomes **`spec.generators[0].matrix.generators[1].clusters.selector`** in the rendered manifest.

Example (only clusters labeled `usage: workload`):

```yaml
applicationset_clusters_selector:
  matchLabels:
    usage: workload
```

Leave **`{}`** to include every cluster secret Argo CD manages (including **`in-cluster`**).

### Namespace modes

- **`basename`**: destination namespace = **`path.basenameNormalized`** (per cluster, namespaces are isolated).
- **`fixed`**: set **`applicationset_destination_namespace`** for every Application.

## Playbook

[`applicationsets.yaml`](../../applicationsets.yaml) runs **`configure_argo`** then **`applicationsets_git`**.

```bash
ansible-playbook playbooks/argo/applicationsets.yaml \
  -e '{"applicationset_clusters_selector":{"matchLabels":{"region":"us-east"}}}'
```

Hub-only (single static server, no cluster generator):

```bash
ansible-playbook playbooks/argo/applicationsets.yaml \
  -e '{"applicationset_cluster_generator_enabled":false}'
```
