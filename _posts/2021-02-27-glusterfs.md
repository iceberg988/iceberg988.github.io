---
layout:     post
title:      "GlusterFS"
subtitle:   "A distributed file system"
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/carmel1.jpg"
tags:
  - Distributed System
  - Gluster
  - Performance  
---

## What is Gluster?

[Gluster](https://docs.gluster.org/en/latest/Administrator-Guide/GlusterFS-Introduction/) is a scalable, distributed file system that aggregates disk storage resources from multiple servers into a single global namespace.

Advantages

* Scales to several petabytes
* Handles thousands of clients
* POSIX compatible
* Uses commodity hardware
* Can use any ondisk filesystem that supports extended attributes
* Accessible using industry standard protocols like NFS and SMB
* Provides replication, quotas, geo-replication, snapshots and bitrot detection
* Allows optimization for different workloads
* Open Source

## Installation and configuration 

1. To install gluster and start gluster service:

   ```shell
   [root@centos83-1 ~]# cat /etc/centos-release
   CentOS Linux release 8.3.2011

   [root@centos83-1 ~]# systemctl stop firewalld
   [root@centos83-1 ~]# systemctl disable firewalld

   [root@centos83-1 ~]# yum install -y centos-release-gluster
   [root@centos83-1 ~]# yum install -y glusterfs-server
   [root@centos83-1 ~]# rpm -qa |grep gluster
   glusterfs-cli-8.3-1.el8.x86_64
   libvirt-daemon-driver-storage-gluster-6.0.0-28.module_el8.3.0+555+a55c8938.x86_64
   glusterfs-client-xlators-8.3-1.el8.x86_64
   qemu-kvm-block-gluster-4.2.0-34.module_el8.3.0+555+a55c8938.x86_64
   libglusterd0-8.3-1.el8.x86_64
   glusterfs-8.3-1.el8.x86_64
   pcp-pmda-gluster-5.1.1-3.el8.x86_64
   glusterfs-fuse-8.3-1.el8.x86_64
   centos-release-gluster8-1.0-1.el8.noarch
   libglusterfs0-8.3-1.el8.x86_64
   glusterfs-server-8.3-1.el8.x86_64

   [root@centos83-1 ~]# systemctl enable glusterd
   [root@centos83-1 ~]# systemctl restart glusterd

   [root@centos83-1 ~]# systemctl status glusterd
    glusterd.service - GlusterFS, a clustered file-system server
    Loaded: loaded (/usr/lib/systemd/system/glusterd.service; enabled; vendor preset: enabled)
    Active: active (running) since Tue 2021-01-05 17:28:42 PST; 1 months 22 days ago
     Docs: man:glusterd(8)
    Main PID: 1420 (glusterd)
    Tasks: 26 (limit: 409792)
    Memory: 152.3M
    CGroup: /system.slice/glusterd.service
   ```

2. To form a trusted storage pool with the second server:

   ```shell
   [root@centos83-1 ~]# gluster peer probe centos83-2
   [root@centos83-1 ~]# gluster peer status
   Number of Peers: 1

   Hostname: centos83-2
   Uuid: b07d3d6e-4d6e-42a9-ad21-018223843fd5
   State: Peer in Cluster (Connected)
   ```

3. To create brick on the first server:

   ```shell
   [root@centos83-1 ~]# lsblk | egrep "NAME|sdb"
   NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sdb                  8:16   0    1T  0 disk

   [root@centos83-1 ~]# pvcreate /dev/sdb
   [root@centos83-1 ~]# vgcreate vg_bricks /dev/sdb
   [root@centos83-1 ~]# lvcreate -L 800g -n gfslv1 vg_bricks
   [root@centos83-1 ~]# mkfs.xfs /dev/vg_bricks/gfslv1
   [root@centos83-1 ~]# mkdir -p /bricks/vm1_brick1
   [root@centos83-1 ~]# vim /etc/fstab
   /dev/vg_bricks/gfslv1 /bricks/vm1_brick1        xfs     defaults        0 0
   [root@centos83-1 ~]# mount -a
   [root@centos83-1 ~]# df -h |grep gfs
   /dev/mapper/vg_bricks-gfslv1  800G  5.7G  794G   1% /bricks/vm1_brick1
   ```

4. To create brick on the second server:

   ```shell
   [root@centos83-2 ~]# lsblk | egrep "NAME|sdb"
   NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sdb                  8:16   0    1T  0 disk

   [root@centos83-2 ~]# pvcreate /dev/sdb
   [root@centos83-2 ~]# vgcreate vg_bricks /dev/sdb
   [root@centos83-2 ~]# lvcreate -L 800g -n gfslv1 vg_bricks
   [root@centos83-2 ~]# mkfs.xfs /dev/vg_bricks/gfslv1
   [root@centos83-2 ~]# mkdir -p /bricks/vm2_brick1
   [root@centos83-2 ~]# vim /etc/fstab
   /dev/vg_bricks/gfslv1 /bricks/vm2_brick1        xfs     defaults        0 0
   [root@centos83-2 ~]# mount -a
   [root@centos83-2 ~]# df -h |grep gfs
   /dev/mapper/vg_bricks-gfslv1  800G  5.7G  794G   1% /bricks/vm2_brick1
   ```

5. To create distributed volume with the two bricks which are created on the two nodes:

   ```shell
   [root@centos83-1 ~]# gluster volume create gv0 centos83-1:/bricks/vm1_brick1/gv0 centos83-2:/bricks/vm2_brick1/gv0
   [root@centos83-1 ~]# gluster volume start gv0
   ```

6. To verify the volume status:

   ```shell
   [root@centos83-1 ~]#  gluster volume info gv0

   Volume Name: gv0
   Type: Distribute
   Volume ID: ee08d16a-f940-4ec2-aba8-5f1fcfe41bd4
   Status: Started
   Snapshot Count: 0
   Number of Bricks: 2
   Transport-type: tcp
   Bricks:
   Brick1: centos83-1:/bricks/vm1_brick1/gv0
   Brick2: centos83-2:/bricks/vm2_brick1/gv0
   Options Reconfigured:
   storage.fips-mode-rchecksum: on
   transport.address-family: inet
   nfs.disable: on
   ```

7. To mount the distributed volume on one of the servers(treat it as client for simple demonstration):

   ```shell
   [root@centos83-1 ~]# mkdir /testmnt
   [root@centos83-1 ~]# mount -t glusterfs centos83-2:/gv0 /testmnt
   [root@centos83-1 ~]# df -h | grep testmnt
   centos83-2:/gv0               1.6T   28G  1.6T   2% /testmnt
   ```
   
   As shown above, the usable storage size is the sum of the brick size from two nodes.

## References

* <https://docs.gluster.org/en/latest/Administrator-Guide/GlusterFS-Introduction/>