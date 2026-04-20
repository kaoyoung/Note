# Design Requirements

>[!question] Why we need the architectural support for virtualization
>We want to **eliminate the need for CPU paravirtualization and binary translation techniques**, and thereby enable the implementation of VMMs that can support a broad range of unmodified guest operating systems while **maintaining high levels of performance**.

The problems that we may face
- Ring alliasing and compression: guest kernel code designed to run at %cpl=0 must use the remaining three rings to guarantee isolation. This creates an alias as at least two distinct guest levels must use the same actual ring.
- Address space compression: the hypervisor must be somewhere in the linear address space in a portion that the guest software cannot access or use.
- Non-faulting access to privileged state: some of the infamous Pentium 17 instructions provideread-only access to privileged state and are behavior-sensitive. These structures are controlled by the hypervisor and will be in a different location than the guest operating system specifies.
- Unnecessary impact of juest transition: to address the efficiency criteria, it is essential that sensitive instructions (which must trigger a transition) be rare in practice. Unfortunately, modern operating systems rely extensively on instructions that are privilege-level sensitive, e.g., to suspend interrupts within critical regions, or to transition between kernel and user mode. Ideally, such instructions would not be sensitive to virtualization.
- Interrupt virtualization: Since the flag can never be cleared when the guest is running as this would violate safety, any direct use of that instruction by the guest will lead to incor rect behavior.
- Access to hidden state: the x86-32 architecture includes some “hidden” state, originally loaded from memory, but inaccessible to software if the contents in memory changed.

>[!Note]
>These problems all stem from a fundamental privilege conflict: an operating system kernel expects to run in the most privileged execution mode (Ring 0), but in a virtualized environment, the hypervisor must use that highest privilege to maintain isolation and control. Consequently, the guest OS is forced into a lower privilege level ('ring compression' or 'ring deprivileging'). This architectural shift creates significant challenges, specifically in handling sensitive instructions that the guest OS attempts to execute, as they either fail to trap to the hypervisor or behave incorrectly in lower privilege states.

>[!Note] The design goal of VT-x
>Intel's central design goal is to fully meet the requirements of the Popek and Goldberg theorem.
>- **Equivalence**: Intel’s architects designed VT-x to provide absolute architectural compatibility between the virtualized hardware and the underlying hardware.
>- **Safety**: Architectural support specifically designed for virtualization, a much-simplified hypervisor can provide the same characteristics with a much smaller code base. This **reduces the potential attack** surface on the hypervisor and the **risk of software vulnerabilities**.
>- **Performance**: Ironically, an increase in performance over existing, state-of-the-art virtualization techniques was **not** a release goal with the first-generation hardware support for virtualization. Indeed, the first generation processors with hardware support for virtualization were not competitive with state of-the-art solutions using DBT (Dynamic Binary Translation).

# The VT-X Architexture

>[!Note] The design principles
>1. Do **not** change the semantics of individual instructions of the ISA. 
>2. Duplicates the entire architecturally visible state of the processor and introduces a new mode of execution: the **root mode**. Hypervisors and host operating systems run in root mode whereas virtual machines execute in **non-root mode**.

>[!question] Why did we adopt these design principles for VT-x?
>As mentioned earlier, relying on the original architecture meant we had to address numerous complex issues, as outlined in the design requirements, which would have incurred significant performance overhead. Moreover, we have to make sure the new architecture maintained bakward compatibility with the existing ISA. To solve this, we introduced a **new execution tier** specifically to handle hypercalls from the VM. Essentially, this technique adds a new **architectural laye**r to cleanly intercept and manage virtual machine executions.

