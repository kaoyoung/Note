# Virtualization
>[!definition] The definition of virtualization
>Virtualization is the **application** of the **layering** principle through **enforced modularity**, whereby the exposed virtual resource is identical to the underlying physical resource being virtualized.

According to the definition above, virtualization is based on two fundamental principles: layering and enforced modularity.

>[!concept] The concept of layering
>It is the presentation of a single abstraction, realized by adding a level of indirection, when (i) the indirection relies on a single lower layer and (ii) uses a well-defined namespace to expose the abstraction.

>Indirection: In computer science, **indirection** means _not_ interacting with something directly, but through an intermediary.

>[!concept] The concept of enforced modularity
>Compared to layering, it additionally guarantees that the clients of the layer cann't bypass the abstraction layer.

>[!note] Conclusion of Virtualization
>Virtualization utilizes layering and enfored modularity to separate the physical resouces from virtaul operations. Enfored modularity hides the underlying physical resources and forces virtual operations to interact indirectly via an interface. Crucially, this interface exposes a virtual resource that is identical to the underlying physical one, allowing software to run exactly as it would on bare metal. The concept of virtualization is quite similar to network layering.

Virtualization is always achieved by using and combining three simple techniques
![[Virtualization_method.png]]
- Multiplexing : Exposes a resource among multiple virtual entities. ere are two types of multiplex ing, in space and in time.
- Aggregation : It takes multiple physical resources and makes them appear as a single abstraction.
- Emulation : It relies on a level of indirection in software to expose a virtual resource or device that corresponds to a physical device, even if it is not present in the current computer system.

>[!note]
>These categories may be viewed as a mapping from physical resources to abstactions:
>- Multiplexing : One-to-many mapping.
>- Aggregation : Many-to-one mapping.
>- Emulation : One-to-one mapping.
>
>Note that many-to-many mappings can be constructed by combining these three methods.

---
# Virtual Machine

>[!definition] The definition of virtual machine 
>A virtualmachine is an abstraction of a complete compute environment through the combined virtualization of the processor, memory, and I/O components of a computer.

>From the definition, it seems we only care about processor, memmory, and I/O component. This is what Von Neumann architecture cares about.

![[Von_Newman_architecture.png]]
source : [Difference between Von Neumann and Harvard Architecture](https://www.geeksforgeeks.org/computer-organization-architecture/difference-between-von-neumann-and-harvard-architecture/)

![[virtual_machine_classification.png]]

---
# Hypervisor

>[!definition] The relation between a virtual machine and hypervisor (Popek and Goldberg)
>A virtual machine is taken to be an efficient,isolated, and duplicate of the real machine. We explain these notions through the idea of a virtual machine monitor (VMM). As a piece of software, a VMM has three essential characteristics. First, the VMM provides an environment for programs which is essentially identical with the original machine; second, programs running in this environment show at worst only minor decreases in speed; and last, the VMM is in complete control of system resources

---
# Type-1 and Type-2 Hypervisor

- type-1: the VMM runs on a bare machine
- type-2: the VMM runs on an extended host, under the host operating system.

---
# A sketch hypervisor: multiplexing and emulation
![[VM_basic_architexture.png]]
- NIC: network interfalce



---
# Approaches to Virtualization and Paravirtualization

- Full (software) virtualization: This refers to hypervisors designed to maximize hardware com patibility, and in particular run unmodified operating systems, on architectures lacking the full support for it. The hypervisor must (at least some of the time) translate guest instruction sequences before execution.
- Hardware Virutalization (HVM): This refers to hypervisors built for architectures that provide architectural support for virtualization, which includes **all recent processors**. HVM hypervisors rely exclusively on **direct execution** to execute virtual machine instructions.

>Direct Execution: The hypervisor sets up the hardware environment, but then lets the virtual machine instructions execute directly on the processor. As these instruction sequences must operate within the abstraction of the virtual machine, their execution causes traps, which must be emulated by the hypervisor.

>上文的 "their execution causes traps" 是強調虛擬機直接在 CPU 上運作的這個行為機制本身，就會不斷產生 traps 讓 Hypervisor 來接手處理，而不是每個動作都會造成 trap 。

- Paravirtualization:  In its contemporary use on architectures with full virtualization support, paravirtualization is still used to augment the HVM through platform-specific extensions often implemented in device drivers


---
