---
layout:     post
title:      "Patrol read"
subtitle:   ""
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/coke1.jpg"
tags:
  - Performance
  - Storage
---

## What is patrolread
The Patrol Read feature is designed as a preventative measure to ensure hard drive health and data integrity. Patrol Read scans for and resolves potential problems on configured hard drives.

## Patrol Read Behavior
The following is an overview of Patrol Read behavior:

Patrol Read runs on all disks on the controller that are configured as part of a virtual disk, including hot spares.
Patrol Read does not run on hard drives that are not part of a virtual disk or are in Ready state.
Patrol Read adjusts the amount of controller resources that are dedicated to Patrol Read operations based on outstanding disk I/O. For example, if the system is busy processing I/O operation, then Patrol Read uses fewer resources to allow the I/O to take a higher priority.
Patrol Read does not run on any disks that are involved in any of the following operations:
Rebuild
Replace Member
Full or Background Initialization
Consistency Check

## Patrol Read Modes
Patrol Read Mode can be set in BIOS configuration utility and UEFI RAID Configuration Utility. The default Patrol Read mode is automatic. When Patrol Read is in automatic mode, Patrol Read runs automatically at its scheduled interval.

Below is a summary of the different Patrol Read modes:

In Auto Mode, Patrol Read will run every 168 hours (7 days). Whenever started, the duration depends on the size of storage. For NBFS, it would last 24 hours
In Auto Mode, Firmware allows Patrol Read to be started manually
In Auto Mode, Patrol Read can be stopped manually.
In Manual Mode, Patrol Read does not start automatically as per the scheduled time.
In Manual Mode, Patrol Read can be started manually.
In Manual Mode, Patrol Read can be stopped manually.
In Disabled Mode, Patrol Read will not run automatically.
In Disabled Mode, Patrol Read cannot be started manually.
Performance issues
We have observed a few performance issues that were caused by Patrol read.

For example on NBFS:

For restore performance, PR caused 50% - 65% drop at all dedupe rates.
For backup performance, PR caused max 20% drop when 0%, 50% and 80% dedupe. No impact for 98% dedupe backup.
For both backup & restore, when PR is running, see much higher IO service time (svctm) than baseline run. This should be due to the fact that PR would cause the disk head to move back and forth when host is also doing IOs