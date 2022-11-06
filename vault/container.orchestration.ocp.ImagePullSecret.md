---
id: dd5wfeuak0qj7kanwhphjmm
title: ImagePullSecret
desc: ''
updated: 1665049688464
created: 1665049415430
---

Before you can pulling image from a registry, sometimes its require credential to authenticate, in Openshift you can create image pull secret

```bash
oc create secret docker-registry <pull_secret_name> \
    --docker-server=<registry_server> \
    --docker-username=<user_name> \
    --docker-password=<password> \
    --docker-email=<email>
```

if you have docker `.dockercfg` file you can use that and adding it to your secret for pulling an image

```bash
oc create secret generic <pull_secret_name> \
    --from-file=.dockercfg=<path/to/.dockercfg> \
    --type=kubernetes.io/dockercfg
```

Or if you have a $HOME/.docker/config.json file:

```bash
oc create secret generic <pull_secret_name> \
    --from-file=.dockerconfigjson=<path/to/.docker/config.json> \
    --type=kubernetes.io/dockerconfigjson
```

To use a secret for pulling images for pods, you must add the secret to your service account. The name of the service account in this example should match the name of the service account the pod uses. The default service account is default:

```bash
oc secrets link default <pull_secret_name> --for=pull
```