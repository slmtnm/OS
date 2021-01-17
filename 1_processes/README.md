## What is a process?

*Application* -- program on disk, flash memory ... (static entity).

*Process* -- state of a program when executing loaded in memory (active entity)

If the same program launched more than multiple processes will be created, but will
have very different state.
It doesn't mean necesserily that process running (may be waiting for input or waiting for another process executing on CPU).

## What does a process look like?
Process encapsulates all of this data for running applications:

* Text (program instruction set)
* Data (global and static variables)
* Heap (dynamic allocated memory)
* Stack (local variables)

All of these segments must have unique address. OS uses an abstraction called 'address space' to encapsulate all of the
process state.

*Address space* -- range of addresses from some $V_0$ to $V_{max}$. Different parts of process state (code, data...) will
appear in different region of address space.

* Code and data segments exists when process first loads.
* Heap dynamically get expanded and shrinked during execution.
* Stack is a LIFO.

## Process address space.

Addresses that are used by a process are *virtual* because they don't correspond to actual locations in physical memory.
Instead, *memory management hardware* and *OS components* (like page tables) maintain a mapping between virtual addresses
and physical addresses.

## Process address space and memory management.

Process usually doesn't use whole address space (from $V_0$ to $V_{max}$) and we may simply not 
have enough physical memory to store all state. For ex. if virtual addresses are 32 bits, 
virtual address space is 4GB. So, OS doesn't map all address space to physical memory and
dynamically desides which portion of address space is mapped and where. So different parts of
same address space that are mapped may not layout contigiously in physical memory.

## PCB

Process control block -- instance of specific structure and maintained by OS:

* PCB created when process is created
* Certain fields are updated when process state changes. Some fields change too frequently.

It contains:

* Process state (Running, Ready, Waiting)
* Process number
* Program counter
* Values of registers
* Memory limits
* List of open files
* Priority (for scheduling)
* Signal mask
* CPU scheduling info (how long process running, etc.)
* and many more ...

## Context Switch

**Context switch** is mechanism that used by OS of switching execution from context of one process
to the context of another process.

It can be expansive for 2 reasons:

* *Direct costs*: number of cycles for load/store context from/to PCB (load/store register values in CPU).
* *Indirect costs*: cold cache (cache misses) because previous process polluted cache.

## Process state and transitions

Processes can be:

* Ready -- not executing on CPU, but ready for this
* Running -- currently executing on CPU
* Waiting -- blocked while waiting some event, typically I/O operation

Process that is 'Ready', can be transited to 'Running' state by scheduler.

Process that is 'Running', can be transited to 'Ready' by scheduler, or to 'Waiting' when
issuing I/O operation.

Process that is 'Waiting', can transited to 'Ready' when completed I/O operation (for ex.
got data from disk, or writed on console).

New process is always in 'Ready' state. Process can terminate only from 'Running' state.

## Process creation

Processes in most operating systems represented by tree, where tree root is 'sched' process with PID 0
(typically) that is responsible for paging. Process 'init' is special and has PID 1. All
other processes created either by 'init' or it's descendants.

For process creation there are several system calls:

* *fork* -- creates new process with same PCB as it's parent (=> same program counter, stack
pointer, execution context, etc.). But obiously new process has new address space (mappings from same virtual
addresses to new physical addresses).
* *exec* -- it replace's PCB of process that invoked it with completly new PCB, and
take program name and program arguments as argument, loads program and start it.

Typical combination is fork + exec.

System call 'clone' similar to 'fork', but provides more precise control over
what pieces of execution context are shared between calling process and child process.

For example, with 'clone' system call, caller can control whether or not the two
processes share the virtual address space (in this case new thread will be created, but not
whole process), table of descriptors and table of signal handlers. 

This system call also allow the new child process to be placed in separate namespace 
(read about namespaces in linux).