The architectureal extension has the following properties
- The processor is at any point in time either in root mode or in non-root mode and the transitions are atomic.
- The root mode can only be detected by executing specific new instructions, which are only available in root mode.
>[!note]
>1. Why hide the true "Root Mode" status?
>	In a nested virtualization scenario, you have multiple layers (e.g., L0 Bare-Metal Hypervisor → L1 Guest Hypervisor → L2 Innermost VM). L1 is technically just a guest running in "Non-Root Mode." However, because it is a Hypervisor itself, it firmly believes it has absolute authority over the physical hardware. If L1 easily discovers it is actually just a VM, it won't gracefully adapt. Traditional hypervisors aren't built to compromise; if they realize they don't have Ring 0 / Root control, they will likely crash (Kernel Panic/BSOD).
>2. The Clever Design of "Specific New Instructions"
>	Before VT-x, the x86 CPU architecture didn't have a "Root Mode." When Intel introduced this new mode, they paired it with a brand-new set of **VMX instructions** (like `VMXON`, `VMLAUNCH`, `VMREAD`). The hardware is strictly designed so that **you cannot check if you are in Root Mode by simply reading standard memory addresses or registers.** The _only_ way to interact with or verify Root Mode status is by attempting to execute these specific new VMX instructions.
>3. The "Trap and Emulate" Defense
>Restricting Root Mode detection exclusively to these new instructions is exactly what allows the underlying, true Hypervisor (L0) to seamlessly deceive L1.
>	- **L1's Perspective:** Believing it is the ultimate boss, L1 actively executes a privileged VMX instruction (like `VMXON`) to start building its own virtual machine. 
>	- **L0's Perspective:** The moment L1 tries to execute that VMX instruction, the hardware immediately triggers a **VM-Exit (a trap)**. It pauses L1 and hands control down to L0. L0 quietly **emulates** a successful execution in the background, updating virtual states to make it look like the command worked perfectly.   
>	- **The Result:** L0 hands control back to L1. Since L1 didn't receive any hardware errors or exceptions, it wholeheartedly believes it has successfully entered Root Mode and proceeds to create the L2 VM.

- This new mode (root vs. non-root) is only used for virtualization. It is 
	- Orthogonal to other modes of executions of the CPU.
	- Orthogonal to the protection levels of protected mode.
- Each mode defines its own distinct, complete 64-bit linear address space.
- Each mode has its own interrupt flag.
	- External interrupts are generally delivered in root mode and trigger a transition from non-root mode if necessary. The transition occurs even when non-root interrupts are disabled.

![[Standard use by hypervisors of VT-x  root and non-root modes.png]]

The picture above demonstrates how VT-x works. There are still four rings in each mode. The only thing we do just is switch different modes to handle the virtualization. Note that this design ensures the backward compatibility in the architecure since the four ring design isn't change.
## The Popek/Goldberg Theorem

The correspong core VT-x design principle can be informally framed as follows

>In anarchitecture with root and non-root modes of execution and a full duplicate of processor state, a hypervisor may be constructed if all sensitive instructions (according to the non-virtualizable legacy architecture) are root-mode privileged. 
>
>When executing in non-root mode, all root-mode-privileged instructions are either (i) implemented by the processor, with the requirement that they operate exclusively on the non-root duplicate of the processor or (ii) cause a trap.

>[!question] Why there are some root-mode-privileged instructions can run on the non-root duplicate of the processor? 
>In traditional virtualization,** forcing every privileged instruction to trap to the hypervisor caused massive performance bottlenecks** due to constant context switching. To solve this, modern hardware introduced a non-root mode that gives the guest operating system its own isolated duplicate of the processor's state. When a guest executes certain privileged instructions, the CPU intelligently redirects the operations to update only this localized duplicate rather than the actual physical hardware. **Because these instructions exclusively affect the guest's virtual sandbox, they cannot compromise the host system, allowing the hypervisor to safely maintain absolute control**. This hardware-level redirection completely bypasses the computationally expensive trap-and-emulate process for frequent operations like memory or interrupt management. Ultimately, allowing these instructions to run directly on the duplicate state enables virtual machines to achieve near-native execution speeds without sacrificing strict hardware isolation.
## Transitions between root and non-root modes

The state of the virtual machine is stored in a dedicated structure in physical memory called the **Virtual Machine Control Structure (VMCS)**. Once initialized, a virtual machine resumes execution through a `#vmresume` instruction. This privileged instruction loads the state from the VMCS in memory into the register file and performs an **atomic transition** between the host environment and the guest environment. The virtual machine then executes in non-root mode until the first trap that must be handled by the hypervisor or the next external interrupt. This transition from non-root mode to root mode is called a `#vmexit`. The picture below shows the whole process of `#vmexit` , `#vmlaunch`, and `#vmresume`.

