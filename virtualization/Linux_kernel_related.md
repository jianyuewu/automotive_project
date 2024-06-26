Linux kernel is using monolithic kernel, while in automotive, it is mainly using micro kernels.  
# Scheduling  
Scheduler's main task is time-sharing among tasks, need perform context switch.  
Old kernel has O(1), BFS scheduler, now mainly using CFS (also O(1) time complexity).  
Kernel is using task_struct for scheduling, for both process and thread, process ID is in tgid, and thread ID is in pid field. real_parent field points to original parent, while parent field can point to debugger process.  

## Scheduling policy  
We can set scheduling policy via sched_setscheduler().  
Mostly used policies are:  
SCHED_NORMAL: Time equal (vruntime in CFS). Nice value is used for configure the priority and vruntime (10% for each nice level).  
    Here CFS is using red-black tree to store tasks, left most node is smallest vruntime task, priority is 100-139 (nice -20 to +19).  
    vruntime = delta_exec ∗ weight_nice_0 / weight.  
SCHED_FIFO: First In First Out.  
SCHED_RR: RoundRobin.   
SCHED_FIFO/RR is using array and linked-list to store, priority is 0-99.
SCHED_IDLE: It is actually a CFS task, weighted as 3.  

## Scheduling classes  
It is defined by sched_class struct.  
There are 5 sched classes, each sched class's next pointer will store next one. Highest priority is stop class (take cpu offline).  
stop_sched_class->dl_sched_class->rt_sched_class->fair_sched_class->idle_sched_class.  
pick_next_task() will select a new task_struct to run, by scheduling classes and policies.  

## Priority inversion and inheritance  
Priority inversion:  
If tasks A and B share same resource, protected by mutex, and A priority is higher than B.  
If B is holding the lock, then A need wait. If there is another task priority higher than B, then A also need wait.  
i.e. if B switched out by ksoftirq, then both A and B need wait.  
Priority inheritance:  
Task B will inherit the highest priority of A, when it acquires the lock.  

## Synchronization methods  
Mutexes: mutex_lock/_lock_interruptible/_lock_killable/_trylock() and mutex_unlock(). mutex can sleep. Wait_lock is using spin_lock. Note mutex must be released by original owner, and can't be applied recursively, and can't be used in interrupt context.   
Semaphore: up() and down(). down() can sleep. Also have interruptible/killable/trylock versions.  
seqlock(): Used when value rarely changed.  
RCU: Read >> Write. Resources need access via pointers. Write will make a copy of the relevant data structure. rcu_read_lock(), rcu_read_unlock().  
Atomic: ATOMIC_INIT(i), atomic_read(v), atomic_set(v, i), atomic_add/_sub/_inc/_dec().  
Spinlock: spin_lock_init(&lck); spin_lock(&lck); spin_unlock(&lck), spin_is_locked(), spin_trylock(). Note that in linux preempt RT patch, spinlock is also preemptable.  
Preemption: preempt_disable(); preempt_enable();  

## Affinity  
Process affinity:  
sched_getaffinity() and sched_setaffinity().  
Or use:
```bash
# set  
taskset -p $affinity $pid  
# show
taskset -p $pid  
```
Interrupt affinity:  
irq_set_affinity_hint(irq, affinity).  

## Limit & Capabilities
Process resources can be controlled via getrlimit() and setrlimit().  
Capabilities can be checked via capable(), i.e. capable(CAPS_SYS_ADMIN).  
We can use capget/capset() functions to handle the capabilites.  

## kthread usage
kthread_create() init process in sleeping state.  
kthread_bind() bind process to a physical core.  
kthread_run(), run with threadfn(data), and namefmt is name.  
kthread_stop(), stop the thread.  
We can also per threads via for_each_online_cpu(cpu).  

## Cgroup
Currently need use cgroupv2, instead of cgroupv1, because systemd is stop supporting cgroupv1.
We can create different cgroups to control resources like cpuset, cpu, memory.  
In cgroupv1, the default CPU share value is 1024, while in cgroupv2, the default CPU weight (same meaning as share) is 100, with the minimum share/weight being 1. These parameters are controlled via sched_min_granularity_ns and sched_wakeup_granularity_ns.  

