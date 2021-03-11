---
layout:     post
title:      "Semaphore"
subtitle:   ""
author:     "Iceberg"
catalog:    true
header-style: text
tags:
  - Performance
  - Linux/Kernel
  - Tuning
---

## What is semaphore

A semaphore is a very relaxed type of lockable object. A given semaphore has a predefined maximum count, and a current count. You take ownership of a semaphore with a wait operation, also referred to as decrementing the semaphore, or even just abstractly called P. You release ownership with a signal operation, also referred to as incrementing the semaphore, a post operation, or abstractly called V. The single-letter operation names are from Dijkstra’s original paper on semaphores.

Every time you wait on a semaphore, you decrease the current count. If the count was greater than zero then the decrement just happens, and the wait call returns. If the count was already zero then it cannot be decremented, so the wait call will block until another thread increases the count by signaling the semaphore.

## Semaphore tuning

To display the semaphore limits:

```shell
$ cat /proc/sys/kernel/sem
300	307200	32	1024

$ sysctl -a | grep sem
kernel.sem = 300	307200	32	1024

$ ipcs -l | grep -i "sem"
------ Semaphore Limits --------
max semaphores per array = 300
max semaphores system wide = 307200
max ops per semop call = 32
semaphore max value = 32767
```

The values of the semaphore parameters are displayed in the following order.

* SEMMSL - The maximum number of semaphores in a sempahore set.
* SEMMNS - A system-wide limit on the number of semaphores in all semaphore sets. The maximum number of sempahores in the system.
* SEMOPM - The maximum number of operations in a single semop call
* SEMMNI - A system-wide limit on the maximum number of semaphore identifiers (sempahore sets)  

To display the current semaphore status:

```shell
$ ipcs -u | egrep -i "used arrays|sem"
------ Semaphore Status --------
used arrays = 3
allocated semaphores = 3
```

To display the active semaphore sets info:

```shell
$ ipcs -s
------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x00000000 0          root       600        1
0x00000000 32769      root       600        1
0x00005653 229380     root       666        1
```

To adjust the semaphore values on the fly:

```shell
$ echo 300   307200   32   1024 > /proc/sys/kernel/sem
```

To modify system semaphore values permanently:

```shell
$ echo "kernel.sem = 300  307200  32  1024" >> /etc/sysctl.conf
$ sysctl -p 
```