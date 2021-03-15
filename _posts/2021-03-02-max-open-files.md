---
layout:     post
title:      "Max open files limit"
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/sky1.jpg"
tags:
  - Performance Tuning
  - Performance  
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



## User level open files limit

To check hard/soft limits:

```shell
$ ulimit -Hn
40960
$ ulimit -Sn
40960
```

To change hard/soft limits:

```shell
$ vi /etc/security/limits.conf
*  hard nofile 40960
*  soft nofile 40960
```

## Process level open files limit

To check the max open files per process:

```shell
$ cat /proc/sys/fs/nr_open
1048576
```

To check the specific process max open files limit:

```shell
$ cat /proc/`pidof <process-name>`/limits | egrep "Limit |Max open files"
Limit                     Soft Limit           Hard Limit           Units
Max open files            524352               524352               files
```

Sometimes, application process may need change the max open files limits on the fly. 

In Docker container, the process does not have the permission to do so by default. 

Docker provides the following two ways to extend Linux capabilities for the container processes.

* --cap-add	    Add Linux capabilities
* --cap-drop	Drop Linux capabilities
* --privileged	Give extended privileges to this container


When using the "--privileged" option is not allowed for security reason, we can have fine grain control over the capabilities using --cap-add and --cap-drop. 

For example, if we want to grant the container process the permission to change max open files limit on the fly, we can use the following capability option.

* SYS_RESOURCE - Override resource Limits.

We can pass this option to the target docker container.

```shell
$ docker run --cap-add=SYS_ADMIN ...
```

## References

* <https://docs.docker.com/engine/reference/run/>