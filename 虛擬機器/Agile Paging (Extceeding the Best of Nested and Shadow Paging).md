# Motivation
Nested paging and shadow paging have their own advantages and disvantages. Nested paging is good at modifying NPT since the guest page table can be maintained by the VM itself. However, it's not good at handling TLB miss since we need to do 2D traversal. On the other hand, shadow paging is good at handling TLB misses since we only need to do 1D traversal, but it has difficulty dealing with modifying the STP because modifying SPT will cause a VM trap. This leads to an idea
> Why can't we use these two technique together? NPT for modifying page table, and Shadow paging for TLB miss ?
# Observation
### Observation 1
Page tables are not modified uniformly: some regions of an address space see far more changes than others, and some levels of the page table, such as the leaves, are updated far more often than the upper-level nodes.
### Observation 2
Updates to  a page in page table are bimodal at a time interval of 1 second: only one update or many updates (e.g., 10, 50 or 500) within a second.
>For agile paging, if two writes to any level of the page table are detected by the VMM in a fixed time interval, then that level and all levels below it are moved to nested mode. This policy provides a small threshold like the one used in branch predictors for switching modes.
### Observation 3
Nested paging has been shown to be performing well for short-lived process and for processes that have a very small memory-footprint since they don't run long enough to amortize the cost of constructing a shadow pate table or do not suffer from TLB misses.
>With agile paging, an administrative policy can be made to start the process in nested mode (no use of shadow mode) and turn on shadow mode after a small time interval (e.g., 1 sec) if TLB miss overhead is sufficiently large.
# Measurement
We measure
- Total execution cycles for all six configurations 
	- base native ($E_B$) for both page size (4KB and 2MB)
	- nested paging ($E_N$) for both page size (4KB and 2MB)
	- shadow paging ($E_S$) for both page size (4KB and 2MB)
- Number of TLB misses ($M_{B/N/S}$) for both page size
- Cycles spent on TLB missed ($T_{B/N/S}$) for both page size
- Cycle spent in the hypervisor ($H_{B/N/S}$) for both page size
- Number for VMtraps ($V_{B/N/S}$) for both page size

We use tow-step approach to report improvement for agile paging
### Step 1:

>[!Goal]
>Creating a list of dynamically changing gVAs to classify under nested mode and calculate the fraction of VMtraps ($F_{Vi}$) that agile paging eliminates with reason "i" (level i page).

