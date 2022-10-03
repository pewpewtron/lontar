---
id: 48rvbys4bs9nz2x2zt4vosq
title: Firewalld
desc: ''
updated: 1664788617180
created: 1664787529438
---

Manage your firewall and opened port using firewalld, to install reffed here:
[firewalld](https://command-not-found.com/firewalld)

## Create New Firewall Rule

the firewall port can be opened as part of a pre-configured service. For example:

```bash
firewall-cmd --zone=public --permanent --add-service=http
```

Secondly, the ports can be open directly as custom user predefined ports. Example:

```bash
firewall-cmd --permanent --add-port 8080/tcp
```

## Check Firewall Rule

Check service ports opened:

```bash
firewall-cmd --list-services
```

Check for ports opened:

```bash
firewall-cmd --list-ports
```

Check for all open ports and services:

```bash
firewall-cmd --list-all
```

## Other tools to check port

You can use [nmap](https://command-not-found.com/nmap) to check port opened in your remote or local server,

```bash
# Check your hostname to check local exposed port
hostname

# Check Your local server opened port
nmap devsandbox

# Check remote server opened port
nmap <your-ip/address>
```

```output
Starting Nmap 7.70 ( https://nmap.org ) at 2022-10-03 05:09 EDT
Nmap scan report for shark2.labs (10.8.60.126)
Host is up (0.000037s latency).
Other addresses for shark2.labs (not scanned): fe80::98be:97ff:fe70:a4ec fe80::c4ac:40ff:fe26:e637 fe80::7cfb:fdff:feef:7f0c 10.88.0.1
Not shown: 993 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp
5001/tcp open  commplex-link
8081/tcp open  blackice-icecap
8084/tcp open  unknown
8090/tcp open  opsmessaging
9000/tcp open  cslistener

Nmap done: 1 IP address (1 host up) scanned in 1.66 seconds
```