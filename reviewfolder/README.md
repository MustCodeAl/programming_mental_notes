


The Central Processing Unit (CPU) is a tiny state machine continually operating on main memory or Random Access Memory (RAM). RAM stores both code and data.

1. A CPU *fetches* instructions currenty addressed by Instruction Pointer(IP) aka Progran Counter(PC) reguster, 
2. *decodes* the meaning of the fetched instruction which must include an opcode(unique encoding for operation) and may optionally operands(arguments for operatiton)
and/or a prefix (behavioral modifier)., 
3. *executes* instructions from RAM (code)  Internally, this means a Control Unit (CU) passes instruction-specific signals to functional units like the Arithmetic Logic Unit (ALU),
   the functional unit which performs mathematical operations on register values., then writes back results (data) if applicable.

A program must be loaded into memory, creating a process, before it can run. 
That involves mapping it's executable code into RAM and setting up 3 special memory locations:

Static memory - stores global variables and constants.
Stack memory - stores function frames, including local variables.
Heap memory - stores data shared between functions and threads.

Modern CPUs rely on complex optimizations, like instruction pipelining10 and speculative execution11, to speed up the instruction cycle.

Think of a cpu as an implementaation of an isa (instruction set architecture) - A standard ISA defines its basic elements such as data types, register values, various hardware supports, I/O etc. and they all make up the lowest-level language of computing which is the Machine Language Instructions.
 
 Instructions are comprised of instruction code (aka operation code, in short opcode or p-code) which are 
 directly executed by the CPU. An opcode can either have operand(s) or no operand. 
