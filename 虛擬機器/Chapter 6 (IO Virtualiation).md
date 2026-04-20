The three key attributes proposed by Popek and Goldberg are:
- equivalence
- safety
- performance
These attributes help us understand virtualization from a CPU and MMU.
When introducing I/O capabilities to virtual machine, **interposition** becomes handy
>This property allows the host  to **transparently observe, control, and manipulate this I/O**, thereby **decoupling it from the underlying physical I/O devices** and enabling several appealing benefits.

>[!question] Why introducing I/O capabilities to virtual machine, **interposition** becomes handy?
>The next section "Benefits of I/O Interposition" will give you an answer.

>[!question] Why is interposition discussed less in CPU virtualization compared to I/O virtualization?
>The reason continuous interposition is less prominent in CPU virtualization compared to I/O virtualization comes down to hardware uniformity and execution models. Traditional I/O hardware is highly diverse and complex, requiring the hypervisor to actively interpose, translate, and emulate device behavior to route requests to physical hardware safely.
>In contrast, CPU virtualization relies on **Limited Direct Execution**. Because the CPU instruction set is standardized, the guest OS can run unprivileged instructions directly natively on the physical CPU without hypervisor involvement. The hypervisor only uses a **trap-and-emulate** approach to interpose on privileged instructions, making CPU virtualization significantly lighter and closer to native performance.
# Benefits of I/O Interposition
1. Interposition allows the hypervisor to encapsulate the entire state of the virtual machine—including that of its I/O devices—at any given time. The hypervisor is able to **encode the state of the devices** because
	- implements these devices in software.
	- interposing on each and every VM I/O operation.
2. The hypervisor’s role is to map between virtual to physical I/O devices. Thus, it can suspend the VM's execution on a source server, copy its image to a target server, ans resume execution there.
3. Since we give the virtual I/O to VM, we can add new features that are not natively provided by the physical device.
	- allow for snapshots
	- time travel capabilities
	- storage deduplication
4. I/O interposition allows hypervisors to apply all the canonical memory optimizations to the memory images of VMs
	- memory overcommitment
	- page migration
	- transparent huge pages

>[!question] Why we can migrate the VM to a new physical machine with different I/O devices?
>You can migrate a VM between machines with different I/O devices because the **VM is completely blind to the physical hardware**. It relies on standard, software-defined virtual devices that are identical on both the source and destination hypervisors.

>[!question] What's memory overcommitment?
>It is a system management technique where an operating system or hypervisor allocates more memory to processes or Virtual Machines (VMs) than physically exists on the host machine.

>[!question] What's page migration?
>Page migration is an advanced memory management technique used by operating systems to move a block of memory (a "page") from one physical hardware location to another, without disrupting the application that is using it.
>
>**Why we need this technique?**
>In the NUMA (Non-Uniform Memory Access) architecture, there are many CPUs in the motherboard and the RAM is physically divided and wired directly to specific CPUs.  If the CPU 1 wants to access the data in the CPU 2, it must send the request across the motherboard highway to CPUs, ask CPU 2 to fetch the data from its local RAM, and then send the data back across the highway. Hence the speed of accessing data in RAM may be different. In this case we may migrate the page to proper CPU. Page migration

>[!question] What's transparent huge page (THP)?
>When THP is turned on, the OS runs a background process that constantly watches the system's memory. If it sees an application using a lot of standard 4KB pages right next to each other, the OS secretly groups them together and upgrades them into a single 2MB Huge Page.
 
# Physical I/O
## Discovering and Interacting with I/O Devices
![[a possible internal organization of an Intel server.png]]

- CPUs communicate via the Intel **QuickPath Interconnect** (QPI).
- CPU cores access the **memory modules(DIMMs)** through their memory controllers.
- CPU cores communicate with the various I/O devices through the **relevant host bridges**
- communications with Peripheral Component Interconnect Express (PCIe) devices flow through the **host-to-PCIe bridge**.
The **PCIe fabric** poses the greatest challenge for efficient virtualization, because it **delivers the highest throughput as compared to other local I/O fabrics**.