## New scheduler EEVDF  
Kernel's sched_ext will have new algorithms.  
EEVDF (Earliest Eligible Virtual Deadline First): A task is considered "eligible" when it is ready to execute without being blocked by dependencies such as I/O operations. The priority of tasks is dynamically calculated based on their virtual deadlines. This means that the priority can shift based on the urgency of tasks and the state of the system.  
Mainly for solving QoS related, some tasks need low latency.  
Solving by deadline-driven, focues on meeting the deadlines of tasks.  
Add slice for calculating deadline, and pick_eevdf() for select schedule entity.  

## sched_ext  
SCHED_EXT: Another way, which can use BPF to have scheduler plugins, more flexible. Between SCHED_NORMAL and SCHED_IDLE.  

## Debugging scheduling latency  
If we are seeing scheduling latency, usually we can check schedstat, top, ftrace, and /proc/interrupts.  
```bash
# schedstat need CONFIG_SSCHEDSTATS=y, implementation can check show_schedstat().  
echo 1 > /proc/sys/kernel/sched_schedstats  
# Whole system  
cat /proc/schedstat  
# pid detailed sched stats  
cat /proc/<pid>/schedstat  
cat /proc/<pid>/sched  
```
For top, we can press 1 to see the all the cpu's usage, espeically check interrupt (hi, si) and iowait (wa) usage. 
For ftrace, we can use function/function graph/event/kprobe/uprobe traces with tracefs, or C code, like kprobe has pre-/post-/fault-/break-handler, most common usage is function/event trace, only filter with some suspicious functions to see if they are called. Another is kprobe, like add in pre- handler, like a callback function for specific usage.  
/proc/interrupts can also checked to see the interrupt status, we can also check spurious interrupt.  
We can check nr_switches, nr_voluntary_switches and nr_involuntary_switches.  
Also other tools like strace, latencytop, powertop are good for specific scenarios.  
WARN_ONCE(condition, format) and BUG(), BUG_ON(condition) and panic() can also be used for debug.  
We can also use gdb(kgdb is also OK)/crash tools, and create debugfs/procfs if needed, procfs is using seq_file APIs.  
We can use perf tool to see the page fault count, minor/major.  
Sometimes maybe system has RCU stall, or dead lock issues, we can check from output logs accordingly.  

# Memory management  
## Basics  
1. Page & PageTable:  
Mostly we are using 4-level page table (pgd->pud->pmd->pte), or 5-level page table (extra p4d between pgd and pud).  
CPU connects to MMU. MMU has 4kB TLB, large TLB, and TWU (Table Walk Unit), which will traverse page table to find the addr.  
Hierarchic page table will split virtual addr space to 1G table, then 1G table to 2M table, and to 4K table, then offset points to PTE.  
Page can be translated to pfn via page_to_pfn(page), and vice versa via pfn_to_page (pfn).  
Page flags: Present, Accessed, Dirty, Read/Write, User/Superuser, PCD (Used by HW cache), PWD (write back), Page Size.  
2. Zone:  
In 64-bit system, there are ZONE_DMA (24bit ISA DMA, 0-16MB), ZONE_DMA32 (16MB-4GB), ZONE_DEVICE, ZONE_NORMAL (16MB-Highest) and ZONE_MOVABLE.  
In 32-bit system, there is also ZONE_HIGHMEM (896MB-Highest).  
3. NUMA:  
NUMA has distance between nodes. Automatic NUMA balancing moves tasks (which can be threads or processes) closer to the memory they are accessing. It also moves application data to memory closer to the tasks that reference it.  
4. Memory Model:  
Currently we have 3 kinds of models, can get page via:  
    a. flat mem: mem_map[PFN]  
    b. discontig mem：node_mem_map[PFN-OFFSET]  
    c. sparse mem: PFN, vmemmap -> section_mem_map[]  
5. Types of addresses:  
User virtual addr, physical addr, bus addr, kernel logic addr (same as kernel virt addr in 64 bit system).  
0 -> PAGE_OFFSET is user space, PAGE_OFFSET -> 4GB is kernel space.  

