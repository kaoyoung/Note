# The Model

The model has the following features
1. Two execution levels: supervisor mode and user mode
2. Virtual memory is implemented via segmentation.
3. Physical memory is contiguous, starting at address 0, amount of physical memory (SZ).
4. The processor's system state, called the **processor status word (PSW)** consists of the tuple $(M, B, L, PC)$.
	1. The execution level $M = \{s,u\}$;
	2. The segment register $(B,L)$; and
	3. the current program counter (PC), a virtual address.
5. The trap architecture has provisions to first save the content of the PSW to a well-known location in memory (MEM\[0\]), and then load into the PSW the values from another well known location in memory (MEM\[1\]). 
6. The ISA includes at least one instruction or instruction sequence that can load into the hardware PSW the tuple $(M,B,L,PC)$ from a location in virtual memory.
7. I/O and interrupts are ignored to simplify the discussion.

If you are interesting in why we use tuple in feature 4, you may see [[組合數學和集合論基礎物件和操做]] for reference. 

>[!question] 為啥第四點用 processor status word 而不是 processor status set
>首先 word 代表一個有意義的完整概念，正如這邊的 processor status word 代表一個 processor status ，word 本身是由 letter 按照一定順序組合而成，跟這邊的 tuple 依樣要符合特定格式。不選 set 是因為 set 沒有順序的概念 (可參考 [[組合數學和集合論基礎物件和操做]] ) ，而且已有多個地方使用 set ，例如
>- **Instruction Set（指令集）：** CPU 支援的所有指令的總和。    
>- **Cache Set（快取集合）：** 在 Set-associative Cache（集合關聯式快取）中，用來分組記憶體區塊的單位。    
>- **Working Set（工作集）：** 作業系統中，一個行程在特定時間內頻繁使用的記憶體分頁集合。

>[!note] 第四點帶來的啟發
>系統狀態只關心，你在哪個 mode (user mode、supervisor mode)，記憶體的位置，運算的方式 (program counter 指的內容)。

>[!question] What's the difference between feature 5 and feature 6
> Feature 5 talks about trap architecture that changs user mode to supervisor mode. In contrast, feature 6 covers mode switching in both directions—between user mode and supervisor mode—not just transitioning from user to supervisor.

>[!question] Popek and Goldberg's questions
>Given a computer that meets this basic architectural model, under which precise conditions can a VMM be constructed, so that the VMM: 
>1. can execute one or more virtual machines; 
>2. is in complete control of the machine at all times; . 
>3. supports arbitrary, unmodified, and potentially malicious operating systems designed for that same architecture; and
>4. be efficient to show at worst a small decrease in speed?

We can conclude the above questions to three critical criteria for VMM
1. Equivalence (q1, q3):  The **virtual machine** is essentially identical to the underlying processor, i.e., a du plicate of the computer architecture.
2. Safety (q2): The **VMM** must be in complete control of the hardware at all times, without making any assumptions about the software running inside the virtual machine. A **virtual machine** is isolated from the underlying hardware and operates as if it were running on a distinct computer.
3. Performance (q4): The efficiency requirement implies that the execution speed of the program in a virtualized environment is at worst a **minor decrease** over the execution time when run directly on the underlying hardware.

>[!question] 為何不把 Popek and Goldberg's questions 第二點跟第三點順序對調
>問題的第一點跟第三點都是在討論 Equivalence ，那為何不把第二點跟第三點順序對調，讓第一點跟第三點放在隔壁好拿來一起討論。Popek and Goldberg's questions 是從系統架構師的角度切入，層層遞進來建構這問題。第一點先確定 VMM 可以跑至少一個 virtual machine； 第二點確定 VMM 可以掌控整個機器，這時才會出現處理 arbitrary, unmodified, and potentially malicious os 的問題，如果不能掌控整個機器你很難處理 potentially malicious os 的問題；確定 VMM 可以處理 potentially malicious os 後才切入， VMM 要能處理 arbitrary, unmodified, and potentially malicious os 。前三點順序總結來說是，先確定 VMM 可以運行 VM，在說它掌管整台機器 (因此能處理 potentially malicious os ，支援各種 os)，之後說能支援 arbitrary, unmodified, and potentially malicious os ，最後提一嘴 efficiency 。

---
# The Theorem

>[!theorem] Theorem 1
>For any conventional third-generation computer, a virtual machine monitor may be constructed if the set of sensitvie instructinos for that computer is a subset of the set of privileged instructinos.

This theorem is mainly considering the instructions, so it's important that we categorize the instructions of the ISA (instruction set architecture).
- control-sensitive : Instructions that can update the system state.
- behavior-sensitive : Instructions that its semantics depend on the actual values set in the system state.
- innocuous instruciton : Otherwise
An instruction is **privileged** if it can only executed in supervisor mode and cause a trap when attempted from user mode. This theorem mainly talks us
$$
\{ \text{control-sensitive} \} \cup \{ \text{behaviour-sensitive} \} \subseteq \{ \text{privileged} \}
$$ 
Let me give you a proof sketch about theorem 1.

![[Construction of the Popek,Goldberg VMM.png]]
source : Hardware and Software Support for Virtualization, chapter 2.2




