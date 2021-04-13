---
layout:     post
title:      "Using fio to generate millions of files"
author:     "Iceberg"
header-img: "assets/images/header/yellowstone2.jpg"
catalog:    true
tags:
  - Simulation
  - Performance  
---

[fio](https://fio.readthedocs.io/en/latest/fio_doc.html#) is the acronym for Flexible IO Tester and is a tool for I/O performance measurement. 

From time to time, I use fio to run performance benchmark test, either to investigate I/O performance for certain storage or reproduce performance issues.

In this article, we are not target to discuss how to use fio for general I/O performance test. Instead, we will explore how to use it to create millions of files in the file system and simulate the real-world data content.

fio provides two options, dedupe_percentage and buffer_compress_percentage, from which the percentage of identical buffers and percentage of compressible buffer content can be specified. To meet specific file content requirement, a buffer_pattern option can be used to reflect the real-world data content.

```shell
buffer_compress_percentage=int

If this is set, then fio will attempt to provide I/O buffer content (on WRITEs) that compresses to the specified level. Fio does this by providing a mix of random data followed by fixed pattern data. The fixed pattern is either zeros, or the pattern specified by buffer_pattern. If the buffer_pattern option is used, it might skew the compression ratio slightly. Setting buffer_compress_percentage to a value other than 100 will also enable refill_buffers in order to reduce the likelihood that adjacent blocks are so similar that they over compress when seen together. See buffer_compress_chunk for how to set a finer or coarser granularity for the random/fixed data region. Defaults to unset i.e., buffer data will not adhere to any compression level.

buffer_pattern=str

If set, fio will fill the I/O buffers with this pattern or with the contents of a file. If not set, the contents of I/O buffers are defined by the other options related to buffer contents. The setting can be any pattern of bytes, and can be prefixed with 0x for hex values. It may also be a string, where the string must then be wrapped with "". Or it may also be a filename, where the filename must be wrapped with '' in which case the file is opened and read. Note that not all the file contents will be read if that would cause the buffers to overflow. 

So, for example: buffer_pattern="0123456789"

dedupe_percentage=int

If set, fio will generate this percentage of identical buffers when writing. These buffers will be naturally dedupable. The contents of the buffers depend on what other buffer compression settings have been set. It’s possible to have the individual buffers either fully compressible, or not at all – this option only controls the distribution of unique buffers. Setting this option will also enable refill_buffers to prevent every buffer being identical.
```

The following example shows how we use fio to simulate real-world data.

```shell
$ fio -v
fio-3.7

$ cat fio_job_file.ini
[job1]
ioengine=libaio
iodepth=8
rw=write
direct=1
nrfiles=1000
filesize=102400
blocksize=4096
dedupe_percentage=30
buffer_compress_percentage=50
buffer_pattern="0123456789"
numjobs=8
directory=/testdir
```

The following example shows how the generated file content looks like. It contains integer numbers which is defined by buffer_pattern. With the real-world like data generated, we could simulate the workloads and deliver more realistic performance result.

```shell
$ ls -la /testdir/job1.0.0
-rw-r--r--. 1 root root 102400 Feb 20 19:10 /testdir/job1.0.0

$  strings /testdir/job1.0.0 | head
-(=,(
0123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345
2>}o
l&'TI
0123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345
	|6=
&<y|
FSlK
t+pf|
0123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345
```