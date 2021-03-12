---
layout:     post
title:      "Network ring buffer"
author:     "Iceberg"
catalog:    true
header-img: "assets/images/yellowstone1.jpg"
tags:
  - Network
---

## Introduction

Receive ring buffers are shared between the device driver and NIC. The card assigns a transmit (TX) and receive (RX) ring buffer. As the name implies, the ring buffer is a circular buffer where an
overflow simply overwrites existing data. It should be noted that there are two ways to move data from the NIC to the kernel, hardware interrupts and software interrupts, also called SoftIRQs.

The RX ring buffer is used to store incoming packets until they can be processed by the device driver. The device driver drains the RX ring, typically via SoftIRQs, which puts the incoming packets into a kernel data structure called an sk_buff or “skb” to begin its journey through the kernel and up to the application which owns the relevant socket. The TX ring buffer is used to hold outgoing packets which are destined for the wire.

These ring buffers reside at the bottom of the stack and are a crucial point at which packet drop can occur, which in turn will adversely affect network performance.

You can increase the size of the Ethernet device RX ring buffer if the packet drop rate causes applications to report:

* a loss of data,
* cluster fence,
* slow performance,
* timeouts, and
* failed backups.

## Interrupts and Interrupt Handlers

Interrupts from the hardware are known as “top-half” interrupts. When a NIC receives incoming data, it copies the data into kernel buffers using DMA. The NIC notifies the kernel of this data by raising a hard interrupt. These interrupts are processed by interrupt handlers which do minimal work, as they have already interrupted another task and cannot be interrupted themselves. Hard interrupts can be expensive in terms of CPU usage, especially when holding kernel locks. The hard interrupt handler then leaves the majority of packet reception to a software interrupt, or SoftIRQ, process which can be scheduled more fairly.

Hard interrupts can be seen in /proc/interrupts where each queue has an interrupt vector in the 1st column assigned to it. These are initialized when the system boots or when the NIC device driver module is loaded. Each RX and TX queue is assigned a unique vector, which informs the interrupt handler as to which NIC/queue the interrupt is coming from. The columns represent the number of incoming interrupts as a counter value:
```
$ egrep “CPU0|eth2” /proc/interrupts
 CPU0 CPU1 CPU2 CPU3 CPU4 CPU5
 105: 141606 0 0 0 0 0 IR-PCI-MSI-edge eth2-rx-0
 106: 0 141091 0 0 0 0 IR-PCI-MSI-edge eth2-rx-1
 107: 2 0 163785 0 0 0 IR-PCI-MSI-edge eth2-rx-2
 108: 3 0 0 194370 0 0 IR-PCI-MSI-edge eth2-rx-3
 109: 0 0 0 0 0 0 IR-PCI-MSI-edge eth2-tx
```

## SoftIRQs

Also known as “bottom-half” interrupts, software interrupt requests (SoftIRQs) are kernel routines which are scheduled to run at a time when other tasks will not be interrupted. The SoftIRQ's
purpose is to drain the network adapter receive ring buffers. These routines run in the form of ksoftirqd/cpu-number processes and call driver-specific code functions. They can be seen
in process monitoring tools such as ps and top.

The following call stack, read from the bottom up, is an example of a SoftIRQ polling a Mellanox card. The functions marked [mlx4_en] are the Mellanox polling routines in the mlx4_en.ko driver kernel module, called by the kernel's generic polling routines such as net_rx_action. After moving from the driver to the kernel, the traffic being received will then move up to the socket, ready for the application to consume:
```
 mlx4_en_complete_rx_desc [mlx4_en]
 mlx4_en_process_rx_cq [mlx4_en]
 mlx4_en_poll_rx_cq [mlx4_en]
 net_rx_action
 __do_softirq
 run_ksoftirqd
 smpboot_thread_fn
 kthread
 kernel_thread_starter
 kernel_thread_starter
 1 lock held by ksoftirqd
```

SoftIRQs can be monitored as follows. Each column represents a CPU:
```
$ watch -n1 grep RX /proc/softirqs
$ watch -n1 grep TX /proc/softirqs
```

## Displaying the number of dropped packets

The ethtool utility enables administrators to query, configure, or control network driver settings.

The exhaustion of the RX ring buffer causes an increment in the counters, such as "discard" or "drop" in the output of ethtool -S interface_name. The discarded packets indicate that the available buffer is filling up faster than the kernel can process the packets.

To display drop counters for the enp1s0 interface, enter:
```
$ ethtool -S enp1s0
```

