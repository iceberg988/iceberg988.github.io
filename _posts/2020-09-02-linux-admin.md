---
layout:     post
title:      "Linux Administration"
author:     "Iceberg"
header-img: "assets/images/header/yellowstone5.jpg"
catalog:    true
tags:
  - Linux/Kernel
---

## Package management

Install the package:

```shell
$ rpm -ivh blktrace
```

Upgrade the package:

```shell
$ rpm -Uvh blktrace
```

Remove the installed package:

```shell
$ rpm -ev blktrace
$ rpm -ev --nodeps blktrace
```

Display the installed package info:

```shell
$ rpm -qi blktrace
Name        : blktrace                     Relocations: (not relocatable)
Version     : 1.0.1                             Vendor: Red Hat, Inc.
Release     : 6.el6                         Build Date: Wed 07 Sep 2011 12:51:04 PM PDT
Install Date: Fri 23 Dec 2016 12:15:57 PM PST      Build Host: x86-006.build.bos.redhat.com
Group       : Development/System            Source RPM: blktrace-1.0.1-6.el6.src.rpm
Size        : 1042319                          License: GPLv2+
Signature   : RSA/8, Fri 23 Sep 2011 04:19:31 AM PDT, Key ID 199e2f91fd431d51
Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
URL         : http://brick.kernel.dk/snaps
Summary     : Utilities for performing block layer IO tracing in the linux kernel
Description :
blktrace is a block layer IO tracing mechanism which provides detailed
information about request queue operations to user space.  This package
includes both blktrace, a utility which gathers event traces from the kernel;
and blkparse, a utility which formats trace data collected by blktrace.

You should install the blktrace package if you need to gather detailed
information about IO patterns.
```

Find out what package a file belongs to:

```shell
$ rpm -qf /usr/bin/blktrace
blktrace-1.0.1-6.el6.x86_64
```

Display list of configuration files for a package or command:

```shell
$ rpm -qc <package-name>
$ rpm -qcf /path/to/file
```

Download a package and its dependencies:

```shell
$ yum install yum-utils
$ yumdownloader --destdir=./ --resolve blktrace
```

## Network bonding

Configure LACP bonding without reboot:

```shell
$  cat /sys/class/net/bond0/bonding/mode
802.3ad 4
$  cat /sys/class/net/bond0/bonding/xmit_hash_policy
layer2 0
$  echo 1 > /sys/class/net/bond0/bonding/xmit_hash_policy
$  cat /sys/class/net/bond0/bonding/xmit_hash_policy
layer3+4 1
```

Configure LACP bonding permanent to reboot:

```shell
$ vi /etc/sysconfig/network-scripts/ifcfg-bond0
BONDING_OPTS="mode=802.3ad xmit_hash_policy=layer3+4"

$ service network restart
```