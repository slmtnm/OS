### Inter process communication

**IPC** -- OS-supported mechanisms for interation among processes (coordination and communication)

Types of IPC:

* message passing (e.g. sockets, pipes, message queue)
* memory-based IPC (shared memory, mem. mapped files)
* higher-level semantics: files, RPC ("higher" because these methods will determine protocol of interaction)
* synchronization primitives

## Message based IPC

Processes create messages and then send/recv them. OS responsible for creating and maintaining a channel
(buffer, FIFO queue, ...) and provides interface to processes - a *port*:

* processes send/write message to a port
* processes recv/read messages from a port

Kernel required to 

* establish communication
* perform each IPC operation
  * send: system call + data copy
  * recv: system call + data copy

* (-) many user/kernel crossings and data copies.
* (+) simple: kernel does channel management and synchronization, handles errors

# Forms

* pipes:
  * carry raw byte stream between 2 processes
* message queues:
  * carry "messages" among processes
  * OS management includes priorities, scheduling of msg delivery...
  * APIs: SysV and POSIX

Socket is an example of message queue. They have notion of *port*.
Sockets:

* send(), recv() -- pass message buffers
* socket() -- create kernel level socket buffer
* associate necessary kernel-level processing (TCP/IP, ...)
* if different machines, channel is between process and network interface
* if same machine, bypass full protocol stack

## Shared memory IPC

Processes *read* and *write* to shared memory region. Os establish shared channel between processes.

1. physical pages mapped into virtual address space
2. VA(p1) and VA(p2) map to the same physical address, VA(p1) != VA(p2)
3. physical memory doesn't need to be contigious

* (+) system calls only for setup 
* (+) data copies potentially reduced (not fully eliminated)
* (-) explicit synchronization by process (complexity)
* (-) communication protocol determined by processes (complexity)
* (-) shared buffer management

Shared memory APIs: SysV, POSIX, memory mapped files, Android ashmem.

## Copy (messages) vs Map (shared memory)

Goal of both: transfer data from one into target address space

Messages perform *copy* of data:

* CPU cycles to copy data to/from channel (port) and user/kernel crossings

Shared memory perform *map*:

* CPU cycles to map memory into address space (costly)
* CPU to copy data to channel

Set up once use many times -> good payoff. Can perform well even for 1-time use, if the data is big.
Copy usually used with small data.

## POSIX shared memory API

1. shm_open()
  * returns file descriptor
  * file not on disk, but in tmpfs
  * rest of the operation uses this file descriptor
2. mmap() and unmmap()
  * mapping virtual => physical addresses
3. shm_close()
  * can use regular close()
  * remove file descriptor from process, but memory will still exist until reboot
4. shm_unlink()
  * destroy shared memory region

## Shared Memory and synchronization

Memory in shared memory region can be accessed concurrently by different processes.
Synchronization methods:

1. mechanism supported by process threding library (pthreads)
2. OS-supported IPC for synchronization

Either method must coordinate:

* number of concurrent accesses to shared segment
* when data is available and ready for consumption

# PThreads sync

There is a flag PTHREAD_PROCESS_SHARED in pthread_mutexattr_t (or pthread_condattr_t), that determines if 
synchronization construct is private to process or shared among processes. 

**Synchronization data structures must be allocated in shared region.**

## Message queues and synchonization

Syncronization (for shared memory) may be achieved also by using message queues and semaphores.
With message queues we can implement "mutual exclusion" via send/recv.

Example protocol:

* p1 writes data to shmem, send "ready" to queue
* p2 receives msg, reads data and sends "ok" msg back

Semaphores:

* binary semaphore (mutex)
  * if value == 0 => stop/blocked
  * if value == 1 => decrement (lock) and go/proceed
