Example inventory:

```yaml
all:
  children:
    local:
      hosts:
        localhost:
          ansible_connection: local
    bastion:
      hosts:
        bastion-xxxxx.xxxxx.dynamic.redhatworkshops.io:
          ansible_user: lab-user
          ansible_password: <ssh_password>
          ansible_become_password: <ssh_password>
          ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
  vars:
    home_dir: /home/sscaling
    tmp_dir: "{{ home_dir }}/tmp"
    openshift_version: "4.20"
    ocp_patch_version: "13"
    force_update: true
    ocp_cluster:
      name: highmark-cluster
      base_domain: xxxxx.dynamic.redhatworkshops.io

    vcenter:
      hostname: vcsnsx-vc.infra.demo.redhat.com
      username: sandbox-xxxxx@demo
      password: "<vmware_password>"
      datacenter: SDDC-Datacenter
      cluster: Cluster-1
      datastore: workload_share_xxxxx #Find this in vSphere
      network: segment-sandbox-xxxxx
      folder: "/{{ vcenter.datacenter }}/vm/Workloads/sandbox-xxxxx"
      rhcos_template: "rhcos-4.20-template"

    ocp_nodes:
      bootstrap:
        name: "corp-p-ocp-boot-01"
      masters:
        - name: "corp-p-ocp-cp-01"
        - name: "corp-p-ocp-cp-02"
        - name: "corp-p-ocp-cp-03"
      workers:
        - name: "corp-p-ocp-wk-01"
        - name: "corp-p-ocp-wk-02"
        - name: "corp-p-ocp-wk-03"

    aws:
      account_id: ""
      aws_access_key_id: ""
      aws_secret_access_key: ""
      aws_region: us-east-2

    install_config:
      cluster_name: "{{ ocp_cluster.name }}"
      base_domain: "{{ ocp_cluster.base_domain }}"
      control_plane:
        instance_type: "m6a.2xlarge"
      workers:
        instance_type: "m6a.xlarge"
        replicas: "3"
      infra:
        instance_type: "m6a.4xlarge"
        replicas: "3"
      storage:
        instance_type: "m6a.4xlarge"
        replicas: "3"
      ssh_key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"
      pull_secret: "{{ lookup('file', '~/pull-secret.json') | from_json }}"

```

---

### Inventory Variables

Before running the provisioning playbooks, you must populate the `<var_name>` placeholders in your inventory file. Here is a breakdown of what each section controls:

#### 📁 General Configuration

* **`home_dir`**: The absolute path to your local home directory (e.g., `/home/username`). This is used as the base path for downloads and configurations.
* **`openshift_version`** & **`ocp_patch_version`**: Dictates the exact OpenShift release to download and install (e.g., `4.20` and `12` installs `4.20.12`).
* **`force_update`**: Set to `true` if you want the playbook to overwrite your existing `oc` and `openshift-install` binaries with fresh downloads.

#### 🌐 Cluster Identity

* **`ocp_cluster.name`**: The unique name for your OpenShift cluster. This will be used to tag all AWS resources.
* **`ocp_cluster.base_domain`**: The Route53 DNS zone hosting your cluster (e.g., `sandbox.example.com`). Your final console URL will be `console-openshift-console.apps.<name>.<base_domain>`.

#### ☁️ AWS Credentials

* **`aws.account_id`**: Your 12-digit AWS Account ID.
* **`aws.aws_access_key_id`** & **`aws.aws_secret_access_key`**: An IAM user's programmatic access keys. This user needs sufficient privileges to provision VPCs, EC2 instances, Route53 records, and IAM roles.
* **`aws.aws_region`**: The AWS region to deploy into (e.g., `us-east-2`).

#### 🏗️ Node Architecture (`install_config`)

This block defines the specific EC2 instance types and replica counts for your OpenShift node topology:

* **`control_plane`**: The master nodes managing the cluster state (default: `m6a.xlarge`).
* **`workers`**: General-purpose worker nodes for standard application workloads.
* **`infra`**: Dedicated nodes for OpenShift ingress routers, image registry, and monitoring. Moving these off standard workers saves on subscription consumption.
* **`storage`**: Dedicated `m6a.4xlarge` heavy-compute nodes specifically tainted and labeled to house OpenShift Data Foundation (ODF) and your Ceph storage clusters.

#### 🔐 Secrets & Access

* **`ssh_key`**: Points to your local public SSH key (typically `~/.ssh/id_ed25519.pub`). This is injected into the EC2 instances so you can SSH into the nodes for emergency debugging.
* **`pull_secret`**: The path to your Red Hat pull secret (downloaded from `console.redhat.com`). This authenticates your cluster to pull core OpenShift and Operator images.

---

# RHCOS 4.20 on vmWare
The demo.redhat environment for VMware only includes rhcos-4.16 by default. So if you want to install a different version,
you will need to follow the following steps:

- Copy this exact URL to your clipboard:

    `https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.20/latest/rhcos-vmware.x86_64.ova`

- Log into the vCenter UI (https://your-vcenter.infra.demo.redhat.com/ui/).

- Navigate to the VMs and Templates view

- Right-click your assigned folder (sandbox-xxxxx) and select Deploy OVF Template.

- Select an OVF template: 
  - Choose the URL option, paste the URL from step 1, and click Next.

- Select a name and folder: 
  - Name it exactly rhcos-4.20-template (to match your inventory) and ensure your sandbox-xxxxx folder is selected.

- Select a compute resource: Choose your Cluster.

- Select storage: Choose the datastore you picked earlier (e.g., workload_share_xxxxx).

- Select networks: Choose segment-sandbox-xxxxx.

- Click Finish and wait for the task to complete in the "Recent Tasks" pane at the bottom of vCenter.

Once the VM appears in your folder:

- Right-click the new rhcos-4.20-template VM.

- Select Template > Convert to Template.
