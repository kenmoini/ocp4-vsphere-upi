# OpenShift VMWare UPI Deployment

This set of resources will deploy an OpenShift cluster to vSphere via User Provisioned Infrastructure (UPI).

It is assumed that you already have the networking and DNS in place to deploy - the load balancer can be externally set or deployed as a RHCOS host onto the Helper node.

## Prerequisites

### 1. Install Ansible

```bash
python3 -m pip install --upgrade pip

python3 -m pip install ansible
```

### 2. Install the needed Python module

```bash
python3 -m pip install -r requirements.txt
```

### 3. Install the needed Ansible Collections

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

---

## Deployment

### 1. Create a cluster-config.yaml file

Copy the `example_vars/cluster_config.yaml` file to the local directory and edit it to your needs.

```bash
cp example_vars/cluster_config.yaml CLUSTER_NAME.cluster_config.yaml
```

### 2. Execute the Bootstrap playbook

```bash
ansible-playbook -e "@CLUSTER_NAME.cluster_config.yaml" bootstrap.yaml
```

---

## Post Deployment

You can also just run the `post-configuration.yaml` Playbook and achieve the following tasks.

```bash
ansible-playbook -e "@CLUSTER_NAME.cluster_config.yaml" post-configuration.yaml
```

### 1. Remove the CSR Auto Approval workload from the cluster

```bash
oc delete project csr-auto-approver
```

### 2. Shutdown & Delete the Bootstrap Node

### 3. Remove the Bootstrap Node from the Load Balancer Pools

### 4. Add the vCenter CPI and CSI

If all the intended VMs in the cluster will be running on VMWare you can add the Cloud Platform Integrations and the Cluster Storage Interfaces - if using a mixed environment then you will need to use an external storage provider, or something like OpenShift Data Foundations.

---

## Helpful Bits

### Auto Approve All Pending CSRs

```bash
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```