>PCIe fabric: The entire interconnected web of PCIe links, switches, and endpoint devices that allow data to flow across the motherboard.

At boot time, the operating system kernel must somehow figure out which devices are available in the physical machine. The **firmware (BIOS or UEFI)** then provides a description of the available devices in some standard format, such as the one dictated by the **Advanced Configuration and Power Interface (ACPI)**.

>[!question] Why we move from BIOS to UEFI?
>We transitioned from BIOS to UEFI because the decades-old BIOS architecture simply could not support the demands of modern computer hardware. Most notably, **BIOS is mathematically restricted to a 2.2-terabyte limit for boot drives**, whereas UEFI easily handles massive modern storage capacities. Furthermore, **BIOS operates in a slow 16-bit mode** that initializes hardware sequentially, creating a bottleneck during startup. In contrast, UEFI utilizes 64-bit processing to load components in parallel, resulting in significantly faster boot times. UEFI also introduces critical security features like Secure Boot, which prevents malicious software from hijacking the system before the operating system even loads. Finally, this modern standard allows for a user-friendly graphical menu with mouse support, completely replacing the clunky, keyboard-only text screens of the past.

>[!question] What's memory modules(Dual In-line Memory Module, DIMMs)?
>It is your **RAM**. "Dual In-line" just refers to the fact that there are independent electrical pins on both sides of the stick.

>[!question] What's Peripheral Component Interconnect Express (PCIe) devices?
>It is the high-speed **expansion slot** system for adding heavy-duty accessories to your computer.  PCIe is the standard **physical connection interface** used for high-bandwidth devices. If you plug a dedicated Graphics Card (**GPU**), a blazing-fast NVMe **SSD** storage drive, or a **high-end Wi-Fi card** into a desktop motherboard, you are plugging it into a PCIe slot.

![[IO interact with the CPU and Memory.png]]

>[!note] MMIO (Memory-mapped I/O) and PIO (Port-mapped I/O)
>Port-mapped I/O (PIO) and memory-mapped I/O (MMIO) provide the most basic method for **CPUs to interact with I/O devices**. 
>- Addresses of PIO—called “ports”—are separated from the memory address space and have their own dedicated physical bus. They are used via the special OUT and IN x86 instructions, which write/read 1–4 bytes to/from the I/O devices.
>- MMIO is similar, but the device registers are associated with physical memory addresses, and they are referred to using regular load and store x86 operations through the memory bus.

>[!note] DMA (Direct Memory Access)
>Using PIO and MMIO to move large amounts of data from I/O devices to memory and vice versa can be **highly inefficient**.  A better, more performant alternative is to **allow I/O devices to access the memory directly**, without CPU core involvement. Such interaction is made possible with the **Direct Memory Access(DMA) mechanism**. The core only initiates the DMA operation, asking the I/O device to **asynchronously notify** it when the operation completes (via an **interrupt**).

>[!note] Interrupts
>I/O devices trigger **asynchronous event notifications** directed at the CPU cores by issuing interrupts. Each interrupt is associated with a number (denoted “interrupt vector”), which corresponds to an entry in the x86 Interrupt Descriptor Table (IDT).

>[!note] LAPIC
>During execution, the OS occasionally performs interrupt-related operations, including: enabling and disabling interrupts; notifying the hardware upon interrupt handling completion; sending inter-processor interrupts (IPIs) between cores; and configuring the timer to deliver clock interrupts. All these operations are performed through the per-core Local Advanced Programmable Interrupt Controller (LAPIC).

>[!question] Why we need to send inter-processor interrupts (IPIs) between cores?
>While hardware automatically manages _cache coherency_ (ensuring all cores see the same data in their L1/L2 caches without OS intervention), the operating system relies on **IPIs** to manage _state coherency_—ensuring that all cores are synchronized regarding memory maps, thread scheduling, power states, and system health.
## Driving Devices through Ring Buffers
I/O devices, such as PCIe solid state drives (SSDs) and network controller (NICs), can deliver high throughput rates. Overwhelmingly, these devices stream their I/O through one or more producer/consumer **ring buffers**. 

