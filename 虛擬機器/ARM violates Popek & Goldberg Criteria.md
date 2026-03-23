>[!question] 
>What's Popek and Goldberg Criteria

>For a perfect hardware architecture, sensitive instructions (instructions that affect the core system) must also be previliege instruction (requiring the highest permission).

>[!question] 
>What's trap-and -emulate approach

>When the guest OS tries to execute a priviledged instruction, the hypervisors traps that instruction and emulate that instruction.

>[!question]
>Why in the book, it said "The ARM architecture was not virtualizable. It contained virtualization-sensitive, unprivileged instructions, which violated the Popek and Goldberg criteria for strict virtualization. This ruled out the traditional trap-and-emulate approach to virtualization.", but we still can do hypervisor

>There are two main strategies:
>1. Software workaround
	- Paravirtualization: modified the source code for the guest OS, so it proactively sent request to the hypervisor when doing sensitive tests.
	- Binary translation: check the guest OS's instruction line by line. If it spotted those non-trapping sensitive instruction, it dynamically intercepted and replaced them with safe code.
>2. Hardware
	Modern ARM has four distinct levels: EL0 (noraml apps), EL1 (guest os), EL2 (hypervisor), and EL3 (lower-level secure monitor).

