---
layout:     post
title:      "Max open files limit"
author:     "Iceberg"
catalog:    true
header-img: "assets/images/sky1.jpg"
tags:
  - Performance Tuning
---

## System wide open files limit

To check system wide files limit:

```shell
$ cat /proc/sys/fs/file-max
4875932
$ sysctl -a | grep file-max
fs.file-max = 4875932
```

To change system wide files limit:

```
$ echo "fs.file-max = 4875932" >> /etc/sysctl.conf
$ sysctl -p /etc/sysctl.conf
```