![[IO ring demonstration.png]]

>[!note] What's ring buffer
>A ring is a memory array shared between the OS device driver and the associated device. The ring is circular in that the device and driver wrap around to the beginning of the array when they reach its end. The entries in the ring are called **DMA descriptors**.  Their exact format and content may vary between I/O devices, but they typically specify at least the **address and size** of the corresponding DMA target buffers—the memory areas used by the device DMAs to write/read incoming/outgoing data. The descriptors also commonly contain status bits that help the driver and the device to synchronize.
>- DMA descriptors usually stores the position of the data, rather than data itself.

>[!note] The direction of each requested DMS
>Devices must also know the **direction of each requested DMA**, namely, whether the data should be **transmitted from memory (into the device) or received (from the device) into memory**. The direction can be specified in the **descriptor**, as is typical for disk drives, or the device can employ **different rings** for receive and transmit activity, as is typical for NICs. (In the latter case, the direction is implied by the ring.)

>[!note] How does the ring buffer work with the case that OS wants to transmit two packets?
>- Step 1: The OS device driver allocates the rings and configures the I/O device with the ring sizes and base locations.
>- Step 2: The Tx head and tail point to the same descriptor $k$, signifying (with status bits) that Tx is empty.
>- Step 3: The OS driver sets the $k$ and $k + 1$ descriptors to point to the two packets, turns on their “produced” bits, and lets the NIC know that new packets are pending by updating the tail register to point to $k + 2$ (modulo N).
>- Step 4: The NIC sequentially processes the packets, beginning at the head $(k)$, which is incremented until it reaches the tail $(k + 2)$.
>- Step 5: With Tx, the head always “chases” the tail throughout the execution, meaning the NIC tries to send the packets as fast as it can.
>
> Note that: The tail is updated by OS and the head is updated by devices. The CPU notifies the device of pending packets by updating the tail register via PIO/MMIO.

>[!question] What's the benefit of using a ring buffer?
>We first look at the Poducer-Consumer model. In this scenario, a Producer continuously creates data, and a Consumer continuously processes it.
>1. The benefit of a buffer: Producers and consumers **rarely operate at the exact same speed.** A buffer acts as a waiting area, **absorbing temporary bursts of data.** This allows the producer to drop off data quickly without having to wait for the consumer to finish processing the previous batch.
>2. The benefit of the Ring: We can implement a buffer using a standard array, but arrays have a fixed length. If we process data from the front, we either leave wasted space or we are forced to slowly shift all remaining data forward. A **ring buffer** (circular buffer) solves this. When the data reaches the end of the array, the pointer simply wraps around to the beginning.
## PCIe
The topology of the PCIe fabric is arranged as a tree, as illustrated below
![[PCIe tree topology.png]]

![[PCIe configuration space.png]]

>[!note] Message Signaled Interrupts (MSI)
>PCIe supports a third type—for interrupts. Message Signaled Interrupts (MSI) allow a device to send a PCIe packet whose destination is a LAPIC of a core. The MSI interrupt propagates upstream through the PCIe hierarchy until it reaches the host bridge, which forwards it to the destination LAPIC that is encoded in the packet.
# Virtual I/O without Hardware Support
Our guest is unaware of the fact that it must share, and it would not know how even if it did. Consequently, allowing the guest to access the disk drive directly would most likely result in an immediate crash and permanent data loss.

>To avoid this problem, the hypervisor must prevent guests from accessing real devices while sustaining the illusion that devices can be accessed.

## I/O Emulation (Full Virtualization)
We have noted that
1. The OS discovers and “talks” to I/O devices by using MMIO and PIO operations.
2. The I/O devices respond by triggering interrupts and by reading/writing data to/from memory via DMAs.

