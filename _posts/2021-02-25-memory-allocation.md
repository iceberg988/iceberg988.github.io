---
layout:     post
title:      "kmalloc and vmalloc"
subtitle:   ""
date:       2021-02-25 12:00:00
author:     "Iceberg"
catalog:    true
header-style: text
tags:
  - Performance
  - MemoryManagement
---

### kmalloc

The function allocates contiguous region in physical memory. It's fast and doesn't clear the allocated memory content.

kmalloc function is defined in [include/linux/slab.h](https://elixir.bootlin.com/linux/latest/source/include/linux/slab.h#L538)

```c
/**
 * kmalloc - allocate memory
 * @size: how many bytes of memory are required.
 * @flags: the type of memory to allocate.
 *
 * kmalloc is the normal method of allocating memory
 * for objects smaller than page size in the kernel.
 *
 * The allocated object address is aligned to at least ARCH_KMALLOC_MINALIGN
 * bytes. For @size of power of two bytes, the alignment is also guaranteed
 * to be at least to the size.
 *
 * The @flags argument may be one of the GFP flags defined at
 * include/linux/gfp.h and described at
 * :ref:`Documentation/core-api/mm-api.rst <mm-api-gfp-flags>`
 *
 * The recommended usage of the @flags is described at
 * :ref:`Documentation/core-api/memory-allocation.rst <memory_allocation>`
 *
 * Below is a brief outline of the most useful GFP flags
 *
 * %GFP_KERNEL
 *	Allocate normal kernel ram. May sleep.
 *
 * %GFP_NOWAIT
 *	Allocation will not sleep.
 *
 * %GFP_ATOMIC
 *	Allocation will not sleep.  May use emergency pools.
 *
 * %GFP_HIGHUSER
 *	Allocate memory from high memory on behalf of user.
 *
 * Also it is possible to set different flags by OR'ing
 * in one or more of the following additional @flags:
 *
 * %__GFP_HIGH
 *	This allocation has high priority and may use emergency pools.
 *
 * %__GFP_NOFAIL
 *	Indicate that this allocation is in no way allowed to fail
 *	(think twice before using).
 *
 * %__GFP_NORETRY
 *	If memory is not immediately available,
 *	then give up at once.
 *
 * %__GFP_NOWARN
 *	If allocation fails, don't issue any warnings.
 *
 * %__GFP_RETRY_MAYFAIL
 *	Try really hard to succeed the allocation but fail
 *	eventually.
 */
static __always_inline void *kmalloc(size_t size, gfp_t flags){
  	if (__builtin_constant_p(size)) {
#ifndef CONFIG_SLOB
		unsigned int index;
#endif
		if (size > KMALLOC_MAX_CACHE_SIZE)
			return kmalloc_large(size, flags);
#ifndef CONFIG_SLOB
		index = kmalloc_index(size);

		if (!index)
			return ZERO_SIZE_PTR;

		return kmem_cache_alloc_trace(
				kmalloc_caches[kmalloc_type(flags)][index],
				flags, size);
#endif
	}
	return __kmalloc(size, flags);
}
```

The first argument is the size of the blocks to be allocated. The second argument is the memory allocation type flags. It controls the behavior of kmalloc.

* GFP_KERNEL is the most commonly used flag. It means the allocation is performed on behalf of a process running in kernel space. kmalloc can put the current process to sleep while waiting for the page allocation if the system is in memory starvation. The free memory can be optained either by flushing buffer to disk or swapping out user process memory.
* GFP_ATOMIC is used if the kmalloc call is from outside process context. The kernel can use the reserved free pages to serve the allocation. The allocation can fail if there is no last free page.

### vmalloc

vmalloc allocates a contiguous memory region in the virtual address space. The allocated pages in physical memory are not consecutive and each page is retrieved with separate call to alloc_page. Thus, the vmalloc function is less efficient than kmalloc.

vmalloc function is defined in [mm/vmalloc.c](https://elixir.bootlin.com/linux/latest/source/mm/vmalloc.c#L2650)

```c
/**
 * vmalloc - allocate virtually contiguous memory
 * @size:    allocation size
 *
 * Allocate enough pages to cover @size from the page level
 * allocator and map them into contiguous kernel virtual space.
 *
 * For tight control over page level allocator and protection flags
 * use __vmalloc() instead.
 *
 * Return: pointer to the allocated memory or %NULL on error
 */
void *vmalloc(unsigned long size)
{
	return __vmalloc_node(size, 1, GFP_KERNEL, NUMA_NO_NODE,
				__builtin_return_address(0));
}
```

## Resource

* <https://www.oreilly.com/library/view/linux-device-drivers/0596005903/ch08.html>
* <https://elixir.bootlin.com/linux/latest/source/include/linux/slab.h#L538>