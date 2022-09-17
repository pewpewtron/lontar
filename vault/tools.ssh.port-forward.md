---
id: kmlcj6zf2p8nlwba3j6en6h
title: Port Forward
desc: ''
updated: 1663426680225
created: 1663425881728
---

Typical uses for local port forwarding include:

* Tunneling sessions and file transfers through jump servers

* Connecting to a service on an internal network from the outside

* Connecting to a remote file share over the Internet

port forwarding via ssh can be useful when you trying to troubleshot an issue in remote server. to achieve this use example command below.

```bash
ssh -L 2601:localhost:3306 user@your-server.com
```

Explanation:

ssh command above will connect to a server named your-server.com as user account, then with flag `-L` use for local port forwarding then it will forward port 3306 on your-server.com to port 2601 in local machine.

Command Breakdown

command | desc |
---------|----------
 ssh | call SSH command
 -L | Port forwarding function
 2601 | Local port where server port will be forwarded
 localhost | Localhost at the server
 3306 | Port at the server that will be forwarded
 user | Username for the server
 your-server.com | your remote machine address