The hypervisor can therefore support the illusion that the guest controls the devices by
1. arranging things such that every guest’s PIO and MMIO will trap into the hypervisor
2. responding to these PIOs and MMIOs as real devices would: injecting interrupts to the guest and reading/writing to/from its (guest-physical) memory as if performing DMAs.
>[!note] How to emulate DMA, MMIO, PIO
>- **DMA**: Emulating DMAs to/from guest memory is trivial for the hypervisor, because it can read from and write to this memory as it pleases.
>- **MMIO**: Guest’s MMIOs are regular loads/stores from/to guest memory pages, so the hypervisor can arrange for these memory accesses to trap by mapping the pages as reserved/non-present (both loads and stores trigger exits) or as read-only (only stores trigger exits).
>- **PIOs**: Guest’s PIOs are privileged instructions, and the hypervisor can configure the guest’s VMCS to trap upon them. Likewise, the hypervisor can use the VMCS to inject interrupts to the guest.

**Every hosted virtual machine is encapsulated within a QEMU process, such that different VMs reside in different processes**. For every virtual device that QEMU hands to its VM,it spawns another thread, denoted as **“I/O thread”**. VCPU threads have two execution contexts:
- one for the guest VM: The role of the host VCPU context is to handle exits of the guest VCPU context.
- one for the host QEMU: The role of the I/O thread isto handle asynchronous activity related to the corresponding virtual device, which is **not synchronously initiated by guest VCPU contexts**.

![[IO emulation in the KVMQEMU hypervisor.png]]

>[!note] The workflow of I/O virtualization (full virtualization)
>- Step 1: The guest VM device driver issues MMIOs/PIOs to drive the device.
>- Step 2: Since the device is virtual (these operations are directed to read/write-protected memory location), triggering exits that suspend the VM VCPU context and invoke KVM
>- Step 3: KVM relays the events back to the very same VCPU thread, but to its host, rather than guest, execution context. (Because we have VT-x)
>- Step 4: QEMU’s device emulation layer then processes these events typically through regular system calls. (QEMU is in ring 3 of VMX **Root** Mode and )
>- Step 5: The emulation layer emulates DMAs by writing/reading to/from the guest’s I/O buffers
>- Step 6: resumes the guest execution context via KVM, possibly injecting an interrupt to signal to the guest that I/O events occurred.
## I/O Paravirtualization
While I/O emulation implements a correct behavior, it might **induce substantial performance overheads**. 

>[!question] Why I/O emulatioin implements induce substantial performance overheads?
>I/O emulation overhead stems from two primary factors. First, legacy hardware was not designed with virtualization in mind, necessitating a **'trap-and-emulate'** approach. This requires frequent, expensive context switches (VM Exits/Entries) between the Guest and the Hypervisor. Second, **MMIO enforcement** occurs at page-level granularity (e.g., 4KB). Because devices often require protection for only small registers, entire pages must be trapped, leading to redundant intercepts and high processing costs for even minor I/O operations.

Virtualization overheads caused by inefficient interfaces of physical devices could, in principle, be eliminated, if we **redesign the devices to have virtualization-friendlier interfaces**. I/O paravir tualization, whereby **guests and hosts** agree upon a (virtual) device specification to be used for I/O emulation, with the explicit goal of minimizing overheads. 

>Such a device is said to be **paravirtual (rather than fully virtual), as it makes the guest “aware” that it is being virtualized**: the **guest must install a special device driver** that is only compatible with its hypervisor, not with any real physical hardware.

![[virto framework of paravitual.png]]

>[!note] Virtio
>The framework of paravirtual I/O devices of KVM/QEMU is called virtio, offering a common guest-host interface and communication mechanism. As usual, each paravirtual device driver (top of the figure) corresponds to a matching emulation layer in the QEMU part of the hypervisor (bottom). The central construct of virtio is **virtqueue**, which is essentially a ring where buffers are posted by the guest to be consumed by the host. Guests that use virtqueues never trigger exits unless they consciously intend to do so, by invoking the `virtqueue_kick` function.

