## Process vs Thread.

Single-threaded process is represented by it's *address space* (code, data, files...) and
*execution context* (values of registers, stack pointer, program counter etc...). OS encapsulate all
these information in *process control block (PCB)*.

**Threads** represent multiple independent execution contexts, but same address space. So they will share
code, data, files... But they will execute different instructions, access different portions of that
address space, operate on different portions of the input etc. 

PCB for multi-threaded process will contain one address space mappings and multiple instances of execution context -- one per thread.

## Benefits of multithreading

1. Several threads, that belongs to once process, can execute same code (but different instructions) *simultanously* on different CPU cores.
For example, they can process different parts of input at one time.
So, we achieve speed up by **parallelization**.

2. Multiple threads can execute different portions of program that correspond to specific functions. In large web application, threads can operate on different customer requests. We can differentiate how we manage those threads based on for what tasks they respond. For example, we can give higher priority to those threads that handle more important tasks or more important customers. That called **specialization**. Another benefit of specialization comes from fact that performance depends on how much state can be present in processor cache. If thread executes just one portion of program (one task),
more of that state will be actually present in cache. So, **cache is hot**.

3. More memory efficient than multi-process alternative (see below)

Why not multi-process? 
1. Because we have to allocate address space for every single process, 
so multi-threaded applications are more memory efficient and not require as many swaps from disk. 
2. Because synchronization between processes with IPC is more costly. For multi-threaded apps, synchronization is achieved
with variables, that stored in shared address space.

## Are threads useful on a single CPU?

Yes, when thread do IO operation, and it waiting more than 2 context switches. In this case,
we could perform context switch to other thread, do some useful work, and then switch back.

That related to multiple process too, but switching context from one process to another is much more costly
because of loading new virtual-physical address mappings.