## Increasing the RX ring buffer to reduce a high packet drop rate

The ethtool utility helps to increase the RX buffer to reduce a high packet drop rate.

1. To view the maximum RX ring buffer size:
```
$ ethtool -g nic0
 Ring parameters for nic0:
 Pre-set maximums:
 RX:             4078
 RX Mini:        0
 RX Jumbo:       0
 TX:             4078
 Current hardware settings:
 RX:             2048
 RX Mini:        0
 RX Jumbo:       0
 TX:             2048
```

2. If the values in the Pre-set maximums section are higher than in the Current hardware settings section, increase RX ring buffer:

   * To temporary change the RX ring buffer of the nic0 device to 4078, enter:
```
$ ethtool -G nic0 rx 4078
```
   * To permanently change the RX ring buffer create a NetworkManager dispatcher script. For details, see the How to make NIC ethtool settings persistent (apply automatically at boot) article and create a dispatcher script.

## Understanding the maximum RX/TX ring buffer

From [ethtool](https://elixir.bootlin.com/linux/latest/source/include/linux/ethtool.h) source code, we can find the following function which is used by command "ethtool -g [nic]"
```
* @get_ringparam: Report ring sizes

```

For different NIC vender, the [driver](https://elixir.bootlin.com/linux/latest/C/ident/get_ringparam) may be implemented differently.

In this example, the NIC driver is bnx2x.
```
$ ethtool -i nic0
driver: bnx2x
```

So, we can check the [source code](https://elixir.bootlin.com/linux/latest/source/drivers/net/ethernet/broadcom/bnx2x/bnx2x_ethtool.c#L3677) as below.
```
static void bnx2x_get_ringparam(struct net_device *dev,
				struct ethtool_ringparam *ering)
{
	struct bnx2x *bp = netdev_priv(dev);

	ering->rx_max_pending = MAX_RX_AVAIL;

	/* If size isn't already set, we give an estimation of the number
	 * of buffers we'll have. We're neglecting some possible conditions
	 * [we couldn't know for certain at this point if number of queues
	 * might shrink] but the number would be correct for the likely
	 * scenario.
	 */
	if (bp->rx_ring_size)
		ering->rx_pending = bp->rx_ring_size;
	else if (BNX2X_NUM_RX_QUEUES(bp))
		ering->rx_pending = MAX_RX_AVAIL / BNX2X_NUM_RX_QUEUES(bp);
	else
		ering->rx_pending = MAX_RX_AVAIL;

	ering->tx_max_pending = IS_MF_FCOE_AFEX(bp) ? 0 : MAX_TX_AVAIL;
	ering->tx_pending = bp->tx_ring_size;
}
```

MAX_RX_AVAIL is the place to define the maximum RX ring buffer. We can further check the formula as below.
```
#define MAX_RX_AVAIL		(MAX_RX_DESC_CNT * NUM_RX_RINGS - 2)

#define NUM_RX_RINGS		8

#define MAX_RX_DESC_CNT		(RX_DESC_CNT - NEXT_PAGE_RX_DESC_CNT)
#define RX_DESC_CNT		(BCM_PAGE_SIZE / sizeof(struct eth_rx_bd))
#define NEXT_PAGE_RX_DESC_CNT	2

#define BCM_PAGE_SIZE		(1 << BCM_PAGE_SHIFT)
#define BCM_PAGE_SHIFT		12

/*
 * The eth Rx Buffer Descriptor
 */
struct eth_rx_bd {
	__le32 addr_lo;
	__le32 addr_hi;
};
```

So, based on the formula above, we can calculate the maximum RX ring buffer as below.
```
rx_max = MAX_RX_AVAIL 
       = MAX_RX_DESC_CNT * NUM_RX_RINGS - 2
       = (RX_DESC_CNT - NEXT_PAGE_RX_DESC_CNT) * NUM_RX_RINGS - 2
       = ((BCM_PAGE_SIZE / sizeof(struct eth_rx_bd)) - 2) * 8 - 2
       = (((1 << BCM_PAGE_SHIFT) / sizeof(struct eth_rx_bd)) - 2) * 8 - 2
       = ((4096 / 8 ) - 2) * 8 - 2
       = 4078
```


## Resource

* [Red Hat Enterprise Linux Network Performance Tuning Guide](/assets/docs/20150325_network_performance_tuning.pdf)
* [MONITORING AND TUNING THE RX RING BUFFER](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/monitoring-and-tuning-the-rx-ring-buffer_configuring-and-managing-networking)