---
layout:     post
title:      "Analyze the library and system calls"
date:       2021-02-18 12:00:00
author:     "Iceberg"
header-img: "assets/images/yellowstone1.jpg"
catalog:    true
tags:
  - Profiling/Tracing
---

When the application source code is not available, we can trace the function calls during its runtime in order to understand how the application code works. 

Library calls take place in user space. [System calls](https://man7.org/linux/man-pages/man2/syscalls.2.html) is the fundamental interface between application and the kernel. System calls are not invoked directly but rather via wrapper functions in glibc(or other library).

In this article, we will study how to use ltrace/strace/perf tracing tools to help understand the target application. 

Both strace and ltrace are tracing tools which use ptrace(process trace) to inspect the internal of the target process. strace only works for tracing system calls while ltrace has no such restriction. 

We use a Redhat6.2 Linux system for this study.

```shell
$ cat /etc/redhat-release
Red Hat Enterprise Linux Server release 6.2 (Santiago)

$ uname -r
2.6.32-220.el6.x86_64
```

We have ltrace, strace and perf installed in the system.

```shell
$ rpm -qa ltrace
ltrace-0.5-16.45svn.1.el6.x86_64

$ rpm -qa strace
strace-4.5.19-1.10.el6.x86_64

$ rpm -qa perf
perf-2.6.32-220.el6.x86_64
```

## perf profiler

perf is the official Linux profiler which is suited for CPU analysis. It can help analyze software functions as well.

```shell
$ pid=`pidof <process-name>`
$ perf record -p $pid -a -g -- sleep 10
$ perf report -n --stdio > perf.<process-name>.out

$ cat perf.<process-name>.out

# Events: 1K cycles
#
# Overhead  Samples    Command         Shared Object                       Symbol
# ........ ..........  .......         .................  ...........................
#

    10.17%   131       <process-name>  libc-2.12.so       [.] __memcpy
```

From the output, the memcpy function is called to the library libc-2.12.so.


## Library function calls

ltrace is a library call tracer.

```shell
$ man ltrace

ltrace(1)

NAME
       ltrace - A library call tracer

SYNOPSIS
       ltrace  [-CdfhiLrStttV]  [-a  column] [-e expr] [-l filename] [-n nr] [-o filename] [-p pid] ... [-s strsize] [-u username] [-X extern] [-x extern]
       ... [--align=column] [--debug] [--demangle] [--help] [--indent=nr] [--library=filename] [--output=filename] [--version] [command [arg ...]]

DESCRIPTION
       ltrace is a program that simply runs the specified command until it exits.  It intercepts and records the dynamic library calls which are called by
       the executed process and the signals which are received by that process.  It can also intercept and print the system calls executed by the program.

       Its use is very similar to strace(1).
```
```shell       
$ ltrace -h

Usage: ltrace [option ...] [command [arg ...]]
Trace library calls of a given program.

  -a, --align=COLUMN  align return values in a secific column.
  -c                  count time and calls, and report a summary on exit.
  -C, --demangle      decode low-level symbol names into user-level names.
  -d, --debug         print debugging info.
  -e expr             modify which events to trace.
  -f                  follow forks.
  -h, --help          display this help and exit.
  -i                  print instruction pointer at time of library call.
  -l, --library=FILE  print library calls from this library only.
  -L                  do NOT display library calls.
  -n, --indent=NR     indent output by NR spaces for each call level nesting.
  -o, --output=FILE   write the trace output to that file.
  -p PID              attach to the process with the process ID pid.
  -r                  print relative timestamps.
  -s STRLEN           specify the maximum string size to print.
  -S                  display system calls.
  -t, -tt, -ttt       print absolute timestamps.
  -T                  show the time spent inside each call.
  -u USERNAME         run command with the userid, groupid of username.
  -V, --version       output version information and exit.
  -x NAME             treat the global NAME like a library subroutine.

Report bugs to ltrace-devel@lists.alioth.debian.org
```

We use the following commands to trace the target process on the specified library calls. The "-c" option is used to get the summary report.

```shell
$ ltrace -p $pid --library=/lib/libc-2.12.so -c -o ltrace.<process-name>.c.out

$ cat ltrace.<process-name>.c.out

% time     seconds  usecs/call     calls      function
=========================================================
 63.69    8.028116        2386      3364 write
 18.63    2.348951         190     12323 memcpy
 16.37    2.063338         613      3365 time
  0.62    0.078129          34      2243 access
  0.36    0.044836          19      2242 strncpy
  0.33    0.041813          18      2243 __errno_location
=========================================================
100.00   12.605183                 25780 total
```

From above tracing output, the process spent 63.69% time in write function call and 18.63% time in memcpy function call.

To know more detail for the function calls, we remove the "-c" option and trace again.

```shell
$ ltrace -p $pid --library=/lib/libc-2.12.so -o ltrace.<process-name>.out

$ cat ltrace.<process-name>.out

25236 access("", 0)                                = -1
25236 time(0x7fff8b770d58)                         = 1613691764
25236 memcpy(0x7fc462d54000, "\323j\377\305\207\037\332\234\304CH\317\270\005\036\250\271\304w"`\262\214\322\213\357\\\356\213,\300\357"..., 80384) = 0x7fc462d54000
25236 memcpy(0x7fc462d67a00, "\204\257\265P\261\241\340\232\0278\2132\272\350)\327\025Iwc\342\014#\353{\265d2|\240V\266"..., 131072) = 0x7fc462d67a00
25236 memcpy(0x7fc462d87a00, "\204\257\265Q\261\241\340\233\0278\2133\272\350)\330\025Iwd\342\014#\354{\265d3|\240V\267"..., 131072) = 0x7fc462d87a00
25236 memcpy(0x7fc462da7a00, "\204\257\265R\261\241\340\234\0278\2134\272\350)\331\025Iwe\342\014#\355{\265d4|\240V\270"..., 131072) = 0x7fc462da7a00
25236 memcpy(0x7fc462dc7a00, "\204\257\265S\261\241\340\235\0278\2135\272\350)\332\025Iwf\342\014#\356{\265d5|\240V\271"..., 50688) = 0x7fc462dc7a00
25236 __errno_location()                           = 0x7fc4631416a8
25236 write(1, "\323j\377\305\207\037\332\234\304CH\317\270\005\036\250\271\304w"`\262\214\322\213\357\\\356\213,\300\357"..., 524288) = 524288
```

From above tracing output, there are five memcpy function calls to totally copy 524288 bytes in memory, and followed by a write function call to write the same amount of data to a file of file descriptor 1.

We can read the memcpy function call definition from the library manual page.

```shell
$ man 3 memcpy

MEMCPY(3)                  Linux Programmer’s Manual                 MEMCPY(3)

NAME
       memcpy - copy memory area

SYNOPSIS
       #include <string.h>

       void *memcpy(void *dest, const void *src, size_t n);

DESCRIPTION
       The  memcpy() function copies n bytes from memory area src to memory area dest.  The memory areas should not overlap.  Use memmove(3) if the memory
       areas do overlap.

RETURN VALUE
       The memcpy() function returns a pointer to dest.
```

## System calls

ltrace provides a "-S" option to display the system calls.

```shell
$ ltrace -p $pid --library=/lib/libc-2.12.so -S -c -o ltrace.<process-name>.l.S.c.out

$ cat ltrace.<process-name>.l.S.c.out
% time     seconds  usecs/call     calls      function
=========================================================
 41.04   10.231204        3444      2970 write
 40.78   10.166975        3420      2972 SYS_write
  9.25    2.306825         212     10877 memcpy
  8.33    2.076351        1048      1980 access
  0.22    0.055967          18      2971 time
  0.16    0.039493          19      1980 strncpy
  0.15    0.037153          18      1980 __errno_location
  0.07    0.016229           8      1980 SYS_access
  0.00    0.000037          37         1 _IO_putc
=========================================================
100.00   24.930234                 27711 total
```

We can also use strace to do the similar trace to system calls.

```shell
$ strace -p $pid -o > strace.<process-name>.c.out

$ cat strace.<process-name>.c.out

% time     seconds  usecs/call     calls   errors syscall
=========================================================
 99.85    0.039798          11      3533           write
  0.15    0.000060           0      2356      2356 access
=========================================================
100.00    0.039858                  5889      2356 total
```

The function call details can be traced by removing the "-c" option.

```shell
$ strace -p $pid -o > strace.<process-name>.out

$ cat strace.<process-name>.out

write(1, "\4\363\263\370EJ\n\336W,\270a\275\221;\7`\261\364\24\354_t7\6#\354\212}\v\325\230"..., 524288) = 524288

```

Similar to the output from ltrace, it writes 524288 bytes data from the buffer to a file of file descriptor 1.

We can read the manual page to understand the parameters and return value for the write system call.

```shell
$ man 2 write

NAME
       write - write to a file descriptor

SYNOPSIS
       #include <unistd.h>

       ssize_t write(int fd, const void *buf, size_t count);

DESCRIPTION
       write() writes up to count bytes from the buffer pointed buf to the file referred to by the file descriptor fd.

RETURN VALUE
       On success, the number of bytes written is returned (zero indicates nothing was written).  On error, -1 is returned, and  errno  is  set  appropriately.
```

If we want to know which file is actually written, we can get the file descriptors from /proc/$pid/fd

```shell
$ ls -la /proc/$pid/fd > proc.fd.out
total 0
dr-x------ 2 root root  0 Feb 18 15:49 .
dr-xr-xr-x 7 root root  0 Feb 18 15:39 ..
lrwx------ 1 root root 64 Feb 18 15:49 0 -> socket:[39812145]
lrwx------ 1 root root 64 Feb 18 15:49 1 -> socket:[39812145]
lrwx------ 1 root root 64 Feb 18 15:49 2 -> socket:[39812186]
```

Now, we understand it actually writes to a socket but rather a real file. This makes sense because the application is a data generator which produce data in memory and send to remote server through network.

## Source

* <https://en.wikipedia.org/wiki/Ptrace>
* <https://man7.org/linux/man-pages/man2/syscalls.2.html>
* <https://developers.redhat.com/blog/2014/07/10/ltrace-for-rhel-6-and-7>
* <https://www.tutorialspoint.com/User-Mode-vs-Kernel-Mode>
* <https://code.woboq.org/gcc/libgcc/memcpy.c.html>
* <https://code.woboq.org/userspace/glibc/sysdeps/unix/sysv/linux/write.c.html#__libc_write>