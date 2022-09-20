---
id: pnmk1zllw1oujn0up0kfxvo
title: Bare Metal Installation
desc: ''
updated: 1652964375823
created: 1652243506310
---


## Requrement to install OKD/OpenShift on bare metal

> NOTE: example here is using okd version 4.10.0-0.okd-2022-05-07-021833

### OKD/OCP installer

download okd/openshift installer from release page and extract it

* [OKD Releases Page](https://github.com/openshift/okd/releases)
* [OCP Releases Page](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/)

```bash
wget https://github.com/openshift/okd/releases/download/4.10.0-0.okd-2022-05-07-021833/openshift-install-linux-4.10.0-0.okd-2022-05-07-021833.tar.gz
tar -xf openshift-install-linux-4.10.0-0.okd-2022-05-07-021833.tar.gz
```

### Check OS requirement

exstract downloaded installer and run following comand to find iso required by cluster.

```bash
./openshift-install coreos print-stream-json
```

The output should be like below

```json
"iso": {
    "disk": {
    "location": "https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/35.20220327.3.0/aarch64/fedora-coreos-35.20220327.3.0-live.aarch64.iso",
    "signature": "https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/35.20220327.3.0/aarch64/fedora-coreos-35.20220327.3.0-live.aarch64.iso.sig",
    "sha256": "d7f1accac2eb03f68089f30d921bd49310b24501a0364e7b9e28624bcf0b6fab"
                            }
                        },
```

you can download the base os using the link from "location" field above, depend on your env you might need other version that available on the json file.

```bash
wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/35.20220327.3.0/aarch64/fedora-coreos-35.20220327.3.0-live.aarch64.iso
```

### Instance List

Instance Name | vCPU | Memory | Disk | OS |
--------------|------|--------|------|----|
 Helper/svc | 4 Core | 6GB | 150GB | RHEL8 |
 Bootstrap | 6 Core | 10 GB | 50GB | RHCOS |
 Master00 | 4 Core | 8 GB | 50 GB | RHCOS |
 Master01 | 4 Core | 8 GB | 50 GB | RHCOS |
 Master02 | 4 Core | 8 GB | 50 GB | RHCOS |
 Worker00 | 4 Core | 8 GB | 50 GB | RHCOS |
 Worker01 | 4 Core | 8 GB | 50 GB | RHCOS |

## Setup helper/svc instance

### 1. Install OS in helper/svc instance

* [ ] Remove the home dir partition and assign all free storage to '/'
* [ ] Optionally you can install the 'Guest Tools' package to have monitoring and reporting in the VMware ESXi dashboard
* [ ] Enable the LAN NIC only to obtain a DHCP address from the LAN network and make note of the IP address (ocp-svc_IP_address) assigned to the vm

### 2. Set a Static IP for internal OCP network interface using nmtui

* [ ] Address: 192.168.22.1
* [ ] DNS Server: 127.0.0.1
* [ ] Search domain: ocp.lan
* [ ] Never use this network for default route
* [ ] Automatically connect

### 3. Update OS

```bash
dnf update -y
```

### 4. Install git

```bash
dnf install -y git
```

### 5. Setup Firewalld

```bash
ifconfig #View netwwork interface name
nmcli connection modify <network-interface> connection.zone internal
nmcli connection modify <network-interface> connection.zone internal
```

Set masquerading (source-nat) on the both zones.

```bash
firewall-cmd --zone=external --add-masquerade --permanent
firewall-cmd --zone=internal --add-masquerade --permanent

# reload settings
firewall-cmd --reload

# check settings
firewall-cmd --list-all --zone=internal
firewall-cmd --list-all --zone=external

# check ipforwarding
cat /proc/sys/net/ipv4/ip_forward
```

### 6. Install and configure BIND DNS

```bash
dnf install bind bind-utils -y
```

Apply config from template

```bash
\cp ~/ocp4-metal-install/dns/named.conf /etc/named.conf
cp -R ~/ocp4-metal-install/dns/zones /etc/named/
```

Configure the firewall for DNS

```bash
firewall-cmd --add-port=53/udp --zone=internal --permanent
# for OCP 4.9 and later 53/tcp is required
firewall-cmd --add-port=53/tcp --zone=internal --permanent
firewall-cmd --reload
```

Enable and start the service

```bash
systemctl enable named
systemctl start named
systemctl status named
```

> At the moment DNS will still be pointing to the LAN DNS server. You can see this by testing with `dig ocp.lan`

Change the LAN nic (ens192) to use 127.0.0.1 for DNS AND ensure `Ignore automatically Obtained DNS parameters` is ticked

```bash
nmtui
```

Restart NetworkManager and check dig sees the correct DNS results by using the DNS Server running locally

```bash
systemctl restart NetworkManager

dig ocp.lan
# The following should return the answer ocp-bootstrap.lab.ocp.lan from the local server
dig -x 192.168.22.40
```

### 7. Install & configure DHCP

Install DHCP Server

```bash
dnf install dhcp-server -y
```

Edit dhcp config file located on `/etc/dhcp/dhcpd.conf`, you can also backup default config file and edit it manually.

```bash
vim /etc/dhcp/dhcpd.conf
```

Configure the Firewall
```bash
firewall-cmd --add-service=dhcp --zone=internal --permanent
firewall-cmd --reload
```

Enable and start the service
```bash
systemctl enable dhcpd
systemctl start dhcpd
systemctl status dhcpd
```

### 8. Install & configure Apache Web Server

Install APache Web Server

```bash
dnf install httpd -y
```

Change default listen port to 8080 in httpd.conf located on `/etc/httpd/conf/httpd.conf`

```bash
vim /etc/httpd/conf/httpd.conf
# OR
sed -i 's/Listen 80/Listen 0.0.0.0:8080/' /etc/httpd/conf/httpd.conf
```

Configure the firewall for Web Server traffic

```bash
firewall-cmd --add-port=8080/tcp --zone=internal --permanent
firewall-cmd --reload
```

Enable and start the service

```bash
systemctl enable httpd
systemctl start httpd
systemctl status httpd
```

### 9. Install & configure HAProxy

Install HAProxy

```bash
dnf install haproxy -y
```

Edit HAProxy config file located on `/etc/haproxy/haproxy.cfg`, you can also backup default config file and edit it manually.

```bash
vim /etc/haproxy/haproxy.cfg
```

Configure the Firewall
>Note: Opening port 9000 in the external zone allows access to HAProxy stats that are useful for monitoring and troubleshooting. The UI can be accessed at: `http://{ocp-svc_IP_address}:9000/stats`

```bash
firewall-cmd --add-port=6443/tcp --zone=internal --permanent # kube-api-server on control plane nodes
firewall-cmd --add-port=6443/tcp --zone=external --permanent # kube-api-server on control plane nodes
firewall-cmd --add-port=22623/tcp --zone=internal --permanent # machine-config server
firewall-cmd --add-service=http --zone=internal --permanent # web services hosted on worker nodes
firewall-cmd --add-service=http --zone=external --permanent # web services hosted on worker nodes
firewall-cmd --add-service=https --zone=internal --permanent # web services hosted on worker nodes
firewall-cmd --add-service=https --zone=external --permanent # web services hosted on worker nodes
firewall-cmd --add-port=9000/tcp --zone=external --permanent # HAProxy Stats
firewall-cmd --reload
```

Enable and start the service

```bash
setsebool -P haproxy_connect_any 1 # SELinux name_bind access
systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy
```

### 10. Install & configure NFS Server

Install and configure NFS for the OpenShift Registry. It is a requirement to provide storage for the Registry, emptyDir can be specified if necessary.

```bash
dnf install nfs-utils -y
```

* Create the Share

Check available disk space and its location `df -h`

```bash
mkdir -p /shares/registry
chown -R nobody:nobody /shares/registry
chmod -R 777 /shares/registry
```

Export the Share

```bash
echo "/shares/registry  192.168.22.0/24(rw,sync,root_squash,no_subtree_check,no_wdelay)" > /etc/exports
exportfs -rv
```

Set Firewall rules:

```bash
firewall-cmd --zone=internal --add-service mountd --permanent
firewall-cmd --zone=internal --add-service rpc-bind --permanent
firewall-cmd --zone=internal --add-service nfs --permanent
firewall-cmd --reload
```

Enable and start the NFS related services

```bash
systemctl enable nfs-server rpcbind
systemctl start nfs-server rpcbind nfs-mountd
```

## Generate and host install files

### 1. Generate an SSH key pair keeping all default options

```bash
ssh-keygen
```

### 2. Create an install directory

```bash
mkdir ~/ocp-install
```

### 3. Copy/create the install-config.yaml to the install directory

example of install-config.yaml

```bash
./openshift-install create install-config.yaml
```

```yaml
apiVersion: v1
baseDomain: ocp.lan
compute:
  - hyperthreading: Enabled
    name: worker
    replicas: 0 
    # Replicas Must be set to 0 for User Provisioned Installation as worker nodes will be manually deployed.
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: your.cluster.name.here # Cluster name
networking:
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
    - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: '{"auths": ...}'
sshKey: "ssh-ed25519 AAAA..."
```
> Note: In here you must provide some spesific detail regarding your cluster, such as cluster name, your pull secret, and sshkey.

### 4. Generate Kubernetes manifest files

```bash
./openshift-install create manifests --dir ~/ocp-install
```

> Note: A warning is shown about making the control plane nodes schedulable. It is up to you if you want to run workloads on the Control Plane nodes. If you dont want to you can disable this with: `sed -i 's/mastersSchedulable: true/mastersSchedulable: false/' ~/ocp-install/manifests/cluster-scheduler-02-config.yml`. Make any other custom changes you like to the core Kubernetes manifest files.

Generate the Ignition config and Kubernetes auth files

```bash
./openshift-install create ignition-configs --dir ~/ocp-install/
```

### 5. Create a hosting directory to serve the configuration files for the OpenShift booting process and copy all the file on installation directory

```bash
mkdir /var/www/html/ocp4
cp -R ~/ocp-install/* /var/www/html/ocp4
```

### 6. Move the Core OS image to the web server directory (later you need to type this path multiple times so it is a good idea to shorten the name)

```bash
mv ~/rhcos-X.X.X-x86_64-metal.x86_64.raw.gz /var/www/html/ocp4/rhcos
```

Change ownership and permissions of the web server directory

```bash
chcon -R -t httpd_sys_content_t /var/www/html/ocp4/
chown -R apache: /var/www/html/ocp4/
chmod 755 /var/www/html/ocp4/
```

Confirm you can see all files added to the /var/www/html/ocp4/ dir through Apache

```bash
curl localhost:8080/ocp4/
```

## Deploy OpenShift

### 1. Power on the ocp-bootstrap host and ocp-cp-# hosts and select 'Tab' to enter boot configuration. Enter the following configuration:

```bash
# Bootstrap Node - ocp-bootstrap
coreos.inst.install_dev=vda coreos.inst.image_url=http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz coreos.inst.insecure=yes coreos.inst.ignition_url=http://192.168.22.1:8080/ocp4/bootstrap.ign

# Or if you waited for it boot, use the following command then just reboot after it finishes and make sure you remove the attached .iso
sudo coreos-installer install /dev/vda -u http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz -I http://192.168.22.1:8080/ocp4/bootstrap.ign --insecure --insecure-ignition
```

```bash
# Each of the Control Plane Nodes - ocp-cp-\#
coreos.inst.install_dev=vda coreos.inst.image_url=http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz coreos.inst.insecure=yes coreos.inst.ignition_url=http://192.168.22.1:8080/ocp4/master.ign

# Or if you waited for it boot, use the following command then just reboot after it finishes and make sure you remove the attached .iso
sudo coreos-installer install /dev/vda -u http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz -I http://192.168.22.1:8080/ocp4/master.ign --insecure --insecure-ignition
```

### 2. Power on the ocp-w-# hosts and select 'Tab' to enter boot configuration. Enter the following configuration:

```bash
# Each of the Worker Nodes - ocp-w-\#
coreos.inst.install_dev=sda coreos.inst.image_url=http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz coreos.inst.insecure=yes coreos.inst.ignition_url=http://192.168.22.1:8080/ocp4/worker.ign

# Or if you waited for it boot, use the following command then just reboot after it finishes and make sure you remove the attached .iso
sudo coreos-installer install /dev/sda -u http://192.168.22.1:8080/ocp4/rhcos/rhcos-48.84.202109241901-0-metal.x86_64.raw.gz -I http://192.168.22.1:8080/ocp4/worker.ign --insecure --insecure-ignition
```

## Monitor Bootstrap process

You can monitor the bootstrap process from the ocp-svc host at different log levels (debug, error, info)

```bash
./openshift-install --dir ~/ocp-install wait-for bootstrap-complete --log-level=debug
```

> Note Once bootstrapping is complete the ocp-boostrap node can be removed