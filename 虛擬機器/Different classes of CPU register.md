### Ref : [Different Classes of CPU Registers](https://www.geeksforgeeks.org/computer-organization-architecture/different-classes-of-cpu-registers/)

# What do cpu registers do
CPU registers play the role in 
- data manipulation
- memory addressing
- tracking processor status
They work in coordinate with cpu's memory, in order to enhance the speed of storing and retrieving data.
# Different type of cpu registers
- Accumulator : Store data taken from memory.
- Memory Address Register (MAR) : Hold the address of the location to be accessed from memory.
- Memory Data Register (MDR) : Contain data to be written into or to be read out from the address location. MDR contains "data to be read out", it means the data has already been fetched from the RAM, and is now sitting in the MDR waiting to be read out by the rest of the CPU.
- General Purpose Register : Store temporary data during any ongoing operation.
- Program Counter : PC points to the address of the next instruction to be fetched from the main memory when the previous instruction has been successfully completed.
- Instruction Register : Hold the instruction which is just about to be executed.
- Stack Pointer : Points to the top of the stack.
- Flag Register (Status Register) : Indicate the status of the CPU or the outcome of various operations such as
	- Zero flag: Indicates if the result of an operation was exactly zero. It's used to check for equality.
	- Carry flag: Indicates if the operation caused an "overflow" for an unsigned number.
	- Sign flag: Indicates if the result of a math operation is positive or negative.
	- Overflow flag: Similar to the Carry flag, but specfically for signed number.
	- Interrupt Enable Flag: whether or not to listen to onuside interruptions.
