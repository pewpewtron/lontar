---
id: mw90kbzw4ehibp5ebal7ipy
title: Service Account and Security Context Constraints
desc: ''
updated: 1669191224429
created: 1669189499216
---

## Service Account overview

A service account is an OpenShift Container Platform account that allows a component to directly access the API. Service accounts are API objects that exist within each project. Service accounts provide a flexible way to control API access without sharing a regular user’s credentials.

```bash
oc create sa <service_account_name>
```

## Security Context Constraints

Red Hat OpenShift provides security context constraints (SCCs), a security mechanism that
restricts access to resources, but not to operations in OpenShift.

SCCs limits the access from a running pod in OpenShift to the host environment. SCCs control:

* Running privileged containers
* Requesting extra capabilities to a container
* Using host directories as volumes
* Changing the SELinux context of a container
* Changing the user ID

Some containers developed by the community might require relaxed security context constraints
to access resources that are forbidden by default, such as file systems, sockets or to access a
SELinux context

OpenShift provides eight SCCs:

* anyuid
* hostaccess
* hostmount-anyuid
* hostnetwork
* node-exporter
* nonroot
* privileged
* restricted

To change the container to run using a different SCC, you need to create a service account bound
to a pod.

### grant access to a specific service account

```bash
oc policy add-role-to-user <role_name> -z <service_account_name>
```
