# Install Prerequisites
```
ansible-playbook install-acm-prerequisites.yaml --ask-become-pass
```

# Create ACM Service Account
`admin_create_service_account` will create a ServiceAccount in OCP and capture and encrypt the api token for use with further automation.

Example:
```
oc login <cluster admin login>
```
```
ansible-playbook create-acm-service-account.yaml -e cluster_address=<cluster address>
```

# Install ACS via ACM Subscription
`install_acs_from_subscription` installs ACS to the hub and managed clusters using ACM policies, created using a Subscription.

Example:
```
ansible-playbook install-acs-via-acm.yaml --vault-password-file=~/.passfile -e cluster_address=<cluster address>
```

# Install OpenShift Pipelines via ACM Subscription
`install_ocp_pipelines_from_subscription` installs OpenShift Pipelines to all managed clusters using ACM policies, created using a Subscription.

Example:
```
ansible-playbook install-pipelines-via-acm.yaml --vault-password-file=~/.passfile -e cluster_address=<cluster address>
```