We process the trace to find which areas of the page tables are changing by looking at  the reasons for VM traps. We recordtheg VAs being dynamically changed due to changes on any level of the page table. This helps us create four lists of gVAs (since it's four level page table) under nested mode corresponding to their switching level of page table.The lists of gVAs are considered under nested paging for step2. This step also finds the fraction of VMM interventions that agile paging will reduce ($F_{Vi}$) from the trace since areas of the page  table under nested paging are known.
### Step 2:

>[!goal]
>Finding the fraction of TLB misses that would be serviced under nested mode ($F_{Ni}$) for each level ”i“ the switch occurs.

With nested paging along with BadgerTrap: a tool that converts all x86-64 TLB misses to a trap,which allows us to analyze TLB misses and classify TLB addresses, while enabling full-speed execution of instructions withTLB hits. We conservatively assume that when a TLB miss is serviced in nested mode $F_{N1}$ pays half the cost of a nested TLB miss beyond native and the rest $F_{N2}$, $F_{N3}$  and $F_{N4}$ pays full cost of nested paging.
### Cost of VMtraps
We use LMbench and microbenchmarks to measure the cost of the VMtrap for a context switch, page table update and page fault. We calculate reduction in cycles spent in the VMM, by subtracting the number of VMtraps of each type multiplied by its cost.
### Performance Model
The linear model takes the performance of shadow paging, subtracts the cost of VMtraps avoided, but adds in the higher cost of nested TLB misses.
# Question

>[!question] Using nested and shadow paging at the same time. How do you make sure SPT and NPT maintain correctly ?
>We maintain these two pages correctly with the hardware and VMM support. In hardware support, we have three architectural page table pointer: shadow, guest, and host page table. The shadow page table maintains a switching bit per page table entry for switch from shadow paging to nested paging (the opposite direction is not supported).  
>The VMM support makes these three pages work correctly. In guest page table, VMM marks as read-only just the parts of the GPT that covered by the partial SPT, and the other part marks as read-write access. The **shadow page table is partial and cannot translate all gVAs fully**. The shadow page table entry at each switching point holds the hPA of the next level of guest page table with the **switching bit set** (as shown in Figure 3). This enables hardware to perform the page walk correctly with agile paging using both techniques. The processor will walk the host page table for addresses using nested mode (at any level), and hence the VMM must build and maintain a complete host page table for each guest virtual machine as in nested paging. In these three page tables we use accessed, write-enable, and dirty bit to synchronize GPT under shadow mode. Note that when we do the page walk, the walk in different level are seperated.

>[!question] In shadpw paging, how to distinguish the GVA from different VMs?
>We need to handle this problem only when the context switch. Hence context switches require a  VMtrap for the VMM to determine the shadow page table for the imcomig process.
>

>[!question] Why in agile paging the context switching follows the mechanism used by the shadow paging for all processes instead of switching guest page table? We use nested paging so switch gpt is more nature.
>The guest OS wite to the gpt register, which triggers a trap to the VMM (like shadow pagging). We use the same technique since the agile paing starts from the shadow paging. The key reason is that if agile paging is enabled, virtualized page walk starts in shadow paging and then switches, in the same page walk, to nested paging if required. The agile paging starts in shadow pagging since the root page table won't change frequently.

>[!question] Why we default assume that the guest process starts in full shadow mode in agile paging?
>The main reason is that agile paging starts in shadow mode and switches at some level of the GPT to nested mode. Moreover in shadow paging modifying the PTE will cause VM trap which makes detecting dynamic page table more easily.

>[!question] Why we default assume that the guest process starts in full shadow mode in agile paging and allow guest process starts in nested paging?
>You misunderstand the agile paging. Agile paging provides a mechanism for virtualized ad dress translation that starts in shadow mode and switches at some level of the guest page table to nested mode.

>[!question] Why in agile paging, if two writes to any level of the page table are detected by the VMM in a fixed time interval, then that level and  all levels below it are moved to nested mode? Can we just make that level be nested?
>There are three reasons that we won't make a single level nested. The first and main reason is that the hardware only support the switch from the shadow to nested mode.  The second reason is that partial SPT doesn't have the information below that layer. The third reason is that a higher-level page table acts as a directory that points to lower-level page tables. If an OS is modifying a high-level directory (e.g., allocating a massive new chunk of memory that requires a new L2 table), it is almost guaranteed that the OS is about to violently modify the lower levels (L1) as well to map all the individual 4KB pages.

>[!question] Why in agile paging,  theVMM detects the page-table writes to clear referenced bits and converts leaf-level page tables to nested mode to avoid the VMtraps when memory is scarce?
>Since modifying guest page table is handled by hardware in nested mode. In nested mode, the Guest-initiated swapping of memory or page tables to the guest disk can be handled natively by the Guest OS without causing VM traps, **provided that the underlying Host Physical Pages (HPA) remain resident in the host's physical memory.**

>[!question] Why the parent level of the guest page table is converted to shadow mode before converting child levels?
>The reason is the as the previous question (Why in agile paging, if two writes to any level of the page table are detected by the VMM in a fixed time interval, then that level and  all levels below it are moved to nested mode? Can we just make that level be nested?)

>[!question] Why we use the linear model to predict the performance instead of implementing this technique and test it?
>- **Hardware is Hardwired:** Memory management is physically built into the CPU chip. Modifying it requires manufacturing custom silicon, which costs millions of dollars and takes years—far too expensive for a proof-of-concept.
>- **Kernel Code is Unstable:** Hacking deep into the operating system's memory subsystem is incredibly difficult. A single bug crashes the whole machine. Researchers want to prove the math works, not spend years debugging software.
>- **Zero System Noise:** Real-world computers have random variables like background tasks, OS updates, or thermal throttling that skew test results. A mathematical model provides a 100% sterile environment for precise, undisputed calculations.
>- **Simulating the Future:** Models allow researchers to test hypothetical "what-if" scenarios (e.g., _"What if next year's CPU is twice as fast?"_). You cannot physically test hardware that hasn't been invented yet.
