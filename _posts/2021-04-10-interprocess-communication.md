---
layout:     post
title:      "Interprocess communication(IPC)"
subtitle:   ""
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/yellowstone3.jpg"
tags:
  - Linux/Kernel
---

# Pipes

Pipes are the oldest form of UNIX System IPC and are provided by all UNIX systems. Pipes have two limitations.

1.  Historically, they have been half duplex (i.e., data flows in only one direction). Some systems now provide full-duplex pipes, but for maximum portability, we should never assume that this is the case.
2.  Pipes can be used only between processes that have a common ancestor. Normally, a pipe is created by a process, that process calls fork, and the pipe is used between the parent and the child.

FIFOs get around the second limitation, and UNIX domain sockets get around both limitations.

# FIFOs

FIFOs are sometimes called named pipes. Unnamed pipes can be used only between related processes when a common ancestor has created the pipe. With FIFOs, however, unrelated processes can exchange data.

Creating a FIFO is similar to creating a file. Indeed, the pathname for a FIFO exists in the file system.

# Message Queues

A message queue is a linked list of messages stored within the kernel and identified by a message queue identifier. We’ll call the message queue just a queue and its identifier a queue ID.

A new queue is created or an existing queue opened by msgget. New messages are added to the end of a queue by msgsnd. Every message has a positive long integer type field, a non-negative length, and the actual data bytes (corresponding to the length), all of which are specified to msgsnd when the message is added to a queue. Messages are fetched from a queue by msgrcv. We don’t have to fetch the messages in a first-in, first-out order. Instead, we can fetch messages based on their type field.

# Semaphores

A semaphore isn’t a form of IPC similar to the others that we’ve described (pipes, FIFOs, and message queues). A semaphore is a counter used to provide access to a shared data object for multiple processes.

To obtain a shared resource, a process needs to do the following:

1. Test the semaphore that controls the resource.
2. If the value of the semaphore is positive, the process can use the resource. In this case, the process decrements the semaphore value by 1, indicating that it has used one unit of the resource.
3. Otherwise, if the value of the semaphore is 0, the process goes to sleep until the semaphore value is greater than 0. When the process wakes up, it returns to step 1.

When a process is done with a shared resource that is controlled by a semaphore, the semaphore value is incremented by 1. If any other processes are asleep, waiting for the semaphore, they are awakened.

To implement semaphores correctly, the test of a semaphore’s value and the decrementing of this value must be an atomic operation. For this reason, semaphores are normally implemented inside the kernel.

# Shared Memory

Shared memory allows two or more processes to share a given region of memory. This is the fastest form of IPC, because the data does not need to be copied between the client and the server. The only trick in using shared memory is synchronizing access to a given region among multiple processes. If the server is placing data into a shared memory region, the client shouldn’t try to access the data until the server is done. Often, semaphores are used to synchronize shared memory access. But record locking or mutexes can also be used.

# Network IPC: Sockets

The socket network IPC interface can be used by processes to communicate with other processes, regardless of where they are running, on the same machine or on different machines. Indeed, this was one of the design goals of the socket interface. The same interfaces can be used for both intermachine communication and intramachine communication.

A socket is an abstraction of a communication endpoint. Just as they would use file descriptors to access files, applications use socket descriptors to access sockets. Socket descriptors are implemented as file descriptors in the UNIX System. Indeed, many of the functions that deal with file descriptors, such as read and write, will work with a socket descriptor.

Normally, the recv functions will block when no data is immediately available. Similarly, the send functions will block when there is not enough room in the socket’s output queue to send the message. This behavior changes when the socket is in nonblocking mode.

# Reference

* Advanced Programming in the UNIX Environment