>[!note] Virtqueue
>Each virtqueue is associated with two modes of execution that assist the guest and host to further reduce the number of interrupts and exits. The modes are `NO_INTERRUPT` and `NO_NOTIFY`.
>- `NO_INTERRUPT`: it informs the host to refrain from delivering interrupts associated with the paravirtual device until the guest turns off this mode.
>- `NO_NOTIFY`: it informs the guest to refrain from kicking it.
>
>Note: When a **guest virtual machine** wants to send a large amount of data, it turns on the `NO_INTERRUPT` mode to stop the host from sending constant interrupts. For example, the virtio-net transmission queue uses this mode to silently recycle used buffers, only re-enabling interrupts when the queue is nearly full. Conversely, when the **host** is processing a sudden batch of data sent by the guest, the **host turns on** the `NO_NOTIFY` mode. This specific mode signals the guest to stop repeatedly "kicking" or waking up the host for every new packet added to the queue. Since TCP traffic is often bursty, the host uses this mode to process an entire loop of frames uninterrupted after receiving just one initial kick from the guest. Together, these two mutually beneficial execution modes drastically reduce virtualization overhead and improve overall network performance.



![[Netperf paravirtual result.png]]

By Table 6.2, virtio-net performs significantly better than e1000, delivering 22x higher throughput. The reasons are
- reducing the number of exits
- reducing the number of exits
- virtio kicks KVM explicitly when needed, rather than implicitly triggering unintended exits due to legacy PIOs/MMIOs.
- This stack employs a **batching algorithm** that aggregates messages, attempting to get more value from the NIC’s TSO capability, as larger segments translate to less cycles
	- TSO: TCP Segmentation Offload
	- The profile of e1000 networking, which is much slower, discourages this sort of segment aggre gation.
## Front-Ends and Back-Ends

![[front-end and back-end of Paravirtual.png]]
- front-end: It encompasses a guest virtual device driver and a matching hypervisor emulation layer that understands the device’s semantics and interacts with the driver at the guest.
- back-end: It is used by the front end to **implement the functionality** of the virtual device using the physical resources of the host system.
## Summary
>Hypervisors employ I/O emulation by **redirecting guest MMIOs and PIOs to read/write-protected memory**. Emulation exposes virtual I/O devices that are implemented in software but support hardware interfaces. **Paravirtualization improves upon emulation by favoring interfaces that minimize virtualization overheads**. The downside of paravirtualization is that it requires hypervisor developers to **implement per-guest OS drivers**, and it hinders portability by requiring users to **install hypervisor-specific software**.

# Virtual I/O with Hardware Support
If we are willing to forego I/O interposition and its many advantages and there is an extra physical devices not strictly need, the hypervisor may assign that physical device to the VM, such that no other VM and hypervisor could use that physical device. This approach denoted **direct device assignment** as illustarted bellow.
![[direct device assignment demonstration.png]]

>[!question] What's the drawback of utilizing direct device assignment? 
>1. Scalabilty (Hardware Limitations): We need extra physical hardware to support direct device assignment, but the number of VM is larger than the number of physcial device in the modern server. Hence we don't have enought hardware to assign a dedicated device to every VM.
>2. Security (Loss of Isolation): When a VM is given direct control of a physical device, that device uses Direct Memory Access (DMA) to transfer data. Without advanced hardware protections, DMA bypasses the CPU's standard memory management. This gives the device (and therefore the VM controlling it) the ability to access the entire physical memory of the host. This completely breaks VM isolation, significantly increasing the risk that a malicious VM could compromise the host or other VMs.
>3. Loss of Flexibility (Live Migration & Memory):  Directly assigning physical hardware permanently ties the VM to that specific host machine. Because of this, the hypervisor cannot perform **Live Migration** (moving a running VM from one physical server to another without downtime). Additionally, the hypervisor cannot safely swap or move the VM's memory, forcing all of the VM's memory to be "pinned" (locked) in the physical RAM.

- I/O Memory Management Unit (IOMMU): handle security.
- Single-Root I/O Virtualization (SRIOV): handle scalabity
The combination of **SRIOV** and **IOMMU** makes device assignment a viable performant approach, leaving one last major source of virtualization overhead: interrupts.

