## Overview

**CPU scheduler** -- component of OS that decides how and when processes and their threads access shared CPU.
We will use term *task* -- process or thread.

Scheduler schedules tasks running *user-level processes/threads* as well as *kernel-level threads*.

Scheduler chooses one of the task in *ready queue* to run on CPU.
![image](scheduling.png)

Scheduler runs when

* CPU becomes idle (for ex. task perform IO op.)
* New task becomes ready (to check if new task has higher priority, and if then, interrupt current task)
* Timeslice of current task expires

Once scheduler selects a thread to be scheduled, thread is *dispatched* on CPU: context switch, enter user mode, set PC and go!

So, *scheduling* is choosing task from ready queue. But which task should be selected depends on scheduling policy/algorithm. How task is retrieved from ready queue depends on *runqueue* structure.

Usualy type of runqueue and scheduling algorithm are tightly coupled.

### Scheduling algorithms

Metrics (that we need to compare algorithms): 

* Throughput ($\frac{\text{total number of tasks}}{\text{total amount of time to complete them}}$)
* Avg. job completion time
* Avg. job wait time
* CPU utilization

## Run-to-completion scheduling

As soon as a task is assigned CPU, it will run until it finishes or until it completes.

Assumptions:

* We have group of task/jobs that we need to schedule (can assume that they are arrived at same time)
* Known their execution times
* No preemtion (that is what means "to-completion")
* Have a single CPU

# First-come first-serve algorithm (FCFS)

* Schedules tasks in order of arrival (regarding their execution time, load of CPU etc.)
* Runqueue structure: simple FIFO queue 

# Shortest Job First (SJF)

* Schedules tasks in order of their execution time
* Runqueue structure: 
  * FIFO queue, ordered by execution time
  * or tree with key=execution time

## Preemptive scheduling

Assumptions:

* Task not arrive at same time

# Shortest Job First (SJF)

* When new task enter new queue (or current task completes), scheduler is being invoked, and compare
execution times and decides if it should preempt current task (if new task is shorter).
* Execution time of a task is unknowned and depends on too much many factors,
so we use some heuristics based on history (how long did a task run last N times).

# Priority scheduling

* Tasks have different priority levels
* All same as SJF, but for deciding "should we preempt current task" we use task's priority values.
