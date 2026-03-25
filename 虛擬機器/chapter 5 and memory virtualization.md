# Motivation
We know that memory management in a virtualized environment requires two distinct page tables. The Guest Page Table (managed by the VM) defines the mapping between the Guest Virtual Address (GVA) and the Guest Physical Address (GPA). Meanwhile, the Host Page Table (managed by the VMM/Hypervisor) defines the mapping between the GPA and the Host Physical Address (HPA).

>[!question] 
> For a memory access initiated by the VM's CPU, the GVA needs two transformations to resolve to the actual HPA. How do the hypervisor and the underlying hardware coordinate these two page tables to achieve efficient address translation?

>[!question] 
>There seems to have two page tables do the transfomation between GVA and HPA, but the MMU can only do the transformation between virtual address and physical address.

---
# Shadow paging

>[!motivation]
>We as description in motivation creat two page tables. One table for transforming GVA to GPA managed by guest os, and the other for transforming GPA to HPA managed by hypervisor. 
>Since the MMU can only walk a single page table, the hypervisor makes and maintains a page table called shadow page table that maps GVA to HPA for the CPU to use.

An easy method is that we set the memory of guest page table as read only. Once the VM wants to do update a page table entry (allocating new memory, freeing memory, changing page permissions, or swapping a page to disk), VM will write to guest page table. Since guest page table is set to read only, write operation will cause a trap, and do VM exits. VMM will intercepts this trap, emulates the guest's write to the guest page table, and updates the corresponding entry in the Shadow Page Table. However, because the VMM is software running on the host system, it is subject to host memory management. When the VMM attempts to read the guest page table or write to the Shadow Page Table, it may trigger a host page fault (e.g., if those memory pages were swapped to disk). If this happens, the Host OS must first resolve the fault by bringing the pages back into physical RAM before the VMM can complete its emulation and resume the VM.

>[!question]
>There are many VMs run on your hypervisor. There may be a GPA with same value among some VMs. How does hypervisor distinguish same GPA from different VM.

The hypervisor achieves isolation by maintaining distinct shadow page tables for each VM (specifically, for each vCPU). When the hypervisor context-switches to run a specific VM, it loads the hardware page table base register (e.g., `CR3` on x86) with the HPA of that VM's specific Shadow Page Table. Therefore, even if VM 1 and VM 2 both use GPA `0x1000`, their respective Shadow Page Tables map that GVA/GPA to completely different isolated HPAs.

---
# Extended page table (nested page table)

>[!note] Drawback of shadow paging
>In shadow paging, any event that alters the active memory mapping in the guest page table-such as modifying page table entries (page fault) or updating the root page table pointer (context switch)-will cause a VM exit. The VMM must intercept these actions to either update the shadow page tables or swap the active shadow page table. This frequent need for VMM intervention introduces significant performance overhead due to the constant VM Exits.  

>[!motivation] Motivation for EPT: Faster Memory Access & Reduced VMM Intervention
>#### Decoupling page faults
>Actually there are two types of page fault that happen in transformation from GVA to HPA
>- Type 1 : The guest page fault (GVA $\to$ GPA fails)
>- Type 2 : The EPT violation (GPA $\to$ HPA fails)
>
>Notice that the type 1 page fault will not necessarily lead to type 2 page fault. For example,  if the Guest OS swaps a page out to its virtual hard drive and later brings it back, it triggers a Type 1 fault. Hence if we can have type 1 handled natively by the guest os, the VMM intervention will be introducing less performance overhead.
>####  Native Context Switching
>EPT eliminates VMM intervention for routine OS operations like context switching
>- Shadow paging : Updating the root page table pointer (e.g., the `CR3` register) forces the VMM to intercept and swap the active shadow page table, triggering a costly **VM Exit**.
>- EPT : The `CR3` register inside the guest only holds a Guest Physical Address (GPA). Updating it merely changes which guest page table the hardware MMU reads. Because the underlying EPT mapping ($GPA \to HPA$) remains unchanged, **no VM Exit occurs**.
>

