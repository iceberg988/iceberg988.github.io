---
layout:     post
title:      "Memory fragmentation"
date:       2020-07-23 12:00:00
author:     "Iceberg"
header-img: "assets/images/parasailing.jpg"
catalog: true
tags:
  - Linux/Kernel    
  - Memory Management
---

## Memory fragmentation

Memory page availability can be checked from /proc/buddyinfo as below.

```
$  cat /proc/buddyinfo
Node 0, zone      DMA      1      0      1      0      1      1      1      0      1      1      3
Node 0, zone    DMA32   3342   2441   2138   5025   1871    236      3      0      0      0      0
Node 0, zone   Normal 143135   6057 150803   4005    330     62      1      0      0      0      0
```

If you see the system log message file(/var/log/message) is reporting "kernel: nf_conntrack: falling back to vmalloc", it highly indicates the memory is being fragmented.

Following command can be used to compact fragmented memory pages.

```
echo 1 > /proc/sys/vm/compact_memory
```

## extfrag_threshold

This parameter affects whether the kernel will compact memory or direct
reclaim to satisfy a high-order allocation. The extfrag/extfrag_index file in
debugfs shows what the fragmentation index for each order is in each zone in
the system. Values tending towards 0 imply allocations would fail due to lack
of memory, values towards 1000 imply failures are due to fragmentation and -1
implies that the allocation will succeed as long as watermarks are met.

The kernel will not compact memory in a zone if the
fragmentation index is <= extfrag_threshold. The default value is 500.

Example output:

```shell
$  sysctl -a | grep extfrag_threshold
vm.extfrag_threshold = 500

$  cd /sys/kernel/debug/extfrag/
$  ls
extfrag_index  unusable_index

$  cat unusable_index
Node 0, zone      DMA 0.000 0.000 0.000 0.001 0.001 0.009 0.018 0.035 0.035 0.035 0.173
Node 0, zone    DMA32 0.000 0.000 0.000 0.000 0.000 0.001 0.001 0.003 0.007 0.013 0.028
Node 0, zone   Normal 0.000 0.007 0.020 0.045 0.096 0.192 0.342 0.552 0.803 1.000 1.000
Node 1, zone   Normal 0.000 0.006 0.021 0.051 0.104 0.190 0.325 0.522 0.775 1.000 1.000

$  cat extfrag_index
Node 0, zone      DMA -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000
Node 0, zone    DMA32 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000
Node 0, zone   Normal -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 0.955 0.978
Node 1, zone   Normal -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 -1.000 0.956 0.978

$  cat /proc/buddyinfo
Node 0, zone      DMA      1      0      1      0      2      1      1      0      0      1      3
Node 0, zone    DMA32      7     10     13     15     11      8      9     10      8      9    309
Node 0, zone   Normal 1134904 1124019 975094 1027980 973525 755862 528091 316182 123744      0      0
Node 1, zone   Normal 864976 1041246 994760 895916 726438 565284 415625 266573 118000      0      0
```

## References

* <https://www.kernel.org/doc/Documentation/sysctl/vm.txt>