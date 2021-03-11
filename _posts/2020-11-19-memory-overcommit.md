---
layout:     post
title:      "Memory overcommit"
subtitle:   " \"\""
date:       2020-11-19 12:00:00
author:     "Iceberg"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Performance 
    - MemoryManagement
---

The Linux kernel supports the following overcommit handling modes

0	-	Heuristic overcommit handling. Obvious overcommits of
		address space are refused. Used for a typical system. It
		ensures a seriously wild allocation fails while allowing
		overcommit to reduce swap usage.  root is allowed to 
		allocate slightly more memory in this mode. This is the 
		default.

1	-	Always overcommit. Appropriate for some scientific
		applications. Classic example is code using sparse arrays
		and just relying on the virtual memory consisting almost
		entirely of zero pages.

2	-	Don't overcommit. The total address space commit
		for the system is not permitted to exceed swap + a
		configurable amount (default is 50%) of physical RAM.
		Depending on the amount you use, in most situations
		this means a process will not be killed while accessing
		pages but will receive errors on memory allocation as
		appropriate.

		Useful for applications that want to guarantee their
		memory allocations will be available in the future
		without having to initialize every page.

The overcommit policy is set via the sysctl `vm.overcommit_memory'.

The overcommit amount can be set via `vm.overcommit_ratio' (percentage)
or `vm.overcommit_kbytes' (absolute value).

The current overcommit limit and amount committed are viewable in
/proc/meminfo as CommitLimit and Committed_AS respectively.