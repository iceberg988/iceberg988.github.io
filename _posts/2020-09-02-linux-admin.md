---
layout:     post
title:      "Linux Administration"
author:     "Iceberg"
header-img: "assets/images/yellowstone5.jpg"
catalog:    true
tags:
  - Linux/Kernel
---

## Package management

rpm is a powerful Package Manager for Red Hat, Suse and Fedora Linux. It can be used to build, install, query, verify, update, and remove/erase individual software packages. A Package consists of an archive of files, and package information, including name, version, and description:

![Image](/assets/images/posts/rpm-command-cheat-sheet.png){:.shadow}

## Enable passwordless ssh login

```shell
$rpm -ivh sshpass-1.06-1.el7.x86_64.rpm

$ for i in `seq 0 239`
do
    sshpass -p "password" ssh-copy-id -i /root/.ssh/id_rsa.pub -o StrictHostKeyChecking=no server$i
done
```