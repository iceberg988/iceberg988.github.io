---
layout:     post
title:      "Tuning kernel parameters in Docker"
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/yellowstone6.jpg"
tags:
  - Performance Tuning
  - Performance
  - Docker
---

# Configure namespaced kernel parameters(sysctl) at runtime

The --sysctl sets namespaced kernel parameters (sysctls) in the container. For example, to turn on IP forwarding in the containers network namespace, run this command:

```shell
$ docker run --sysctl net.ipv4.ip_forward=1 someimage
```

**Note**

Not all sysctls are namespaced. Docker does not support changing sysctls inside of a container that also modify the host system. As the kernel evolves we expect to see more sysctls become namespaced.

**CURRENTLY SUPPORTED SYSCTLS**

IPC Namespace:

* kernel.msgmax, kernel.msgmnb, kernel.msgmni, kernel.sem, kernel.shmall, kernel.shmmax, kernel.shmmni, kernel.shm_rmid_forced.
* Sysctls beginning with fs.mqueue.*
* If you use the --ipc=host option these sysctls are not allowed.

Network Namespace:

* Sysctls beginning with net.*
* If you use the --network=host option using these sysctls are not allowed.

# Hard and soft ulimit settings

There are two types of ulimit settings:
* The hard limit is the maximum value that is allowed for the soft limit. Any changes to the hard limit require root access.
* The soft limit is the value that Linux uses to limit the system resources for running processes. The soft limit cannot be greater than the hard limit.

## Updating hard and soft ulimit settings in Linux

```shell
To change the open files value on your operating system:
On RHEL and CentOS, edit the /etc/security/limits.d/91-nofile.conf file as shown in the following example:
@streamsadmin - nofile open-files-value
On SLES, edit the /etc/security/limits.conf file as shown in the following example:
@streamsadmin - nofile open-files-value

To change the max user processes value on your operating system:
On RHEL and CentOS, edit the /etc/security/limits.d/90-nproc.conf file as shown in the following example:
@streamsadmin hard nproc max-user-processes-value
@streamsadmin soft nproc max-user-processes-value
On SLES, edit the /etc/security/limits.conf file as shown in the following example:
@streamsadmin hard nproc max-user-processes-value
@streamsadmin soft nproc max-user-processes-value

To set the hard stack and soft stack values, add the following lines to the /etc/security/limits.conf file:
@streamsadmin hard stack unlimited 
@streamsadmin soft stack 20480

Use the following ulimit commands to verify the updated settings:
To verify the updated hard limit, enter the following command:
ulimit -aH
To verify the updated soft limit, enter the following command:
ulimit -aS
```

## Set ulimits in container (--ulimit)

Since setting ulimit settings in a container requires extra privileges not available in the default container, you can set these using the --ulimit flag. --ulimit is specified with a soft and hard limit as such: <type>=<soft limit>[:<hard limit>], for example:

```shell
$ docker run --ulimit nofile=1024:1024 --rm debian sh -c "ulimit -n"
1024
```

**Note**

If you do not provide a hard limit, the soft limit is used for both values. If no ulimits are set, they are inherited from the default ulimits set on the daemon. The as option is disabled now. In other words, the following script is not supported:

```shell
$ docker run -it --ulimit as=1024 fedora /bin/bash
```

The values are sent to the appropriate syscall as they are set. Docker doesn’t perform any byte conversion. Take this into account when setting the values.