## Allocators  
1. Buddy Allocator:  
It can allocate continuous VA and PA area, min block size is 4kB (order 0), max block size is 4MB (order 11).  
Buddy has free_area[] array in struct zone, points to page linked lists.  
There is also per_cpu_pages linked list, has hot pages and cold pages for order 0, will use hot pages firstly.  
APIs: alloc_page(gfp_mask)/alloc_pages(gfp_mask, order)/free_page(addr)/free_pages(addr, order).  
Check usage via: cat /proc/buddyinfo  
2. Vmalloc Allocator:  
It can allocate continuous VA and dis-continuous PA area, no max block size restriction (only restrict by avail va and pa).  
VA is using struct vmap_area to manage, from free_vmap_area_root.  
PA is pages from Buddy Allocator.  
PageTable: can find vmap_area, then find vm_struct, and pages.  
APIs: vmalloc(sz)/vfree(addr).  
Check usage via: cat /proc/vmallocinfo  
3. Slab/ Slub/ Slob Allocator:  
It is pre-allocated for specific size, or specific structures, so in runtime, it is very fast to use.  
Pages also from Buddy allocator, use struct slab and struct kmem_cache to maintain.  
APIs: kmalloc(size, gfp_mask)/kfree(addr)/kmem_cache_create(). There are also mempool_create/_alloc/_free() APIs.  
Check usage via: cat /proc/slabinfo   

## Mappings  
1. Linear mapping:  
It can use ZONE_DMA, ZONE_NORMAL.  
Start from __START_KERNEL_map, and can use APIs __pa() and __va() to quick calc PA and VA.  
2. Temporary mapping:  
Those memory will create temporary mapping when needed.  
RSVDMEM: memmap(), va from vmalloc space.  
MMIO: ioremap(), va from vmalloc space.  
ZONE_HIGHMEM: kmap(), va from fixmap space, it is PFN mapped mem.  
3. Fix mapping:  
APIC, KMAP, PCIe MCFG, IO-APIC will use fix mapping.  
For permanent mapping, same VA will be used during kernel startup and runtime usages.  
4. File mapping:  
In kernel mode, there is page cache, and user mode will create pagetable, mapping va to page cache.  
Without this mapping, we will need use copy_to_user and copy_from_user APIs to transfer data.  
copy_from_user is copy from user process's kernel space to user space.  
The copy_from_user() function is necessary because:  
    a. The buffer pointed to by the pointer may be freed by user space, and kernel space does not know about it. Moreover, other processes in kernel space do not know the specific PA that this VA points to.  
    b. It also checks whether the address is truly from user space; if it's from kernel space, then no copy is needed.  
5. Anonymous mapping:  
Anonymous page will use anon vma, and pagetable to map.  
6. Shared mapping:  
Different tasks, use different page table to points to same physical page, so data can be shared.  
It can use shared file (file shared mem), pseudo file (anon shared mem), or tmpfs file (shared mem).  
7. PFN mapping:  
Reserved mem and MMIO can use PFN mapping, in VMA, flag VM_PFNMAP will be added.  
8. Reverse mapping:  
It can be used in mem migrate, reclaim, compact scenarios.  
Normally it is:  
Page -> anon vma -> vma, when it is anon mapping (mapped by VMA -> anon VMA).  
Page -> address_space -> vma, when it is file mapping (mapped by VMA -> file -> inode -> address_space -> XARRAY).  