>Note: IOMMU is also used in the commuication between user space program and physical device. By using the IOMMU, the OS can say: "I am going to map this physical network card directly to this user-space program. The program can use DMA to talk to the card directly. The IOMMU will make sure the program doesn't accidentally (or maliciously) overwrite other memory."
## IOMMU
Recall that device drivers of the operating system initiate DMAs to **asynchronously** move data from devices into memory and vice versa, **without having to otherwise involve the CPU**.

>[!note] The previous workflow of hardware using DMA (without IOMMU)
>Since the hardware device only understands physical addresses, the device driver (which is software running inside the Operating System) has to do a little bit of setup work first.
>-  Step 1 (Virtual Allocation): The OS allocates a buffer for the data using virtual memory.    
>- Step 2 (Translation by the Driver): When the device driver is ready to tell the device to start moving data, the driver looks up the virtual address in the OS's page tables to find out exactly where that buffer lives in _physical_ memory. 
>- Step 3 (Pinning): The driver "pins" that memory so the OS doesn't accidentally page it out to the hard drive while the device is trying to access it. 
>- Step 4 (Handing off the Physical Address): The driver populates the device's "ring buffer descriptors" with the **raw physical address**.
>- Step 5 (Execution): The device uses that physical address to write directly to the RAM.

>[!question] The problematic design of previous DMA
>- Security: VM with assigned devices can **indirectly** read/write any memory location. VMs can **indirectly** trigger any interrupt vector they wish
>- Accessibilty: VMs don't know the (real) physical location of their DMA buffers, since they use guest-physical rather than host-physical address. 

>[!question] What's the meaning of indirectly read/write?
>- Direct Access: It typically executes a CPU instruction (like `LOAD` or `STORE`). The CPU directly attempts to access the RAM.
>- Indirect Access: It uses a **proxy**—the hardware device assigned to it (like a physical network card passed through to the VM). The VM programs the device, saying, _"Hey, please perform a DMA transfer to/from this physical address."_ The device then does the actual reading or writing on the VM's behalf.

The IOMMU consists of two main components
- DMA remmaping engine (DMAR): DMAR allows DMAs to be carried out with I/O virtual addresses (IOVAs)
- Interrupt remapping engine (IR): IR translates interrupt vectors fired by devices based on an interrupt translation table configured by the hypervisor.

>[!note] DMA Remmaping
>![[IOVA and IOMMU.png]]
>- PCIe dictates that each DMA will be associated with the 16-bit bus-device-function (BDF) number that uniquely identifies the corresponding I/O device in the PCIe hierarchy
>- The DMA propagates upstream until it reaches the root complex (RC) where the IOMMU resides.
>- The IOMMU uses the 8-bit bus identifier to index the root table in order to retrieve the physical address of the context table. It then indexes the context table using the 8 bit concatenation of the device and function identifiers. The result is the **physical location of the root of the page table hierarchy** that houses all the IOVA$\Rightarrow$PA translations of this **particular I/O device**.
>- The IOMMU walks the page table similarly to the MMU, checking for translation validity and access permissions at every level.
>- IOMMU caches translations using its IOTLB
>- IOMMU typically does not handle page faults gracefully. I/O devices usually expect their **DMA target buffers to be present and available**.
>
>![[IOMMU possible combination.png]]
>2D IOMMU transfers gVA(IOVA) to hPA and the whole procedure works like nested paging. This technique is supported in moder server like Intel VT-d and ARM SMMUv3.
>It make sense to virtualize the IOMMU, such that guest and host would directly control their own page tables, and the hardware would conduct a 2D page-walk. 
>- 2D IOMMUs allow guests to protect themselves against errant or malicious I/O devices (by mapping and unmapping the target buffer of each DMA right before the DMA is programmed and right after it completes.).
>- 2D IOMMUs helps guest to use legacy devices.
>- 2D IOMMUs allow guests to directly assign devices to their own processes, allowing for user-level I/O.