![[VT-x transitions and control structure.png]]

The reasons for an exit can be grouped into categories including the following
- Any attempt by the guest to execute a root-mode-privileged instruction **that is configured to cause a trap rather than operating exclusively on the non-root duplicate**.
- The new vmcall instructions
- Exceptions result from the execution of any innocuous instruction in non-root mode
	- page fault caused by shadow paging
	- the access to memory-mapped I/O devices
	- general-purpose fault due to segment violations
- EPT violations
	- Page Fault
	- Permission Deny
- External interrupts that occurred while the CPU was executing in non-root mode
- The interrupt window opens up whenever the virtual machine has enabled interrupt and the virtual machine has a pending interrupt.
- The ISA extensions introduced with VT-x to support virtualization are also control  sensitive and therefore causea `#vmexit`, each with a distinct `exit reason`.
# KVM-A Hypervisor For VT-X

We use KVM, the lLinux-based Kernel Virtual Machine, as a case study ot put the innovation in practice.
- KVM is the most relevant open-source type-2 hypervisor.
- KVM relies on QEMU, a distinct open-source project, to emulate I/O.  Together with KVM, the combination is a type-2 hypervisor, with **QEMU** responsible for the userspace implementation of all I/O front-end device emulation, the **Linux host** responsible for the I/O backend (via normal system calls) and the **KVM kernel** module responsible to multiplex the CPU and MMU of the processor.
>QEMU: It's a complete machine simulator with support for cross-architectural binary translation of the CPU, and a complete set of I/O device models.
- Unlike Xen or VMware Workstation, KVM was designed from the ground up assuming the existence of hardware support for virtualization.

>[!question] Why we need to use QEMU instead of implementing everything in KVM and Linux kernel host?
>The decision to split KVM and QEMU comes down to a core operating system design principle: **keeping the host kernel as small, fast, and secure as possible**. Emulating hundreds of hardware devices requires millions of lines of complex code that must parse potentially malicious input from the guest virtual machine. If all this emulation code lived inside the Linux kernel, a single bug in a virtual USB or floppy drive could trigger a kernel panic, crashing the entire physical server and all its running VMs. By placing this emulation in QEMU's user-space process, a fatal bug only crashes that specific virtual machine, leaving the host and other VMs completely safe. Furthermore, forcing the kernel to manage every legacy and modern hardware device would make it massively bloated and nearly impossible to maintain. Device emulation simply doesn't require the highest hardware privileges, as writing to a virtual hard drive is often just writing to a standard file on the host. Keeping QEMU in user space also allows administrators to patch or upgrade virtual device support without needing to reboot the entire physical host machine. Ultimately, KVM handles only what requires absolute privilege—CPU and memory virtualization—while offloading the dangerous and complex hardware emulation to QEMU.
## Challenges in Leveraging VT-x
The Popek and Goldberg's three core attributes of a VM as follows
>[!Note] Equivalence
>A KVM virtual machine should be able to run any x86 operating system (32-bit or 64-bit) and all of its applications without any modifications. KVM must provide sufficient compatibility at the hardware level such that users can choose their guest operating system kernel and distribution.

>[!Note] Safety
>KVM virtualizes all resources visible to the virtual machine, including CPU, physical memory, I/O busses and devices, and BIOS firmware.

>[!Note] Performance
>KVM should be sufficiently fast to run production workloads. However, KVM’s explicit design of a **type-2** architecture implies that resource management and scheduling decisions were left as part of the host Linux kernel.



## The KVM kernel Module

The kernel module only handles the basic CPU and platform emulation issues. This includes
- CPU emulation
- memory managemt
- MMU virtualization
- Interrupt virtualization
- chipset emulation
But **exclude** I/O device emulation. 

Given that KVM was designed only for processors that follow the Popek/Goldberg prin ciples, the design is in theory straightforward
1. configure the hardware appropriately
2. let the virtual machine execute directly on the hardware
3. upon the first trap or interrupt, the hypervisor then regains control, and “just” emulates the trapping instruction according to the semantic.

