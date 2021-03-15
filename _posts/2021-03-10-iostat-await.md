---
layout:     post
title:      "Understanding await in iostat"
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/idaho1.jpg"
tags:
  - Observability Tools
  - Linux/Kernel
---


## What's meaning of await in iostat?

The following is the description provided for await field in iostat man page.

```shell
$ man iostat
await
    The average time (in milliseconds) for I/O requests issued to the device to be served. This includes the time spent by the requests in queue and the time spent servicing them.
```    

It is a measure of disk I/O latency in milliseconds. The latency is from the front of the I/O scheduler to the I/O completion time.

## I/O path

The I/O path mainly includes the following footprints from block layer to underneath storage device.

* Get the I/O requests from application(filesystem)
* Merge the I/O requests to existing device queue
* Dispatch the I/O requests(by the I/O scheduler) to the device driver
* Hypervisor scheduler in virtualization if any
* Multipathing if any
* Hardware handling
  * HBA driver
  * Transportation(bus)
  * FC switch routing if any
  * Storage controller queuing, caching and processing 
  * Actual disk latency

## How the await time is calculated?

The await is the average time on a per I/O basis, measured in milliseconds. It mainly includes the time spent in I/O scheduler queue and time spent on storage servicing it if the HBA/SAN latency is relatively marginal.

There are two queues involved in the I/O processing path.

* The queue in I/O scheduler
* The queue in storage side(e.g. controller)

*nr_requests* limits the maximum number of I/Os in the sorted request queue. The front thread will be blocked if the I/O can't be merged/inserted into the scheduler queue due to the full occupancy of the queue . Note that the *nr_requests* is applied to read and write separately.

After the I/O is passed to the driver, it is no longer in the scheduler queue and doesn't cout to *nr_requests* limit. However, it will count to *avgqu-sz*. So, the *avgqu-sz* could reach the sum of *nr_requests* and LUN *queue_depth*.


## How the svctm time is measured?

await measures the I/O latency on a per I/O basis while svctm take into account parallel I/O. For example, if 100 I/Os are submitted to the I/O scheduler in parallel and queued onto storage(say queue_depth=50), and the 100 I/Os completes in 10ms, the await time would be 10ms, but the svctm time could be 2ms.

## Conclusion

Since await includes the time spent in I/O scheduler and storage queue servicing. We may want to see a breakdown for the two parts by using blktrace. It would tell us the overheads on disk queue(I2D) and actual I/O service latency(D2C). For furhter study of blktrace, you can read this [article]({% post_url 2021-03-11-blktrace %}).