>[!question] Why I/O devices usally expect their DMA target buffers to be present and available? The RAM can accommodate all page table of I/O devices?
>I/O devices require their Direct Memory Access (DMA) target buffers to be permanently present in physical memory because standard DMA controllers lack the ability to handle page faults. If the operating system swaps a target buffer to disk or reallocates its physical address during a transfer, the DMA hardware will either fail or corrupt other memory spaces. To prevent this, the operating system "pins" the required memory pages in RAM to ensure they remain securely locked for the duration of the hardware operation. As for your second question, the system's RAM can effortlessly accommodate the page tables used by I/O devices, which are typically managed by an I/O Memory Management Unit (IOMMU). The RAM footprint remains extremely small because the operating system uses dynamic mapping, creating minimal, temporary page table entries only when a specific DMA transfer is actively occurring. Additionally, modern systems use hierarchical page structures or allow advanced devices to share the CPU's existing page tables, meaning I/O memory tracking never overwhelms the physical RAM.

>[!note] Interrupt Remapping (IR)
>Recall that PCIe defines (MSI/MSI-X) interrupts similarly to DMA memory writes directed at some dedicated address range, which the RC identifies as the "interrupts space". Each interrupt request message is self-describing: it encodes all the information required for the RC (Root Complex) to handle it.>
>![[MSI interrupt demonstration.png]]
>First we take a look at figure 6.16. Since the device is directly assigned to VM, the latter can program the former to DMA-write any value into the interupt space. Without IR, the IOMMU can't distinguish between a legitimate, genuine MSI interrupted fired by d and a rogue DMA that just pretends to be an interrupt.
>The solution is keep the IR table pointed by the IR Table Adress (IRTA) register at the IOMMU. IR table is set by the hypervisor and its entry contains d’s BDF for anti-spoofing, to indicate that d is indeed allowed to raise the interrupt associated with this IRindex. Only after verifying the authenticity and legitimacy of the interrupt, does the IOMMU deliver it to the hypervisor

>[!question] Why causing an unexpected interrupted from VM is problematic? The interrupt is handles by the OS kernel. It should be fine.
>While it is true that a traditional operating system can safely drop an unexpected interrupt on bare metal, virtualization changes the stakes completely. If a virtual machine triggers an unexpected interrupt that reaches the host hypervisor rather than just its own guest OS, it risks crashing the entire physical server and taking down every other VM with it. Even if the host kernel does not crash, a flood of unexpected interrupts forces computationally expensive context switches, draining CPU resources and causing a Denial of Service for all other tenants. Furthermore, a guest VM successfully generating physical host interrupts often indicates a critical failure in the hardware sandbox, potentially allowing a malicious user to exploit the entire system. This risk is especially high when using hardware passthrough, where a VM misconfiguring a physical device can send rogue interrupts that permanently lock up the host's hardware bus. Therefore, hypervisors must treat any unexpected interrupt from a VM as a severe threat to the stability, performance, and security of the shared infrastructure.
## SRIOV
Since 
1. physical servers can house only a small number of physical devices as compared to virtual machines
2. it is probably economically unreasonable to purchase a physical device for each VM
We utilize SRIOV to extends PCIe to support devices that can “self virtualize”. Namely, an SRIOV-capable I/O device can present multiple instances of itself to software.
An SRIOV device is defined to have at least one Physical Function (PF) and multiple Virtual Functions (VFs), which serve as the aforementioned device instances.
- PF: It is a standard PCIe function. It has a standard configura tion space, and the host software manages it as it would any other PCIe function.
- VF: It is a lightweight PCIe function that implements only a subset of the com ponents of a standard PCIe function.
	- doesn't have its own power management
	- can't (de)allocate other VFs.