Figure 4.3 illustrates the key steps involved in the trap handling logic of KVM, from the original `#vmexit` until the `vmresume` instruction returns to non-root mode.
1. `#vmexit`: KVM first saves all the vcpu state in memory
2. KVM then performs a first-level dispatch based on `vmcs.exit_reason`.
3. Most of these handlers are straightforward. In particular, some common code paths rely exclusively on VMCS fields to determine the necessary emulation steps to perform. Depending on the situation, KVM may:
	1. **emulate** the semantics of that instruction and **increment the instruction pointer** to the start of the next instruction;
	2. **determine that a fault or interrupt must be forwarded to the guest environment**. Execution will then resume at the instruction specified by the guest’s interrupt descriptor table;
	3. **changethe underlying environment** and **re-try the execution** (e.g. EPT violation);
	4. **do nothing** (at least to the virtual machine state). (e.g. when an external interrupt occurs, which is handled by the underlying host operating system. Non root execution will eventually resume where it left off.).
>[!Note] Conclusion of handling `#vmexit`
> There are two main categories of events that cause a `#vmexit`. The first category is **Guest-driven events**, where the Guest OS encounters an instruction or environment state it cannot handle in non-root mode. In this case, KVM must take control to resolve the issue by emulating the instruction, forwarding a fault/interrupt back to the guest, or altering the underlying environment (like page tables) and retrying. The second category is **Host-driven events**, specifically external hardware interrupts. In this case, the `#vmexit` occurs simply so the underlying Host OS can service physical hardware, after which the guest execution resumes unchanged.

>[!Note] Deeper look at emulation
>Because the VMCS sometimes lacks sufficient detail to handle a `#vmexit`, the hypervisor must include a general-purpose decoder and emulator to manually process the instruction that triggered the exit. The key steps of the emulator are
>1. **fetch** the instruction from guest virtual memory.
>2. **decode** the instruction, extracting its operator and operands.
>3. **verify** whether the instruction can execute given the current state of the virtual CPU
>4. **read** any memory read-operands from memory
>5. **emulate** the decoded instruction
>6. **write** any memory write-operands back to the guest virtual machine
>7. **update** guest registers and the instruction pointer as needed

Clearly, these steps are complex, expensive, and full of corner cases and possible exception conditions.

![[General-purpse trap-and-emulate flowchart of KVM.png]]

## The Role of the Host OS

Figure 4.4 shows the core KVM VM execution loop, shown for one virtual CPU.
![[KVM VM execution loop.png]]
- User mode: VMX **Root** Mode + Ring 3 (QEMU)
- Kernel mode: VMX **Root** Mode + Ring 0 (KVM module)
- Non-root: VMX **Non-root** mode
#### The loop in user mode
1. enters the KVM kernel module via an ioctl to the character device /dev/kvm
2. the KVM kernel module then executes the guest code until
	- the guest initiates I/O using an I/O instruction or memory-mapped I/O
	- the host receives an external I/O or timer interrupt
3. the QEMU device emulator then emulates the initiated I/O (if required); and
4. in the case of external I/O or timer interrupt, the outer loop may simply return back to the KVMkernel module by using another ioctl(/dev/kvm) without further side-effects.
#### The loop in KVM kernel module
1. restores the current state of the virtual CPU;
2. enters non-root mode using the `vmresume` instruction. At that point, the virtual machine executes in that mode until the next `#vmexit`;
3. handles the `#vmexit` according to the `exit reason`;
4. if the guest issued a programmed IO operation or a memory-mapped IO instruction), break the loop and return to userspace; and
5. if the `#vmexit` was caused by an external event, break the loop and return to userspace.
# Performance Considerations
The design of VT-x is centered around the duplication of architectural state between **root** and **non root** modes, and the ability to atomically transition between them.
- `vmresume`: transitions back to non-root mode and loads then VMCS state into the current processor state.
- `#vmexit`: stores the entire state of the virtual CPU into the VMCS state.

![[Hardware cost of VT-x instructions and vmexit.png]]

Table 4.3 shows that certain instructions, (e.g., `vmread`) went from being implemented in microcode to being integrated directly into the pipeline. Nevertheless, all the atomic transitions (`vmresume` or `#vmexit`) remain very expensive.