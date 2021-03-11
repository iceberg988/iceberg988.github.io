---
layout:     post
title:      "Changed Block Tracking(CBT)"
subtitle:   ""
date:       2021-02-26 12:00:00
author:     "Iceberg"
catalog:    true
header-img: "assets/images/jackson1.jpg"
tags:
  - Virtualization
  - VMware
---

## Introduction

Changed Block Tracking is an incremental backup technology for virtual machines. It helps create faster and smaller backups. It has the following advantages.

* Reduce backup time
* Save disk space by only storing changed data to the previous backup

Block changes are tracked in the virtualization layer, outside the virtual machines. During a backup, only the changed block since the last backup are transmitted. For VMware, the vSphere APIs can be used to request the VMkernel to return the changed blocks from the last snapshot backup. Microsoft provides Resilient Change Tracking(RCT) to provide the native CBT feature for Hyper-V.

Veritas NetBackup Accelerator reduces the backup time for VMware backups. NetBackup uses VMware Changed Block Tracking (CBT) to identify the changes that were made within a virtual machine. Only the changed data blocks are sent to the NetBackup media server, to significantly reduce the I/O and backup time. The media server combines the new data with previous backup data and produces a traditional full NetBackup image that includes the complete virtual machine files.

## Resource

* <https://kb.vmware.com/s/article/1020128>
* <https://www.ibm.com/support/knowledgecenter/SSERB6_8.1.2/ve.hv/c_ve_hv_ovw_rct.html>
* <https://www.veritas.com/support/en_US/doc/21902280-127283730-0/v77418244-127283730>