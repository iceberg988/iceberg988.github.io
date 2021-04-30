---
layout:     post
title:      "Using gstack for docker container process"
subtitle:   ""
author:     "Iceberg"
catalog:    false
header-img: "assets/images/header/yellowstone8.jpg"
tags:
  - Performance
  - Profiling/Tracing
  - Docker
---

In a Docker container environment, we won't get a valid stack trace directly on the container host as below.

```shell
$  ps -ef |grep smbd
root     171118 167977  0 Apr22 ?        00:00:02 /usr/sbin/smbd --foreground --no-process-group
root     171166 171118  0 Apr22 ?        00:00:00 /usr/sbin/smbd --foreground --no-process-group
root     171168 171118  0 Apr22 ?        00:00:00 /usr/sbin/smbd --foreground --no-process-group
root     171208 171118  0 Apr22 ?        00:00:00 /usr/sbin/smbd --foreground --no-process-group
root     190574 186140  0 15:01 pts/3    00:00:00 grep --color=auto smbd

$  gstack 171118
#0  0x00007fb66768afb3 in ?? () from /lib64/libc.so.6
#1  0x00007fb667b70f1b in ?? ()
#2  0x0000000000000060 in ?? ()
#3  0x00007fb668a92f5c in ?? ()
#4  0x0000000000000000 in ?? ()
```

To print the stack trace for a process running inside container, we can do the following.

1. Get the process id inside container

   ```shell
   $  docker exec -it iNfcP_9-0-1 bash

   bash-4.2# ps -ef | grep smbd
   root       1864      1  0 Apr22 ?        00:00:02 /usr/sbin/smbd --foreground --no-process-group
   root       1908   1864  0 Apr22 ?        00:00:00 /usr/sbin/smbd --foreground --no-process-group
   root       1910   1864  0 Apr22 ?        00:00:00 /usr/sbin/smbd --foreground --no-process-group
   root       1950   1864  0 Apr22 ?        00:00:00 /usr/sbin/smbd --foreground --no-process-group
   root     177346 177338  0 15:02 ?        00:00:00 grep smbd
   ```

2. Get the container instance pid

   ```shell
   $  /bin/docker inspect --format '\{\{ \.State\.Pid \}\}' e2590333640e
   167977
   ```

3. Print stack trace with gstack as below
 
   ```shell 
   $  /bin/nsenter -Z -m -n -p -t 167977 /bin/gstack 1864
   #0  0x00007fb66768afb3 in __epoll_wait_nocancel () from /lib64/libc.so.6
   #1  0x00007fb667b70f1b in epoll_event_loop_once () from /lib64/libtevent.so.0
   #2  0x00007fb667b6f057 in std_event_loop_once () from /lib64/libtevent.so.0
   #3  0x00007fb667b6a25d in _tevent_loop_once () from /lib64/libtevent.so.0
   #4  0x00007fb667b6a4bb in tevent_common_loop_wait () from /lib64/libtevent.so.0
   #5  0x00007fb667b6eff7 in std_event_loop_wait () from /lib64/libtevent.so.0
   #6  0x00005588a9175a98 in main ()
   ```


We use nsenter to get the stack trace of the target process in contaienr namespace.

```shell
$ man nsenter

NAME
       nsenter - run program with namespaces of other processes

SYNOPSIS
       nsenter [options] [program [arguments]]

DESCRIPTION
       Enters the namespaces of one or more other processes and then executes the specified program.  Enterable namespaces are:

       mount namespace
              Mounting  and  unmounting  filesystems  will  not affect the rest of the system (CLONE_NEWNS flag), except for filesystems which are explicitly marked as
              shared (with mount --make-shared; see /proc/self/mountinfo for the shared flag).

       UTS namespace
              Setting hostname or domainname will not affect the rest of the system.  (CLONE_NEWUTS flag)

       IPC namespace
              The process will have an independent namespace for System V message queues, semaphore sets and shared memory segments.  (CLONE_NEWIPC flag)

       network namespace
              The process will have independent IPv4 and IPv6 stacks, IP routing tables, firewall rules, the /proc/net and  /sys/class/net  directory  trees,  sockets,
              etc.  (CLONE_NEWNET flag)

       PID namespace
              Children  will have a set of PID to process mappings separate from the nsenter process (CLONE_NEWPID flag).  nsenter will fork by default if changing the
              PID namespace, so that the new program and its children share the same PID namespace and are visible to each other.  If --no-fork is used, the  new  pro‐
              gram will be exec'ed without forking.

       user namespace
              The process will have a distinct set of UIDs, GIDs and capabilities.  (CLONE_NEWUSER flag)

       See clone(2) for the exact semantics of the flags.

       If program is not given, then ``${SHELL}'' is run (default: /bin/sh).
```
