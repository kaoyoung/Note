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

---
# Virtual Machine
>[!definition] The definition of virtual machine 
>A virtualmachine is an abstraction of a complete compute environment through the combined virtualization of the processor, memory, and I/O components of a computer.

>From the definition, it seems we only care about processor, memmory, and I/O component. This is what von-newman architecture cares about.

![[Von_Newman_architecture.png]]
source : [Difference between Von Neumann and Harvard Architecture](https://www.geeksforgeeks.org/computer-organization-architecture/difference-between-von-neumann-and-harvard-architecture/)



---
# Hypervisor


---
# Type-1 and Type-2 Hypervisor


---
# Approaches to Virtualization and Paravirtualization




---
