---
layout:     post
title:      "Enable ssh passwordless login"
subtitle:   ""
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/yellowstone5.jpg"
tags:
  - Linux/Kernel
---

# Install sshpass package

To install sshpass package on the source server from which to enable ssh passwordless login to remote servers:

```shell
-bash-4.2# rpm -ivh sshpass-1.06-1.el7.x86_64.rpm
```

# Enable ssh passwordless login to remote servers

To enable ssh passwordless login to a large amount of remote servers, we can use the following command to automate without needs of providing password for each server.

```shell
-bash-4.2# for n in `seq 0 239`
do
    sshpass -p "password" ssh-copy-id -i /root/.ssh/id_rsa.pub -o StrictHostKeyChecking=no Client-hostname$n
done
```