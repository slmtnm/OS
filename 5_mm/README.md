### Memory management

## Goals of mm

*Goal*: manage physical memory (DRAM) on behalf of one or more executing processes.

We decouple virtual and physical addresses. Virtual address space may be much larger
than actual physical memory, because:

* not all virtual addresses necesserily are mapped to physical memory (only those that used)
* some portion of physical memory swapped on disk

Two main goals:

* Allocate -- how physical memory is used and what is free among physical memory
* Arbitrate -- OS should be quickly able to preform virtual address translation and validation

Virtual address spaces subdivided into **pages** of fixed size. Physical memory
divided into **page frames** of the same size as pages. Role of allocation is 
to map *page* to corresponding *page frame*. Arbitration is done via *page tables*.

Paging is not the only way to decouple virtual and physical memories, there are
also *segment-based* memory management. It doesn't use fixed sized pages, but
flexibly sized segments that can be mapped to some regions in physical memory. Arbitration
uses **segment registers** to validate access.

## Hardware support

Every CPU unit has built-in Memory Management Unit (MMU), that:

* translates virtual to physical addresses
* reports faults: illegal access, permission

There are also special registers, that

* is pointers to page table (in page-based mm)
* has values of base and limit size, number of registers, etc. (in segmend-based mm)

Typcally MMU integrates a small cache of valid VA-PA translations -- **Translation Lookaside buffer**
(TLB).

Actual PA generation done in hardware, however OS should give it page tables to perform translation,
and allocation, replacement algorithms implemented in software.

## Page tables

Page-based mm is most popular mm type.
Page table -- collection of page entries, where every page entry
maps only first address of every virtual page to first address of 
corresponding page frame.

So, every virtual address consists of 2 parts: 

* Virtual page number: number of page in which this address lays
* Offset in that page

VPN (virtual page number) used to produce physical frame number (PFN). 
Actual physical address is PFN + Offset.

Important to note, that mapping from VPN to PFN created only 
when some address in that virtual page is first accessed.

Unused pages are reclaimed.

Page entries have not only VA->PA mapping, but also some bits,
that signals validity of corresponding PA (is it exists in DRAM, or it swapped on disk, etc.).
If hardware detected that the mapping is invalid, it will fault and pass control to OS. On fault,
there will be a mapping that will reastablish between valid virtual address and a valid location
in physical memory.

Page tables are *per process*. On context switch, OS switches to valid page table of process
(for ex. on x86 platform, OS will update register CR3, that points to currently active page table).

## Flags in page table entry

There more than 1 flag in page table entry:

* Present (valid/invalid) -- whether the contents of virtual memory are actually present in physical memory
* Dirty -- get set whenever a page is written to
* Accessed (for read or write) -- wheter the page has been accessed for read/write in some period of time.
* Protection bits -- whether page may be only read, only write or both (0 for read only, 1 for read + write).
* U/S bit -- whether page may be accessed only from supervisor (privileged) mode only, or in usermod too
* Others: caching related info (write through, caching disabled, etc...)
* Unused for future

MMU use page table entries not just to perform address translation, but relies on these bits
to establish validity of the access. If the HW determines that a physical memory access 
cannot be performed, it causes a page fault: CPU will place error code on kernel stack,
and then generate a trap into kernel. That will invoke a page fault handler, that
determuines action based on error code and faulting address. For example, it may be caused
because of page frame not in memory and it is just needed to be brought from disk to memory,
or it may be protection error (OS will generate signal SIGSEGV to process, that tries access that memory). Error code is generated from page table entry flags, and faulting address is stored in CR2 
register.