When a VF is assigned to a virtual machine, the former provides the latter the ability to do direct I/O—the VMcansafely initiate DMAs, such that the hypervisor remains uninvolved in the I/O path.
![[SRIOV and NIC.png]]
## Exitless Interrupts
SRIOV and IOMMU eliminate most of the exits that occur when guests “talk” to their assigned devices (via MMIOs, for example), it **does not address the overheads generated when assigned devices “talk back”** by triggering interrupts to notify guests regarding the completion of their I/O requests.
>[!note] Basic VT-x Interrupt Support
>The VMCS stores the value of the guest’s Interrupt Descriptor Table Register (IDTR), such that it is loaded upon entering guest mode and saved on exit, when the hypervisor’s IDTR is loaded instead.

The hypervisor needs to maintain control, so it sets a VMCS control bit (denoted “external-interrupt exiting”), which configures the core to exit whenever an external interrupt fires. The chain of events that transpire when such an event occurs is depicted below
![[physical device without hardware support.png]]

>[!question] Why the physical device directly talk to guest with the help of IOMMU?
>An IOMMU primarily translates memory addresses to allow a physical device direct memory access (DMA), but it does not manage the CPU states required for interrupt delivery. Even with an IOMMU mapping the memory, a Message Signaled Interrupt (MSI) from the device arrives at the physical CPU, which cannot directly route it to a running virtual CPU. Consequently, the physical interrupt forces a VM-Exit, requiring the hypervisor to intercept the signal, acknowledge it, and manually inject a virtual interrupt into the guest OS. Likewise, when the guest finishes processing the interrupt, its End of Interrupt (EOI) signal targets a virtual interrupt controller, causing another VM-Exit for the hypervisor to handle.

>[!Important]
>Interrupt handling is a two-way street. When the guest OS finishes handling the interrupt, it must signal that it is done by writing an End of Interrupt (EOI) to its Local APIC (interrupt controller).

>[!note] Assigned EOI Register
>The operating system uses the **LAPIC to control all interrupt activity**. To this end, the LAPIC employs multiple registers used to configure, block, deliver, and (notably in this context) signal EOI. If we assume that an interrupt of a physical device can somehow be **safely delivered directly to the guest without hypervisor involvement**, then the EOI register should correspondingly also be assigned to the guest. Thankfully, the current LAPIC interface, x2APIC, exposes its registers using model specific registers (MSRs), which are accessed through “read MSR” and “write MSR” instructions. The CPU exits on LAPIC accesses according to an **MSR bitmap controlled by the hypervisor**. The bitmap specifies the “sensitive” MSRs that cannot be accessed directly by the guest and thus trigger exits. In contrast to **other LAPIC registers, with appropriate safety measures, EOI can be securely assigned to the guest**.

>[!note] Assigned Interrupts
>Assume a guest is directly assigned with a device. By utilizing a **software based technique called exitless interrupts (ELI)**, it is possible to additionally securely assign the device’s interrupts to the guest—without modifying the guest or resorting to paravirtualization. An assigned exitless interrupt does not trigger an exit. It is **delivered directly to the guest without host involvement**. ELI is structured based on the assumption
>- In high-performance, SRIOV-based device assignment deployments **nearly all physical interrupts arriving to a given core are targeted at the guest that runs on that core**.
>
>Hence we want the guest handle the interrupt by itself. While the guest initializes and maintains its own IDT, ELI runs the guest with a different IDT—called shadow IDT—which is prepared by the hypervisor. The hypervisor monitors the updates that the guest applies to its emulated, **write-protected IDT**, and it reacts accordingly so as to provide the desired effect.
>![[Shadow IDT.png]]
>By shadowing the guest IDT, the hypervisor has explicit control over which handlers are invoked upon interrupts. It thus configures the shadow IDT to
>1. deliver assigned interrupts directly to the guest’s interrupt handler
>2. force an exit for non-assigned interrupts by marking the corresponding IDT entries as non-present.

>[!question] what if we frequently updating shadow IDT? I think there will be lots of overhead
>Almost all IDT modifications occur only during the system boot process or when specific device drivers are initially loaded. Consequently, the hypervisor only absorbs the performance hit of the "trap and emulate" mechanism during these brief setup phases.

## Posted Interrupts



