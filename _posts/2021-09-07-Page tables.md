---
layout: post
title: Page table
---

使用页表的好处在于实现进程之间的隔离
隔离的好处在于防止因意外或者恶意导致的进程终止影响到其他的进程甚至内核


地址空间 address space
地址空间的目的在于给每个进程包括内核kernel一个独立的内存（逻辑上独占一块内存）

虚地址空间可能比实地址空间更大也可能更小

虚地址从逻辑上实现了（实地址）内存的复用
多个进程共用一块内存（同样进程复用的还有处理器）
最常用的内存复用的方法就是页表

页表由硬件实现，即Memory management unit（MMU）
如果mmu功能开启的话，CPU通过MMU将虚地址转换为实地址
MMU控制页表

页表同样存储在内存中
同时用一个寄存器存储着页表在内存中的地址
（A register stores where the page table stores）

Every application/process has its own map and that map defines its address space.
在RISC-V中寄存器satp存储着页表在内存中的地址
当CPU或者说操作系统进行上下文切换（context switch）的时候 
satp寄存器中的值也将被替换为新的进程的页表的地址

RISC-V指令集的页表大小为4kB（4096 bytes）
寻址所用到的位数为39位（实际使用的是38位，对应256GB大小的内存）

在RISC-V中，一个虚地址（64位）由三部分组成，前25位为保留位
实际使用的是后39位，其中前27位为index，用于索引physical page number
后12位是offset
即实际用于索引页表的是index，index可以索引到某一张具体的页表
offset根据被索引的页表来确定实地址中的具体位置

A page table is stored in physical memory as a three-level tree.
Question：
"Is it just me? why use 3 level page table? I still didn't see the point."
"I think it's because when using a single page table, 
you always have to allocate memory for the entire table of size 2^27,
but the 3 level organization allows you to create a few small page directories if the process need a little memory."
页表索引分级的一个可能原因是提高效率，如果一个进程不需要很多的内存
那么完全可以只建立小的索引用以提高索引的效率

指令集为64位的意思是其地址用64位表示：
内存中的地址最大可以是2的64次方，同时一个寄存器（存放地址）是64位。
RISC-V 指令集有64个寄存器，32个整数寄存器，32个浮点数寄存器。

在RISV-V中，一个实地址的大小位56位
56位这个大小是由硬件设计者决定的（主板原则上能够支持56位大小的物理内存 ）
这意味着实际寻址只要用到56根地址线而不是64根
即实地址的大小远远大于虚地址
 
A RISV-V page table is logically an array of 2^27（134,217,728）page table entries(PTEs)
页表就是一个PTE的数组
每一个PTE包含一个44位的phisical page number（PPN）和some flags.
一个实地址就是一个PPN（前44位）+来自虚地址中的offset（12位）共56位

A page table is stored in physical memory as a three-level tree.
页表作为一个三级树结构存储在物理内存中
This three-level structure allows a page table to omit entire page table pages in the common case in which large ranges of virtual addresses have no mappings.
（通常情况下大部分的虚拟地址都没有相应的映射，分级提高了地址转换的效率）

Each PTE contains flag bits that tell the paging hardware how the associated virtual address is allowed to be used.
每条PTE包含一些标志位（9位），用于指示MMU如何使用相应的虚拟地址.

To tell the hardware to use a page table, the kernel must write the physical address of the root page-table page into the satp register
硬件如何识别/找到页表？
根页表页，存储在satp这个寄存器中，每个CPU都有一个satp寄存器

Each CPU has its own satp so that different CPUs can run different processes, each with a private address space described by its own page table.
每个CPU都有自己的satp寄存器，因此每个CPU都可以（同时而不是分时）运行一个独立的进程，
这样的每个进程都有一个独立的地址空间（地址空间由页表决定）。

物理地址：storages cells in DRAM
A few notes about terms. Physical memory refers to storage cells in DRAM. A byte of physical
memory has an address, called a physical address. Instructions use only virtual addresses,
which the paging hardware translates to physical addresses, and then sends to the DRAM hardware to read or write storage.
Virtual memory refers to the collection of abstractions and mechanisms the kernel provides to manage physical memory and virtual addresses.


Kernel address space
The kernel uses an identity mapping for most virtual addresses; that is, most of the kernel’s
address space is “direct-mapped.” For example, the kernel itself is located at KERNBASE in both the virtual address space and in physical memory.
内核地址是直接直接对应（映射）的

