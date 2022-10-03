---
id: fsxoczfjjcvklisu3rkvrp8
title: Allow Insecure Registry
desc: ''
updated: 1664101184304
created: 1664099721492
---

There are two ways you can use private insecure registries on OpenShift / OKD cluster.

1. If using self-signed SSL certificate – Import the certificate OpenShift CA trust.
2. Add the registry to insecure registries list – The Machine Config Operator (MCO) will push updates to all nodes in the cluster and reboot them.

## 1. Add additional trust stores for image registry access

## 2. Whitelisting insecure image registries (Easy way)

You can as well add an insecure registry by editing the image.config.openshift.io/cluster custom resource (CR). This is common for registries which only support HTTP connections or have invalid certificates.

```bash
oc edit image.config.openshift.io/cluster
```

the output will look like this

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: config.openshift.io/v1
kind: Image
metadata:
  annotations:
    include.release.openshift.io/ibm-cloud-managed: "true"
    include.release.openshift.io/self-managed-high-availability: "true"
    include.release.openshift.io/single-node-developer: "true"
    release.openshift.io/create-only: "true"
  creationTimestamp: "2022-05-18T07:50:53Z"
  generation: 1
  name: cluster
  resourceVersion: "162011319"
  uid: d41a9534-7cab-4e14-88ef-2d092613c1be
spec: {}
status:
  externalRegistryHostnames:
  - default-route-openshift-image-registry.apps.lab.i3datacenter.my.id
  internalRegistryHostname: image-registry.openshift-image-registry.svc:5000
```

user need to add following line inside spec section to allow insecure registry registry

```yaml
spec:
  registrySources:
    insecureRegistries:
    - your.registry.com
    - 10.8.60.126:5000
```

final file will look like this

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: config.openshift.io/v1
kind: Image
metadata:
  annotations:
    include.release.openshift.io/ibm-cloud-managed: "true"
    include.release.openshift.io/self-managed-high-availability: "true"
    include.release.openshift.io/single-node-developer: "true"
    release.openshift.io/create-only: "true"
  creationTimestamp: "2022-05-18T07:50:53Z"
  generation: 1
  name: cluster
  resourceVersion: "162011319"
  uid: d41a9534-7cab-4e14-88ef-2d092613c1be
spec: 
  registrySources:
    insecureRegistries:
    - your.registry.com
    - 10.8.60.126:5000
status:
  externalRegistryHostnames:
  - default-route-openshift-image-registry.apps.lab.i3datacenter.my.id
  internalRegistryHostname: image-registry.openshift-image-registry.svc:5000
  
```

exit your editor and wait your node restarted.

The Machine Config Operator (MCO) watches the image.config.openshift.io/cluster for any changes to registries and reboots the nodes when it detects changes.
The new registry configurations are written to /etc/containers/registries.conf file on each node.

## Disable an registry

Disabling certain registry work the same way as above. you need to add `blockedRegistries` section and specified registry you want to disable.

```yaml
spec:
  additionalTrustedCA:
    name: registry-config
  registrySources:
    insecureRegistries:
    - ocr.example.com
    blockedRegistries:
    - untrusted.com
```