![[EPT_transformation_step.png]]
source : Hardware and Software Support for Virtualization
The Figure above shows how does the GVA transfter to the HPA. We may notice that if the guest-physical tree is n-level and host-physical tree is m-level then the transformation of GVA to HPA will take $n*(m+1) + m$ steps. It seems will take lots of time, but with the help of TLB this cost reduces a lot.

---
# Handling page fault

Since there are only two scenarios that we could encounter page fault
1. Permission denied
2. Page table or frame is not in the memory
### Shadow paging

>[!question] When can we encounter page fault in shadow pagging ?
>When using shadow paging, three page tables must be maintained: the guest page table (stored in the VM), the host/hypervisor page table (stored in the VMM), and the shadow page table (also stored in the VMM).
>Consequently, a page fault can occur at three different levels:
>1. Guest page fault : Occurs in the Guest OS if a valid page table entry cann't be found in the guest page table. The guest OS handles this.
>2. Host page fault : Occurs if a valid PTE can't be found in the host page table (e.g. if those memory pages were swapped to the host's physical disk). If this happens, the host OS must first resolve the fault by bringing the pages back into physical RAM before the VMM can complete its emulation and resume the VM.
>3. Shadow page fault : Traps to the hypervisor if the hardware can't find a valid PTE in the shadow page table. The VMM resolves this by synchronizing the shadow table with the guest's mapping.

The shadow page table is quite useless since it only combined the information in guest page table and host page table. However MMU can only read one page table, so it's necessary to keep a page that record mapping from GVA to HPA. Moreover transforming from the GVA will make the thing complicated since every process's virtual address space are the same. Hence if we do context switch in guest OS, the shadow page table must changed which will cause VM exits. 

>[!note]
>We want a hardware support architecture that can do transformation from GVA to HPA without shadow page table. Here comes the extend page table.

>[!note]
>We can do permission check on all three page tables.
### Extend page table

>[!question] When can we encounter page fault in extend page table ?
>When using extend page table, two page tables must be maintained: the guest page table (stored in the VM) and the Extended Page Table (stored in the VMM).
>Consequently, a page fault can occur at two different levels:
>1. Guest page fault : Occurs in the Guest OS if a valid page table entry cann't be found in the guest page table. The guest OS handles this.
>2. EPT Violation (Hypervisor trap) : Occurs if the hardware MMU cannot find a valid GPA $\rightarrow$ HPA mapping in the EPT (e.g., the host OS swapped the guest's memory to disk). This traps to the hypervisor, which fetches the page and updates the EPT.

>[!note] Key observation
>GVAs are unique per-process within the guest, but the GPA space is unique per-VM. Because EPT maps GPA $\rightarrow$ HPA, a guest OS context switch (changing GVAs) does not require the hypervisor to alter the EPT at all.

We can do context switch in the guest OS by simply change `CR3` in intel and doing context switch in hypervisor (e.g., change VM1 to VM2) by simply change `EPTP` in intel. 

>[!note]
>We can do permission check on both page tables.

---
# How to Collaborate with TLB

>[!question] In shadow page table, the same GVA value in different contexts maps to different HPAs. How to distinguish the same GVA used by different processes?
>The simple approach is flush all TLB entries when context switch in VM or context switch in VMM, but the cost is huge.  In the specific process the GVA is unique (we may view a VM as a process). Hence we only need to add a tag to tell different shadow page table (process) apart. We call this tag as ASID (Address Space Identifiers) in ARM and PCID (Process-Context Identifiers) in intel. 

>[!question] In EPT/NPT, the same GPA value in different contexts maps to different HPAs.  How to distinguish the same GPA from different VMs ? 
>The simple approach is flush all TLB entries when context-switching VMs, but the cost is huge.  In the specific VM, GPA is unique. Hence we only need to add the tag that can tell different VM apart to each TLB entry. We call this tag as VMID (Virtual Machine Identifier) in ARM and VPID (Virtual Processor ID) in intel. 

**Note:** Modern CPUs combine these tags—like **VPID + PCID**—to uniquely identify a specific process running inside a specific vCPU. This allows the TLB to retain entries across both process switches and VM exits, eliminating unnecessary flushes!

---