## MISCs  
1. Kernel space setup  
kernel will start from 0x7c00 in old config, while in new config, it can start from 0x100000 or other addr.  
Will load kernel image, and map sections like .data, .text, .bss and .__start.  
Then create linear mapping, vmalloc area and others.  
2. User space setup  
After create user process/ thread, each thread will have a task_struct, and points to mm_struct, there is pgd points to page table, and there are several vmas used for different mapping areas.  
When using user space mem, there are several ways, like Pre Alloc (VA + PA + PageTable), Lazy Alloc (VA) and On-Deman Alloc (VA, on-demand alloc PA + PageTable).  
3. Copy on write  
After fork, child process will have the resources owned by the parent, i.e. signal, open files, addr space. Resources are controlled via clone() flags, like CLONE_VM/FS/FILES/SIGHAND.  
Parent process's page table is marked as read only, and child process's page table is also read only. They have separate page tables, but map to same PA addr space. Because they share same vma mapping, so read can get same data. If child process write to the page, then page fault will be triggered, and new page allocated, parent's page data will be copied to child.  
For malloc() API, there are 2 sizes, > 128kB, it uses mmap(), < 128kB, it uses brk() API.  
When page is not mapped, there will be a page fault triggered, and handle_mm_fault() is called, then it will alloc pages from buddy allocator, and create page table accordingly.  
4. SWAP  
For anonymous mem, page table will store swap entry.  
For file mapped, or shared mem, swap entry will be stored in XARRAY, vma -> file -> inode -> address_space.  
Swap space contains swap entry, and physical pages can be recycled.  
For better performance, usually we disable swap by swapoff -a.  
5. ZSWAP  
Performance is better than swap, as it can be compressed in RAM, and no need to do disk IO.  
Page will be compressed, and stored in ZSWAP pool's zswap entry. There are special allocators like zsmalloc, zbud, z3fold.  
6. Reclaim  
a. There is watermark for reclaim usage, if free pages < WMARK_HIGH, hysteric's field low_on_memory will be set, until free pages become WMARK_HIGH, when low_on_memory is set, allocator will free some mem if GFP_WAIT is set.  
b. LRU lists. There are lruvec used for storing the pages, which belongs to active/inactive anon/file, when __alloc_pages_nodemaks() is called, it can start page reclaim, directly by APIs like shrink_active_list() or kswapd thread.  
c. Zone ref lists. In NUMA env, there are many zones can be used, and if higher zone is used up, can use lower zones (NORMAL used up -> use lower DMA32 zone). After mem is freed, reclaim will do its job.  
7. CMA  
For buddy allocator, it can use max 4MiB, but in some scenarios, we need > 4MiB memory, so we need CMA, and used together with DMA.  
CMA buffers can give back to buddy when mem pressure is high.  
8. Hugepage  
There are 2 kinds of hugepages normally, one is hugeTLB fs, and another is Transparent HugePage.
THP will migrate scattered 4k pages to 2M pages, so performance is worse than normal hugepage.  
Normal hugepage can be reserved in manually via: 
```bash
# Reserve 6G hugepages.  
echo 6 > /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages   
```
THP can be used via:  
```bash
# Enable  
echo "madvise" > /sys/kernel/mm/transparent_hugepage/enabled  
# Disable
echo "never" > /sys/kernel/mm/transparent_hugepage/enabled  
```
The main difference is that the Transparent HugePages are set up dynamically at run time by the khugepaged thread in kernel while the regular HugePages had to be preallocated at the boot up time.  
9. Per cpu variables  
Per cpu variables don't need extra synchronizations, so it is fast for use.  
DEFINE_PER_CPU(type, var)  
get_cpu_var(var);  
put_cpu_var(var);  
10. Useful device nodes  
Those nodes are useful for debug, like /dev/mem and also kmem, null, port, zero, random, urandom, kmsg.  
i.e. we can read kernel physical mem via /dev/mem, and create a zeroed file by dd from /dev/zero, also output to /dev/null.  
## Debug  
OOM: Usually there are OOM logs, we can check which file/anon and each zone's free mem.  Also can check /proc/meminfo for Buffers, Cached, Shmem (Shmem and tmpfs), slab, vmallocused, usages.  
Overwrite: Can use mprotect() to protect that page after write, and when others write it, it will trigger crash, so we can know who did that.  
Kernel mem leak: we can use kmemleak tool or config option.  
User mem leak: can check VSS/RSS/PSS/USS via smem tool collection.  

# Virtualization  
There is full virtulizatoin, and para virtualization.  
Software Para-Virtualization:  
Ring 0 is hypervisor and host OS. Ring 1 is guest OS. Ring 3 are apps.  
Software Full-Virtualization:  
Ring 0 is hypervisor. Ring 1 is guest OS. Ring 3 are apps.  
Hardware Full-Virtualization:  
Ring 0 is hypervisor and guest mode. Ring 3 are apps.  

Hypervisor won't care about what OS will run above them.  
Linux kernel has KVM to support full virtualization. Qemu can work together with KVM.  

# References  
Many thanks for Buddy, Zhang (buddy.zhang@aliyun.com), I learned a lot from him, especially memory part.  
https://lbomr.xetlk.com/s/4Ah3Md  
https://github.com/BiscuitOS/BiscuitOS  
https://www.kernel.org/doc/html/latest/scheduler/index.html  
https://www.kernel.org/doc/Documentation/sysctl/vm.txt  
https://www.kernel.org/doc/html/latest/core-api/wrappers/atomic_t.html  
https://dl.acm.org/doi/pdf/10.1145/3342195.3387517  
https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43438.pdf  
https://cloud.tencent.com/developer/article/1821725  
