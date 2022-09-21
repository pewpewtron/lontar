---
id: qta6rjf6i0bjuukdpt8j4sl
title: Create Users
desc: ''
updated: 1663773308087
created: 1663674198445
---

## Create new user with new identity provider

Checklist before create new user via new identity provider

- [ ] httpasswd file containing username and encrypted password
- [ ] Oauth yaml configuration

Create new identity provider

### 1. Add identity provider

to create htpasswd file user need to install htpasswd tools see here [install htpasswd.](https://command-not-found.com/htpasswd)

create new htpasswd file with user

```bash
# Create New htpasswd file (-c used for create new file)
htpasswd -c -B -b htpasswd-file user-name user-password

# Add new user with password to existing htpasswd file
htpasswd -b  -b htpasswd-file user-name1 user-password1
```

Create secret with the generated passwords:

```bash
oc create secret generic secret_name --from-file htpasswd=htpasswd-file -n openshift-config
```

Modify the Identity Provider by adding secret that created above:

Get existing oauth from cluster

```bash
oc get oauth cluster -o yaml > oauth.yaml
```

edit oauth by adding new identity provider

```yaml
spec: 
  identityProviders:
  - name: identity-provider-name
    type: HTPasswd
    mappingMethod: claim
    htpasswd:
      fileData:
        name: secret_name
```

Update the Identity Provider:

```bash
oc replace -f oauth.yaml
```

wait for authentication pod to deploy

```bash
oc get pod -n openshift-authentication
```

### 2. Add user to existing htpasswd identity provider

adding new user to existing identity provider user need to modify secret value that made from htpasswd file. to find this move to `openshift-config` namespace and you will see the secret mentioned.

```bash
oc get secret -n openshift-config 
```

since our secret is created from htpasswd file we have 2 option to update the content

#### 2.1. Change local htpasswd and apply to the secret

User need to update it htpasswd file (add new user) before can apply the change

```bash
htpasswd -b  -b htpasswd-file user-nameN user-passwordN
```

After htpasswd file is updated new user can apply new file to existing secret that contain existing user

```bash
oc set data secret/secret_name --from-file=htpasswd=htpasswd-file -n openshift-config
```

#### 2.2 Edit secret via web console

adding new user also can be achieve from web console by navigating to the `secret` section located on side bar on workload section then change project to `openshift-config`, locate your secret containing your htpasswd, open it then on action button select edit secret. add your new user name and encrypted password from htpasswd command.

> **Note**
>
> Existing pod wont update its secret automaticaly when secret is updated, instead pod need to be restarted/rollout to make new secret available.
