## History
- Unix was developed in 1969 by Ken Thompson and Dennis Ritchie and others at Bell Lab
	-  Unix version 0 runs on PDP-7 
	-  Unix was initially written in assembly; rewritten in C in 1973
- Bell Labs released the ==first Unix (V6) in 1975==, which researchers at universities began using 
	- Ken Thompson and graduate students at UC Berkeley developed the Berkeley Software Distribution ==(BSD) based on V6 in 1977==
- Unix diverges into ==two main versions: BSD and System V== (by AT&T) — two main branches for later Unix-based OS implementations

---
## System Architecture
![[Pasted image 20251216184705.png]]
-  **Applications** : are written and compiled by programmers into binaries 
- **Library routines** : provide pre-compiled binaries (e.g. header files like “stdio.h”) 
- **Shell** : exposes an interactive interface for users to issue commands to run applications
> The shell and library routines are shown with gaps to indicate that they do not always invoke system calls; many of their operations are performed entirely in user space.

---
## Philosophy 
- Emphasizes building **simple**, **modular**, and **extensible** code that can be maintained and repurposed easily by developers other than its creators! • 
	- Write programs that do one thing and do it well 
	- Write programs that work together 
	- Write programs that handle text streams 
> 因為有text streams，才能讓各個程式之間彼此能協作

---
## User ans kernel split
![[Pasted image 20251216191552.png]]
Making a ==system call cause a trap== from the user mode/space to the kernel mode/ space

--- 
## Version
![[Pasted image 20251216192741.png]]