There are a couple of virtual addresses that aren’t direct-mapped:
1 The trampoline page. It is mapped at the top of the virtual address space; user page tables
have this same mapping. A physical page (holding the trampoline code) is mapped twice in the virtual address space of the kernel: once at top of the virtual address space and once in the kernel text.
2 The kernel stack page. Each process has its own kernel stack,which is mapped high so
that below it xv6 can leave an unmapped guard page. The guard page’s PTE is invalid (i.e.,
PTE_V is not set), which ensures that if the kernel overflows a kernel stack, it will likely
cause a fault and the kernel will panic.Without a guard page an overflowing stack would
overwrite other kernel memory, resulting in incorrect operation. A panic crash is preferable.

panic是可接受的，而避免了因为stack overflow导致的内核崩溃。

guard page:用于防止栈内存溢出（overflow）;并且在内存中没有实地址映射（mapping invalid）
The kernel maps the pages for the trampoline and the kernel text with the permissions PTE_R
and PTE_X . The kernel reads and executes instructions from these pages. The kernel maps the other pages with the permissions PTE_R and PTE_W , so that it can read and write the memory in those pages. The mappings for the guard pages are invalid.

kernel virtual address space:
KERNBASE(0x80000000),Kernel text,Kernel data,Free memory,
PHYSTOP(0x86400000),...Kstack1,Guard page,Kstack0,Guard page，Trampoline


Code:creating an address space
Most of the xv6 code for manipulating address spaces and page tables resides in vm.c . The central data structure is pagetable_t, which is really a pointer to a RISC-V root page-table page; a pagetable_t may be either the kernel page table, or one of the per-process page tables. The central functions are walk, which finds the PTE for a virtual address,
and mappages, which installs PTEs for new mappings. Functions starting with kvm manipulate
the kernel page table; functions starting with uvm manipulate a user page table; other functions are used for both. copyout and copyin copy data to and from user virtual addresses provided as system call arguments; they are in vm.c because they need to explicitly translate those addresses in order to find the corresponding physical memory.

pagetable_t 是一个指向root page-table page的指针，可能是内核页表也可能是每个进程的页表


Each RISC-V core caches page table entries in a Translation Look-aside Buffer (TLB), and
when xv6 changes a page table, it must tell the CPU to invalidate corresponding cached TLB
entries. If it didn’t, then at some point later the TLB might use an old cached mapping, pointing to a physical page that in the meantime has been allocated to another process, and as a result, a process might be able to scribble on some other process’s memory. The RISC-V has an instruction sfence.vma that flushes the current core’s TLB. xv6 executes sfence.vma in kvminithart after reloading the satp register, and in the trampoline code that switches to a user page table before returning to user space
每个处理器都将PTE缓存在TLB中，当xv6更换一个页表的时候，它必须告知处理器刷新相应的TLB缓存。
RISC-V使用sfence.vma指令来刷新当前处理器的TLB.
xv6执行sfence.vma in kvminithart，当重新装载完satp寄存器之后。(即更换页表之后立即刷新TLB)

Phisical memory allocation
The kernel must allocate and free physical memory at run-time for page tables, user memory,
kernel stacks, and pipe buffers.
xv6 uses the physical memory between the end of the kernel and PHYSTOP for run-time alloca-
tion. It allocates and frees whole 4096-byte pages at a time. It keeps track of which pages are free by threading a linked list through the pages themselves. Allocation consists of removing a page from the linked list; freeing consists of adding the freed page to the list.


Process address space
Each process has a separate page table, and when xv6 switches between processes, it also changes page tables.
A process’s user memory starts at virtual address zero and can grow up to MAXVA, allowing a process to address in principle 256 Gigabytes of memory.

When a process asks xv6 for more user memory, xv6 first uses kalloc to allocate physical
pages. It then adds PTEs to the process’s page table that point to the new physical pages. Xv6 sets the PTE_W , PTE_X , PTE_R , PTE_U , and PTE_V flags in these PTEs. Most processes do not use the entire user address space; xv6 leaves PTE_V clear in unused PTEs.

分配内存意味着什么？
首先分配物理内存，然后将PTEs添加到相应进程的页表中。（PTEs设置相应的flags）
















