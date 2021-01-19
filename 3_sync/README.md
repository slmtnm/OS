### Thread sync

What do we need to support threads:

* thread data structure -- to identify threads, keep track of resource usage, ...
* mechanisms to create and manage threads
* mechanism to sagely coordinate among threads running concurrently in same address space

*Data race* -- problem when multiple threads access same data at same time (concurrent writes, or concurrent write and read).

Concurrecy control:

* mutual exclusion -- exclusive access to only one thread a time
  * implemented as **mutex**
* waiting on other threads
  * also need to specify what are we waiting for -- specific *condition* before proceeding
  * implemented as **condition variable**

## Mutual exclusion

OS and threading libraries support abstract called *mutex* -- lock that should be used when
accessing data that is shared among threads.

* On **lock** (aquire) mutex, thread has exclusive access to resource
* Other threads that locks same mutex, will be blocked on this lock operation until mutex owner releases it

Mutex is a data structure that has 

* locked flag
* owner
* list of blocked threads

**Critical section** -- portion of the code, protected by mutex.

## Producer/Consumer problem

What if processing you wish to perform with mutual exclusion needs to occur only under 
certain conditions?

For ex., there are some **producer** threads that insert in linked list, and one **consumer** thread,
that prints this list and clears it. We want to run consumer's work only under *condition*: list is full.

This can be done with only mutex contruct: producer acquire a lock and only then perform condition check,
and if condition is false, releases mutex and tries to lock mutex again, ...

But this approach is wasteful, and better to tell consumer where the list is actually full.

## Condition variables

**Condition variable** -- used in conjunction with mutexes to control behaviour of concurrent threads.
With cond. var., consumer will **wait** until list is full. Producer, after insertion will check if list is full, and if it is,
send **signal** to consumer.
  
Both wait and signal operations used with special condition variable as argument, and both of them
are used **inside** critical section (with aquired mutex). On wait operation, consumer must also release
captured mutex, and capture it back once signal is received. 

Summarizing, cond. var. must support following API:

* wait(mutex, cond)
* signal(cond) -- to send signal only for one waiting consumer
* broadcast(cond) -- to send signals for all waiting consumers (if there more than 1 consumer)

## Reader/Writer problem

We threads of 2 kind: 
 
* that are only read shared state
* that are modifying shared state

Problem: we should have 0 or more readers at a time, but only 0 or 1 writer at a time AND if there are readers that accessing shared state, 
then no writers should be that modify it (see def. of *data race*).

Problem cannot be solved just with mutex, because it is not allowed multiple readers to access shared state, but only one (too restrictive).

We can solve it by introducing integer counter, that is equal -1 if there are 1 writer, equal 0 if shared state is free, or N > 0 where N is number of readers.
currently accessing shared state. 

We will control this counter with mutex, and additionaly have 2 cond. var.: *read phase* and *write phase*.

![alt text](rw.png)

## Deadlocks

**Deadlock** -- situation in which concurrent threads waiting on each other to complete,
however none of them do, because each waits on the other one.

![alt text](deadlock.png)

Deadlock appears when two threads locks two different mutexes in different order.

**Wait graph** - oriented graph, where 

* verticies -- threads
* edge from t1 to t2 if t1 waiting on a resource that t2 has

How to avoid:

* maintain lock order (in ex. first m_A, then m_B in both threads)
  * (+) will prevent cycles in wait graph
  * (-) hard to follow this principle in large programs (easy to make mistake)
* be able to detect deadlock situation in runtime and have recover mechanism (rollback execution) that is used when deadlock detected
  * (-) hard to perform rollback, if have inputs or outputs to external sources
* apply Ostrich algorithm -- do nothing and hope there are no deadlocks (if there are, reboot program)

## Spinlocks

Spinlocks are one of the most basic sync. construct. Spinlocks are like mutex, has
lock() and unlock(). But when performing lock operation, if lock is busy, 
thread isn't blocked (like when using mutex), but instead **spinning**: 
running on CPU and repeatedly check if lock is free (burning CPU cycle). Thread 
is suspended from CPU only if it preempted (for ex. if timeslice expired).

## Semaphores

Semaphores are common sync construct in OS kernels. It's like traffic light: STOP or GO. Similar
to a mutex, but more general. 

Semaphore represented by integer value. On init, it is assigned a max value (positive int).

On try (wait, P, proberen), if semaphore value is non-zero => decrement and proceed. Otherwise thread
will blocked. 

If semaphore initialized with 1, it is binary semaphore (mutex).

On exit (post, V, verhogen) from critical section, semaphore will be incremented.

# POSIX API

```c
sem_t sem;
sem_init(sem_t *sem, int pshared, int count)
sem_wait(sem_t *sem);
sem_post(sem_t *sem);
```

*pshared* is flag that determines if semaphore is shared across threads within 
a single process, or among multiple processes.

## Reader/Writer locks


