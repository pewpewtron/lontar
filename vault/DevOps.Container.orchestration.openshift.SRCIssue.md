---
id: nmx2b1qf4bgy74er0tg8lxh
title: CSRIssue
desc: ''
updated: 1662386487526
created: 1651151442682
---

During cluster deployment, src isssue can occurs when cluster is shutting down in less than 24h after cluster is deployed. the solution is to manually aprove the src following step is to fix src issue

### 1. Start and login to cluster via terminal

```bash
oc login -u system:admin -p your-secure-password
oc whoami
```

> Note: make sure the user have cluster administrator role

### 2. Check the cluster status

```bash
oc get nodes
oc get co
oc get csr
```

### 3. Check CSR

when running `oc get csr` command, you will if theres a CSR pending, check the detail by using `oc describe` subcommand

```bash
oc describe csr my-csr
Name:               my-csr
Labels:             <none>
Annotations:        <none>
CreationTimestamp:  Mon, 18 Apr 2022 06:35:11 +0000
Requesting User:    system:serviceaccount:openshift-machine-config-operator:node-bootstrapper
Signer:             kubernetes.io/kube-apiserver-client-kubelet
Status:             Approved,Issued
Subject:
         Common Name:    system:node:i3okd-training-d8n8j-worker-a-m7zm8
         Serial Number:
         Organization:   system:nodes
Events:  <none>
```

> Note: make sure csr description is not containning localhost for its domain.

### 4. Approve CSR

User can aprove csr one by one by using `oc adm certificate approve` command or when theres more than 3 csr pending, user can use script shown below to aprove all pending csr

```bash
oc adm certificate approve my-csr
# or using this command to aprove all pending CSR
oc get csr --no-headers | awk '{print $1}' | xargs oc adm certificate approve
```

> Note: make sure all csr is approved usually new csr is appear after aproving some csr.

### 5. Check the cluster status again

After all csr aprove now check the cluster status again, starting with csr then cluster operators then the nodes

```bash
oc get csr
oc get co
oc get node
```