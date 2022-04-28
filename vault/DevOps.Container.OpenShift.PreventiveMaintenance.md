---
id: yk05se0mtsbfd6ajo7yygm1
title: PreventiveMaintenance
desc: ''
updated: 1651151232032
created: 1651144905621
---

## Preventive Maintenance Task list and what to do

TLDR: Check cluster status and health by using oc get node, co, and mcp, then collect logs for each node (cpu and memory, and disk), and check the logs apps (using naamespace) and collect cpu, and memmory logs .

### 1. Login to bastion

`oc login -u system:admin -p your-super-secure-password`

### 2. Check cluster status

Check cluster node, cluster operator and machine config (mcp)

`oc get nodes`

output example:

```bash
NAME                                  STATUS   ROLES    AGE   VERSION
i3okd-training-d8n8j-master-0         Ready    master   14d   v1.22.3+2cb6068
i3okd-training-d8n8j-master-1         Ready    master   14d   v1.22.3+2cb6068
i3okd-training-d8n8j-master-2         Ready    master   14d   v1.22.3+2cb6068
i3okd-training-d8n8j-worker-a-m7zm8   Ready    worker   14d   v1.22.3+2cb6068
i3okd-training-d8n8j-worker-b-svk22   Ready    worker   14d   v1.22.3+2cb6068
i3okd-training-d8n8j-worker-c-7b625   Ready    worker   14d   v1.22.3+2cb6068
```

`oc get clusteroperator` or `oc get co`

Output example:

```bash
NAME                                       VERSION                         AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
authentication                             4.9.0-0.okd-2022-01-29-035536   True        False         False      32m
baremetal                                  4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
cloud-controller-manager                   4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
cloud-credential                           4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
cluster-autoscaler                         4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
config-operator                            4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
console                                    4.9.0-0.okd-2022-01-29-035536   True        False         False      33m
csi-snapshot-controller                    4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
dns                                        4.9.0-0.okd-2022-01-29-035536   True        False         False      3d9h
etcd                                       4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
image-registry                             4.9.0-0.okd-2022-01-29-035536   True        False         False      34m
ingress                                    4.9.0-0.okd-2022-01-29-035536   True        False         False      34m
insights                                   4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
kube-apiserver                             4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
kube-controller-manager                    4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
kube-scheduler                             4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
kube-storage-version-migrator              4.9.0-0.okd-2022-01-29-035536   True        False         False      31m
machine-api                                4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
machine-approver                           4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
machine-config                             4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
marketplace                                4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
monitoring                                 4.9.0-0.okd-2022-01-29-035536   True        False         False      6d8h
network                                    4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
node-tuning                                4.9.0-0.okd-2022-01-29-035536   True        False         False      32m
openshift-apiserver                        4.9.0-0.okd-2022-01-29-035536   True        False         False      29h
openshift-controller-manager               4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
openshift-samples                          4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
operator-lifecycle-manager                 4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
operator-lifecycle-manager-catalog         4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
operator-lifecycle-manager-packageserver   4.9.0-0.okd-2022-01-29-035536   True        False         False      3d9h
service-ca                                 4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
storage                                    4.9.0-0.okd-2022-01-29-035536   True        False         False      14d
```

`oc get machineconfigpools` or `oc get mcp`

```bash
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master   rendered-master-28a6d31181071d3d6c5aa84ef859bfa8   True      False      False      3              3                   3                     0                      14d
worker   rendered-worker-58f0014151041d84bc0c18875388ba8b   True      False      False      3              3                   3                     0                      14d
```

### 3. Check list to be looking for in grafana

add mapp host name to IP address in this file `C:\Windows\System32\drivers\etc` and add ip adress and list of the host name for example:

```
# add this before end of file
102.54.94.97    rhino.acme.com         
38.25.63.10     x.acme.com
# End of section 
```

> note:
> forgot TBD

to view application logs open grafana and click on box in side bar then select manage > default > TBD

- [ ] Grafana graph resource utilization for each node

  - [ ] CPU
  - [ ] Memory
  - [ ] Disk

to view application logs open grafana and click on box in side bar then select manage > default > Kubernetes / Compute Resources / Namespace (Pods)

> note:
> select time range to 2 weeks
> select refresh range to 5 seconds interval

- [ ] Grafana graph resource utilization for each application

  - [ ] CPU
  - [ ] Memory