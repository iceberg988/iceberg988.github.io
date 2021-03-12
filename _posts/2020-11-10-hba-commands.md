---
layout:     post
title:      "Useful HBA commands"
author:     "Iceberg"
header-img: "assets/images/yellowstone8.jpg"
catalog: true
tags:
  - Fibre Channel
---

## Check HBA card type

```shell
$  lspci | grep -i fibre
18:00.0 Fibre Channel: QLogic Corp. ISP2722-based 16/32Gb Fibre Channel to PCIe Adapter (rev 01)
18:00.1 Fibre Channel: QLogic Corp. ISP2722-based 16/32Gb Fibre Channel to PCIe Adapter (rev 01)
```

## Physical slot, driver, module information

```shell
$  lspci -v -s 18:00.0
18:00.0 Fibre Channel: QLogic Corp. ISP2722-based 16/32Gb Fibre Channel to PCIe Adapter (rev 01)
	Subsystem: QLogic Corp. QLE2692 Dual Port 16Gb Fibre Channel to PCIe Adapter
	Flags: bus master, fast devsel, latency 0, IRQ 231, NUMA node 0
	Memory at aab05000 (64-bit, prefetchable) [size=4K]
	Memory at aab02000 (64-bit, prefetchable) [size=8K]
	Memory at aaa00000 (64-bit, prefetchable) [size=1M]
	Expansion ROM at 9d900000 [disabled] [size=256K]
	Capabilities: [44] Power Management version 3
	Capabilities: [4c] Express Endpoint, MSI 00
	Capabilities: [88] Vital Product Data
	Capabilities: [90] MSI-X: Enable+ Count=16 Masked-
	Capabilities: [9c] Vendor Specific Information: Len=0c <?>
	Capabilities: [100] Advanced Error Reporting
	Capabilities: [154] Alternative Routing-ID Interpretation (ARI)
	Capabilities: [1c0] #19
	Capabilities: [1f4] Vendor Specific Information: ID=0001 Rev=1 Len=014 <?>
	Kernel driver in use: qla2xxx
	Kernel modules: qla2xxx
```

## Check if the driver/module loaded

```shell
$  lsmod | grep qla2xxx
qla2xxx               792059  246
nvme_fc                33640  1 qla2xxx
scsi_transport_fc      64007  1 qla2xxx
```

## Check module file name

```shell
$  modinfo -n qla2xxx
/lib/modules/3.10.0-1062.12.1.el7.x86_64/kernel/drivers/scsi/qla2xxx/qla2xxx.ko
```

## Check HBA driver and version

```shell
$  modinfo -d qla2xxx
QLogic Fibre Channel HBA Driver
QLogic Fibre Channel HBA Driver (Target Mode Support, including 24xx+ ISP)

$  modinfo qla2xxx | grep version
version:        8.07.00.34.Trunk-SCST.19-k
rhelversion:    7.7
srcversion:     B6770D64FCF8A0A0273AD2C
vermagic:       3.10.0-1062.12.1.el7.x86_64 SMP mod_unload modversions
```

## Check if the running driver is same as kernel

```shell
$  modinfo -k `uname -r` -n qla2xxx
/lib/modules/3.10.0-1062.12.1.el7.x86_64/kernel/drivers/scsi/qla2xxx/qla2xxx.ko
```

## Remove current module/driver

```shell
$  modprobe -r qla2xxx
```

## Show dependent devices/modules

```shell
$  modprobe --show-depends qla2xxx
insmod /lib/modules/3.10.0-1062.12.1.el7.x86_64/kernel/drivers/scsi/scsi_tgt.ko.xz
insmod /lib/modules/3.10.0-1062.12.1.el7.x86_64/kernel/drivers/scsi/scsi_transport_fc.ko.xz
insmod /lib/modules/3.10.0-1062.12.1.el7.x86_64/kernel/drivers/scsi/qla2xxx/qla2xxx.ko ql2xfwloadbin=2
```

## Check the available HBA ports

```shell
$  ls -l /sys/class/fc_host
total 0
lrwxrwxrwx. 1 root root 0 Mar  9 13:01 host2 -> ../../devices/pci0000:17/0000:17:00.0/0000:18:00.0/host2/fc_host/host2
lrwxrwxrwx. 1 root root 0 Mar  9 13:01 host3 -> ../../devices/pci0000:17/0000:17:00.0/0000:18:00.1/host3/fc_host/host3
```

## View used HBA ports on server

```shell
$  ls -ltr /sys/class/fc_transport/
total 0
lrwxrwxrwx. 1 root root 0 Mar  9 13:01 target6:0:0 -> ../../devices/pci0000:85/0000:85:02.0/0000:87:00.0/host6/rport-6:0-0/target6:0:0/fc_transport/target6:0:0
lrwxrwxrwx. 1 root root 0 Mar  9 13:01 target4:0:0 -> ../../devices/pci0000:85/0000:85:00.0/0000:86:00.0/host4/rport-4:0-0/target4:0:0/fc_transport/target4:0:0
[...]
```

## Find the WWN numbers for your fc host

```shell
$  cat /sys/class/fc_host/host?/port_name
0x210034800d6baa70
0x210034800d6baa71
```

## Port state

```shell
$  cat /sys/class/fc_host/host?/port_state
Online
Online
```

## Queue depth

```shell
$  cat /sys/module/qla2xxx/parameters/ql2xmaxqdepth
64
```

To set/change the qdepth value on the fly:

```shell
$ echo 16 > /sys/module/qla2xxx/parameters/ql2xmaxqdepth
```

To set the qdepth value permanently:

```shell
$ modinfo qla2xxx | grep ql2xmaxqdepth
parm: ql2xmaxqdepth:Maximum queue depth to set for each LUN. Default is 32. (int)

$ vi /etc/modprobe.conf
alias scsi_hostadapter1 qla2xxx
options qla2xxx ql2xmaxqdepth=16
```

## Scan disks

```shell
$  ls /sys/class/scsi_host
host1  host2  host3  host4  host5  host6  host7

$  for x in `ls /sys/class/scsi_host`
> do
> echo "- - -" > /sys/class/scsi_host/$x/scan
> done
```

## Check HBA speed

```shell
$  grep -Hv "zz" /sys/class/fc_host/host*/speed
/sys/class/fc_host/host2/speed:16 Gbit
/sys/class/fc_host/host3/speed:16 Gbit
```

## Reference

* <https://linoxide.com/storage/scandetect-luns-redhat-linux-outputs-remember/>