
# P3L1 - Scheduling

## What different approaches can an OS scheduler take to assign tasks?

* Assign tasks immediately -- FCFS (first come first serve)
* Assign simple tasks first -- maximize throughput SJF (shortest job first)
* Assign complex tasks first -- maximize utilization of CPU, devices, memory...

All three of these have their own algorithms. They can also be made more complex or simple with the introduction of **priority** and **pre-emption**.

## In general, what does a CPU scheduler do? What is its responsibilities?

A CPU scheduler decides **how** and **when** processes and their threads access **shared CPUs**.
It will check all tasks in the ready queue, adn then decide which one gets to run first.

Its general goal is to make sure the CPU never becomes idle. The goal is to check whether if any of the tasks in the queue are more important than the task that is running, and to interrupt it.

## What happens when the CPU selects a thread to be scheduled?

First, we perform a context switch.
Then, enter user mode to set the program counter to point at the next instruction of the newly selected thread.
Finally, the thread is dispatched on the CPU.

## How does a scheduler decide which task needs to be run first?

The scheduler will decide based on its scheduling policy/algorithm, which is heavily dependent on the data structure implemented to handle the runqueue. The design of the runqueue can **limit which scheduling algorithms** that we can use efficiently.

## Consider that we have a group of tasks that need to be scheduled. Assume that we only have a single CPU, and that no thread will be pre-empted--that its tasks will be seen to completion. What are some metrics we can use to compare the scheduling algorithms against one another?

* Throughput (how many tasks are completed in a given amount of time?)
* Average job completion time (how long does it take to finish all of the jobs?)
* Average job wait time (how long do jobs have to wait before executing to completion?)
* CPU utilization (formula: `[cpu_running_time / (cpu_running_time + context_switching_overheads)] * 100`)

## Define the first-come first-serve scheduling algorithm. How do we organize tasks in the runqueue? What does the scheduler need to know in order to schedule tasks?

The runqueue for FCFS scheduling is simply a FIFO queue. The FCFS scheduling algorithm takes the task that is at the top of the queue and runs it until completion.

## Define the throughput, average completion time, and average wait time for a FCFS algorithm analyzing 3 threads, T1=1s, T2=10s, and T3=1s. Assume that there is no pre-emption.

3/12s = 0.25 tasks per second is the throughput, because it can accomplish 3 tasks in 12 seconds.

(1+11+12) / 3 = 8 seconds for average completion time--this is taken from retrieving the completion second that each task reaches and dividing it by the number of tasks.

(0+1+11) / 3 = 4 seconds is the average wait time. This is how much time each task needs to wait on average before it is able to begin executing.

## Define the shortest job first scheduling algorithm. How do we organize tasks in the runqueue? What does the scheduler need to know in order to schedule tasks?

Here, we schedule tasks based on their **execution time**. The runqueue needs to be removed based on a very specific order, so the data structure we used to represent it will be **an ordered queue**. It can also be **a tree-like data structure** where the nodes in the tree are ordered based on execution time. The tree will need to be re-ordered when inserting new tasks, but the scheduler will simply need to select the left-most node in the tree.

## LECTURE Q: Define the throughput, average completion time, and average wait time for a SJF algorithm analyzing 3 threads, T1=1s, T2=10s, and T3=1s. Assume no pre-emption.

We know that the tasks will be executed in the order: T1, T3, and T2.

Throughput is 0.25 tasks/sec, determined from 3/12 = 0.25.

Average completion time is 5 seconds, from (1+2+12)/3 = 5.

Average wait time is only 1 second, from (0+1+2)/3.

## What is pre-emptive scheduling? Walk through an example of pre-emptive scheduling. Say that we have a single CPU, and 3 tasks: T1=1s, T2=10s, and T3=1s, and T2 arrives first. How does pre-emption improve task scheduling?

![Illustrating pre-emptinve scheduling with SJF](assets/P3L1/16.png)

Pre-emptive scheduling is when a task that is currently scheduled is interrupted, or pre-empted, by another task which arrives. Even thoughTask 2 arrives first, it will be interrupted so that T1 may execute.

## During pre-emptive scheduling with SJF, how can the CPU pre-empt if it does not know the execution time for any given tasks?

We can use heuristics based on history. Past execution time can predict the future. We can compute a **windowed average** for a given task, and use it to predict how long it might take for a task to take in the future.

## Say that we receive several tasks with different priority levels. How does priority scheduling influence pre-emptive schedulers compared with SJF? What kind of run-queue does pre-emptive priority schedulers use?

Priority based scheduling in a pre-emptive scheduler means that **tasks have different priority levels.** If we are allowed to pre-empt, then a task with a higher priority level than the one currently running which arrives while it is running may be allowed to pre-empt it.

We can have a different runqueue for each priority level, and each runqueue can be a priority queue. Alternatively, we can also maintain a tree ordered based on priority.

## What is one danger associated with preemptive scheduling? How do we address it?

A danger associated with pre-emptive scheduling is if **a low-priority task is stuck constantly in a runqueue.** This is a form of starvation. The low priority task may never be selected by the scheduler if more higher priority tasks constantly enter the queue and are selected first.

We can implement **priority aging**, which is where priority is determined **as a function of actual priority and time spent in the runqueue.** Therefore, priority isn't simply assigned, but it is calculated based on how long it has spent queued, meaning that a task that has waited in the queue for a longer amount of time will have its priority increased as a result.

## LECTURE Q: Assume that we receive 3 tasks: T1 arrives at time 5, T2 arrives at time 3, and T3 arrives at time 0. T1 requires 3 seconds to execute, T2 requites 4 seconds to execute, and T3 requires 4 seconds to execute. T1 has a priority of P1, T2 has a priority of P2, and T3 has a priority of P3. If P3 is lower priority than P2, and P2 is a lower priority than P1 (meaning P1 represents the highest priority), at what time does T1, T2, and T3 finish executing? Additionally, assume that T3 obtains a mutex lock that it does not unlock before being pre-empted by T2, and that T1 cannot continue without obtaining T3's lock. How does the OS handle the task afterwards? Assume that the OS scheduler uses a priority-based algorithm with pre-emption to schedule tasks.

![Illustrating the question without priority inversion.](assets/P3L1/1.png)

T3 will arrive and become executed until the arrival of T2. T2 is executed until the arrival of T1, where it will execute until completion. T2 will resume after, and T3 will resume after T1 and T2, which means that:

T1 will finish at 8 seconds
T2 will finish at 10 seconds
T3 will finish at 11 seconds

![Illustrating the question with priority inversion.](assets/P3L1/1.png)

For the second part of the question,
**Without priority boosting,** the order of execution will be T2, T3, T1--this is an illustration of **priority inversion.** Priority inversion can be handled with **priority boosting**, where we can temporarily increase the priority of whichever thread owns the mutex. We can boost its priority temporarily, and lower it in order to resume the highest-priority task as soon as possible.

## Define the round robin scheduling algorithm. How do we organize tasks in the runqueue? What does the scheduler need to know in order to schedule tasks?

Round robin scheduling begins like FCFS, in that it takes the first task off of a runqueue and runs it. When a task stops to wait on an I/O operation, we can schedule the next task in the queue, while the first task returns to the queue after its I/O operation completes. The next task in the queue is taken, and then T1 concludes last.

The runqueue in round robin scheduling is a normal FIFO queue--what separates the round robin scheduling algorithm from FCFS is that **a task may yield** when running I/O operations. When a task *yields*, it will be put back on the runqueue. Round robin scheduling may pre-empt based on a task's assigned priority.

## Generalize the round robin scheduling algorithm with priorities.

If a higher priority task arrives after a lower priority task, then the lower priority task is pre-empted, and returned to the runqueue. If two tasks arrive with the same priorities, the tasks are round-robin'd between on eanother until they complete.

## Generalize the round robin scheduling algorithm with interleaving.

**Timeslicing** is a method in which all tasks in a queue are deliberately interrupted/pre-empted, allowing another task to run until it is pre-empted, until all tasks run to completion. The round robin scheduling algorithm with interleaving takes advantage of this. (Resume lectures at 32:48)

## What is a timeslice and what are they used to achieve?

A timeslice, or a time quantum, is the maximum amount of uninterrupted time given for a task to run. This means **tasks can run in less than the time allotted for a timeslice**, then returned to a queue, where a scheduler will pick another task to execute.

Higher priority tasks will pre-empt a task, ending it before the end of its timeslice.

As a result, timeslices are meant to achieve timesharing, or an interleaved CPU.

## Calculate the metrics using the following data: given 3 tasks, T1 (execution time of 1), T2 (execution time of 10), and T3 (execution time of 1), and given that the 3 tasks arrive at the same time, what is the throughput, avg wait, and avg completion time of a round-robin scheduler with a timeslice of 1? Compare with FCFS (throughput=0.25 tasks/s, avg wait 4s, avg completion time 8s) and SJF (throughput=0.25 tasks/s, avg wait 1s, avg completion time 5s).

![Illustrating the question.](assets/P3L1/3.png)

With timeslicing, task 1 will be pre-empted by task 2, and task 2 will be pre-empted by task 3. Then, task 2 will resume until completion.

Round robin throughput is calculated to be 0.25 tasks/s, with an average wait of 1 second, and average completion time of 5.33 = (1+12+3)/3.

We observe that the advantages of timeslice scheduling allow shorter tasks (like T3) can finish sooner, and we achieve a responsive schedule that permits I/O operations to be intiiated sooner.

However, there are a number of overheads to be considered: we have to interrupt a task, schedule, and perform a task context switch. All of these times are **pure overhead**.

Depending on the length of the time slice and context switch times, we can minimize the overheads. So long as **timeslice length** is significantly larger than **context switch time**, overhead for these operations will be minimized.

## Assume that, for 2 CPU-bound tasks, there is an execution time of 10 seconds, and a context switch time of 0.1 seconds. Compare and contrast the performance of a round-robin scheduler with a timeslice of 1 second and a timeslice of 5 seconds. Calculate the throughput, average wait, and average completion time for both timeslices. What if the timeslice is infinite? Which one is more efficient and why?

![Illustrating the question.](assets/P3L1/4.png)

For throughput and average completion time, we are better off choosing a higher time-slice value. However, for average wait, we prefer a smaller timeslice.

That being said, users are not particularly interested in responsiveness, and the user cares more about when ALL of the tasks complete.

When timeslice is infinite, throughput and average completion time is optimal (0.1 tasks/s with avg completion time of 15 seconds), while average wait time increases (5 seconds), and it becomes the worst of the given timeslices.

## Assume that, for 2 I/O-bound tasks, there is an execution time of 10 seconds, and a context switch time of 0.1 seconds. I/O operations are issued every 1 second, and I/O operations complete in 0.5 seconds. Compare and contrast the performance of a round-robin scheduler with a timeslice of 1 second and a timeslice of 5 seconds. Calculate the throughput, average wait, and average completion time for both timeslices. What happens if only T2 is I/O bound, and T1 is CPU-bound? Which one is more efficient and why?

With a timeslice of 1 second, it is roughly the same as the round robin scheduler with a timeslice of 1 second working with CPU-bound tasks. However, we are also noticing that the values are the same for a timeslice of 5 seconds.

When only T2 is I/O bound, this affects the performance of the round robin scheduler with a timeslice of 5 seconds. Throughput decreases, average completion time also decreases (as in, becomes faster), but average wait time increases significantly.

![Illustrating the question.](assets/P3L1/5.png)

ERRATA:
    for Timeslice = 1sec
        avg. comp. time = (21.9 + 20.8) / 2 = 21.35

    Timeslice = 5 second*
        throughput = 2 / 24.3 = 0.082 tasks/second
        avg. wait time = 5.1 / 2 = 2.55 seconds
        avg. comp. time = (11.2 + 24.3) / 2 = 17.75 seconds

For I/O bound tasks, a smaller timeslice is overall much better--they have a chance to run more qiuckly, issue I/O requests, and respond to a user. This also keeps the CPU and other I/O devices busy, and NOT idle.

## In general, how long should a timeslice be?

For a **cpu-bound task**, timeslices should be longer, while for **I/O bound tasks**, they prefer shorter timeslices. CPU-bound tasks prefer longer timeslices because this improves throughput and average completion time for a CPU bound task. However, I/O-bound tasks prefer shorter timeslices, as this allows them to more quickly issue I/O requests and respond to users.

## LECTURE Q: On a single CPU system, consider the following workload and conditions: 10 I/O bound tasks and 1 CPU bound tasks, I/O bound tasks issue an I/O operation every 1 ms of CPU computing, I/O operations always take 10 ms to complete, and context switching overhead is 0.1 ms. All tasks are long running. What is the CPU utilization (%) for a round robin scheduler where the timeslice is 1 ms? See instructor's notes for formulas on CPU utilization.

STEPS TO SOLVE:

1. Determine a consistent, recurring interval
2. In the interval, each task should be given an opportunity to run
3. During that interval, how much time is spent computing? This is the cpu_running_time
4. During that interval, how much time is spent context switching? This is the context_switching_overheads
5. Calculate!

The CPU utilization formula is:

`[cpu_running_time / (cpu_running_time + context_switching_overheads)] * 100`

Note that `cpu_running_time` and `context_switching_overheads` should be **calculated over a consistent, recurring interval**.

![Illustrating the question.](assets/P3L1/6.png)

For every 1 ms of useful work, we have 1 ms + 0.1 ms of context switching overhead--meaning that, with a 1ms timeslice, CPU utilization is roughly 91%.

For every 10 ms of useful work, we calculate: (10 * 1 + 1 * 10 ) / (10 * 1 + 10 * 0.1 + 1 * 10 + 1 * 0.1) = 95%.

This represents 10 I/O bound tasks (which take 10 ms to finish completing, and which issue an I/O operation for every 1 ms of CPU computing) and 1 cpu-bound task.

## FINAL EXAM Q: How does scheduling work? What are the basic steps and data structures involved in scheduling a thread on the CPU?

Scheduling is basically the responsibility that the OS has to run different task in the system on the CPU in a way that is fair an abides by certain semantics.

In order to achieve this, the scheduler must primarily maintain a runqueue, which is a data structure(s) that holds all of the tasks in the system in some meaningful way. The runqueue only has to be a queue logically; technically, it can be a multi queue structure or even a tree.

Scheduling can be done in a first‐come‐first‐serve manner, in a round‐robin manner, or in a more complex manner. If we know the amount of time a task will take, we can schedule the shortest jobs first.

The type of runqueue that we create depends on the scheduling policy that we want to enforce. For example, if we wish to have priority scheduling, it might make sense to maintain a runqueue for each priority level.

In order to schedule a task on the CPU, the scheduler must first remove it from the runqueue and actually schedule it on the CPU. Once it is running on the CPU, the scheduler needs to be able to preempt the task if a more important task comes along. In some cases, the scheduler may need to maintain a timer in order to evaluate when a timeslice expires. Once the scheduler preempts a task, it needs to place it back on the appropriate (potentially different) runqueue, and context switch to the next task in the system.

## FINAL EXAM Q: What are the overheads associated with scheduling? Do you understand the tradeoffs associated with the frequency of preemption and scheduling? What types of workloads benefit from frequent vs infrequent intervention of the scheduler (short vs long timeslices)?

The primary overhead with scheduling is one of timing. Specifically, the time to context switch be‐ tween threads introduces some non‐trivial amount of delay into the system. Context‐switching fre‐ quently helps to reduce the wait time for any given task in the queue, but frequent intervention will drop the throughput and increase the average completion time.

Workloads that are primarily CPU‐bound will benefit from longer timeslices, as they will be utilizing the CPU completely when they are scheduled. Workloads that are primarily I/O‐bound will benefit from shorter timeslices, as frequent context‐switching can help to hide latency.

## How can we define a runqueue data structure that can handle both CPU-bound and I/O-bound tasks? Will we have to define two different data structures?

This is mostly dependent on whether we want **I/O and CPU bound tasks to have different timeslice values.** We can create a **multiqueue data structure** that has multiple separate queues.

Each queue can have a different timeslice associated with it.

![Illustrating the multiqueue structure.](assets/P3L1/7.png)

This solution, however, is dependent on knowing whether a task is CPU or I/O intensive.

## If creating a multiqueue data structure (such as the MLFQ / multi-level feedback queue) to handle CPU-bound and I/O-bound tasks that have different timeslice values, what do we do if we don't know how I/O intensive or CPU-intensive a task is? What about tasks with dynamic behavior, or new tasks the O/S has not encountered before?

When a task first enters the topmost queue, if a task voluntarily before the end of the first timeslice, then it will remain in the same queue. But if it uses up the entire timeslice, then we will push it down to the next queue. Any tasks that are pre-empted will be pushed down to the next queue with the larger timeslice.

However, if a task is repeatedly releasing the CPU earlier than the timeslice assigned to it, the O/S can also push it up to a queue with a smaller timeslice.

**The MLFQ** is not the same as a priority queue, mostly because of the feedback it offers based on a task's performance in the queue that it is initially assigned to.

## Compare the MLFQ with the Linux O(1) scheduler. How does the Linux O(1) scheduler offer feedback to the operating system? What data structures does the Linux O(1) scheduler use to manage its tasks? And how does the O(1) scheduler assign timeslices to the tasks added to it?

![Illustrating the O(1) scheduler.](assets/P3L1/8.png)

There are certain task categories--realtime, and timesharing. All tasks are assigned a priority based on which category they fall under, with higher priority tasks being real-time tasks (I/O-bound, 0-99), and all others (specifically user processes, which are assigned default 120) are assigned timesharing (CPU-bound, 100-139).

Higher priority tasks are given larger timeslices; however, lower priority tasks are given smaller timeslices.

Feedback is based on **sleep time**, or idle time. Longer sleep times are interactive--and their tasks are decreased (priority-5). This boosts the interactive task's priority so that it may run with higher priority next time. A smaller sleep time indicates that a task is CPU-intensive, and its task priority is increased (priority+5), reducing the task's priority.

The runqeue of the Linux O(1) scheduler consists of 2 arrays of tasks--one array is the active array, and another is the expired task.

The active list is used to pick the next task to run. It only takes O(1) to add or select a task to/from the queue. The scheduler relies on certain instructions that return the first set bit in a sequence of bits, corresponding to a priority level, so it is able to find a priority level containing tasks under constant time. If tasks are yieleded or pre-empted, the time spent on the CPU is subtracted from the assigned timeslice. If time remains, then the task returns to the active array. If no time remains, it will be put in the expired array.

Expired arrays contain tasks not currently active. It will not select tasks from th expired array if any tasks in the active array are running.
When  no more tasks are left on the active array, the pointer of these two lists are swapped, and the active array becomes the expired array.

This is not the default task scheduler due to how it affected the performance of highly interactive tasks. Linux now uses the CFS scheduler instead.

## How can we address the disadvantages of the O(1) scheduler with the CFS (completely fair scheduler)?

The CFS uses a **red-black tree** as the runqueue. Selecting a task takes O(1) time, but it takes O(log N) time to add a task to the run queue. More information about the red-black tree can be seen here: http://algs4.cs.princeton.edu/33balanced/ . Each node corresponds to a task, and are ordered by virtual runtime, or `vruntime`.

Children to the right of a node have spent more time on the CPU (increased vruntime), while children to the left node have spent less time on the CPU (reduced vruntime).

CFS schedulers will always pick the leftmost node. Then, i twill increment the vruntime of the task presently running on the CPU, and then compare it to the vruntime of the leftmost task node in the tree.

**If the value is smaller**, then the task will continue running; **if the value is larger**, then it is preempted and placed appropriately in the tree. Therefore, it will be selected after the shorter task on the tree is selected and executed.

CFS then changes the effective rate of a task's given virtual runtime. For lower priority tasks, time passes quickly, so virtual runtime progresses more quickly. However, for high-priority tasks, time passes more slowly, so they will be allowed to execute on a CPU for longer.

## LECTURE Q: What was the main reason why the Linux O(1) scheduler was replaced by the CFS scheduler? Is it because of A: Scheduling a task under high loads took an unpredictable maount fo time? B: Low priority tasks can wait indefinitely and starve? Or C: Interactive tasks could wait unpredictable amounts of time to be scheduled.

The answer is C: Interactive tasks can wait unpredictable amounts of time to be scheduled. B is a possible scenario, but not the main reason for the O(1) schedule to be replaced.

When a task has been moved to the expired list of the runqueue, it has to wait until ALL low-priority tasks exhausted their time quantum. As general purpose workloads involved more interactive tasks to run on the operating system, the O(1) scheduler was no longer a practical solution.

## What kind of architecture does scheduling on a multi-CPU system involve? What about a multi-core system?

![Illustrating the multiqueue structure.](assets/P3L1/9.png)

In a multi-CPU system, also known as a **shared memory multiprocessor**, there are multiple CPUs (which have their own private caches), their associated caches (**last level caches**, which may or may not be shared), and their system memory, DRAM (**direct access memory**). DRAM is shared amongst ALL CPUs.

In a multi-core system, we have a CPU with multiple cores, and each core has its own private cache. However, they share a last level cache, and once more share their system memory.

The operating system sees all CPUs, or cores in a CPU, as entities upon which they can schedule an execution context upon, specifically a thread.

## Walk through the steps behind scheduling on a multi-cpu system. How does cache access influence scheduling on a multi-CPU system?

A thread that's running on a CPU might be able to bring into its state a lot of relevant information from direct access memory and its LLC. But if it is later scheduled on a different CPU, the cache will no longer be hot--it will pull in the exact same state information from DRAM and accumulate overhead.

Our goal is thus to have this CPU run on the same CPU as before--or to achieve **cache-affinity**. We can maintain a **hierarchical scheduler architecture**, which, at the top level, employs a load-balancing component that divides tasks among CPUs, and a per-CPU scheduler with a per-CPU runqueue repeatedly runs those tasks on its associated CPU as much as possible.

The top-level entity in the scheduler can viewe information such as **length of each runqueue** or observe when a CPU is idle to begin associating more work to them so as to balance tasks across CPUs.

## Besites maintaining a hierarchical scheduler architecture, how else can we improve scheduling performance on a multi-CPU system?

We can employ something called **NUMA**, or non-uniform memory access platforms. We can have two separate memory nodes. CPUs and memory nodes will be interconnected by some kind of interconnect (IE: QPI), and the nodes are configured to be connected to some subset of a CPU. The access from that CPU to its associated memory node is faster than access to a remote memory node associated with a different CPU. But there are interconnects between each CPU to its memory nodes.

From a scheduling perspective, the scheduler will divide tasks in such a way that tasks are bound to the CPUs that are closer to the memory node where the task's states live. This type of scheduling is NUMA-aware scheduling.

## Why do we have to context-switch between threads? How does hypethreading facilitate an improved context switch?

This is because the CPU has only one set of registers to describe the active execution context of a thread running on the CPU, which include the stack pointer and program counter. Overtime however, we have learned to hide some of the overheads associated with context switching by using CPUs that contain sets of registers, each of which contain the context of a separate thread / execution identity. **This is known as hyperthreading.**

Hyperthreading is where multiple hardware supported execution contexts reside in a single CPU. **Nothing about the thread's state needs to be saved or loaded when performing a context switch.**

![Illustrating a hardware multithreaded architecture.](assets/P3L1/9.png)

Hardware today typically supports two hardware threads. We can also enable or disable hardware multithreading at boot time. When it is enabled, each hardware context appears to the operating system's schedule as a separate context, or a virtual CPU, upon which it can schedule threads, given it can load the registers with the thread context concurrently.

## In hardware multithreading, how does the operating system's scheduler determine which two (or more) threads to run concurrently on both hardware contexts of the CPU? Reference Fedorova's paper.

Recall that **if the `t_idle` is greater than 2 * `t_ctx_switch`**, then **context switching is performed in order to hide idle latency.**

If the time to perform a context switch is 0 cycles, while memory loading takes 100 cycles. In this case, it's easier to context switch to another hardware thread as opposed to performing hyperthreading. In other scenarios however, let us assume the following:

* **Threads issue an instrunction every cycle.** A CPU-bound thread will be able to obtain a maximum metric of IPC=1 for a single CPU.
* **Memory access takes 4 cycles.** This means a memory-bound thread will **experience idle cycles** while it is waiting for memory access to return.
* **Time to context switch between hardware threads is instantaneous.**
* **We have an SMT with two hardware threads.**

## What would happen if we co-schedule two CPU-bound threads? What about two memroy-bound threads?

![What happens when we co-schedule two CPU-bound threads.](assets/P3L1/11.png)

Because both CPU-bound threads will compete with each other for the CPU pipeline resource, **performance will degrade 2x**, and the **memory controller is idle.** THere are no memory-accesses which take place.

![What happens when we co-schedule two memory-bound threads.](assets/P3L1/12.png)

On the other hand, when we schedule two memory bound threads, there are many CPU cycles that are wasted while the memory-bound threads are waiting for the memory access to return.

![What happens when we co-schedule one CPU-bound thread, and one memory-bound thread](assets/P3L1/13.png)

The goal is to **co-schedule a memory-bound thread with a CPU-bound thread.** This will allow us to fully take advantage of CPU and memory, not leaving either component idle. This also allows us to limit contention on the processor pipeline. **A degree of interference and degradation will be observed but heavily minimized.**

## How do we know if a thread is CPU-bound or memory-bound?

Look at the threads' past behavior--in other words, apply a heuristic. While w eused sleep-time to determine I/O-bound vs CPU-bound tasks, we can't use that for determining whether threads are CPU-bound or memory-bound. Threads do not sleep when waiting on its memory node. Instead, we can employ the use of **hardware counters**.

Modern hardware has **hardware counters** that keep information related to various aspects of software execution. Cache hits and misses, the number of instructions retired (to calculate IPC) power + energy consumption when executing the thread, etc...

Hardware counters can **estimate what resources (CPU or memory) a thread needs to fulfill in order to execute.** The scheduler can use this information to **pick a mix of threads available on the runqueue** to schedule in the system so that all components of the operating system are properly used.

We can check the cache misses on the hardware counter and estimate if a thread is memory-bound or CPU-bound. Other information that the scheduler can take advantage of may involve a combination of hardware counter results, and information models created for a specific hardware platform that was trained on an understood workload (that might be known to be memory-intensive).

## How do cycles-per-instruction (CPI) help us determine whether a thread is memory-bound or CPU-bound? Refrence Fedorova's paper. What was the purpose of Fedorova's research into this question?

Fedorova observes memory-bound threads are high CPI, while CPU-bound threads are a lower CPI.

Given that there isn't a CPI counter on the hardware used in Fedorova's experimentation, she uses a **simulator** in which a CPU has a CPI counter, and determines whether a scheduler can use this information. Her goal was to determine **whether or not including a CPI counter on hardware would improve scheduling threads on a hyperthreaded system.**

## How did Fedorova test her hypothesis on CPI? Describe her experimental methodology.

She first establishes an environment, or a **testbed**, on a system with 4 cores. Each core is 4-way multithreaded, which means that there are 16 hardware contexts.

She then creates a synthetic workload wher eher threads have a CPI of 1, 6, 11, and 16. The thread with a CPI of 1 will be the most CPU intensive, while the thread with a CPI of 16 will be the most memory-intensive.

The overall workload defines 4 threads of each kind.

She then evaluates--what is the overall performance when a specific mix of thread is assigned to each of these 16 hardware contexts?

She uses a metric of **IPC,or instructions per cycle,** given that a system has 4 cores, then the IPC of 4, or 4 instructions are performed per cycle, will be the best case scenario where every single core completes an instruction in EACH cycle.

![Fedorova's Experiments](assets/P3L1/14.png)

In the first experiment, each core runs the same types of threads. But in the last experiment, each core runs a set of completely separate tasks. **Fedorova is trying to make static decisions** that a scheduler would make.

## LECTURE Q: See the diagram behind this answer. This diagram shows results from Fedorova's experiments. What do you think these results suggest about using CPI as a metric in scheduling?

![Fedorova's CPI Experiment results.](assets/P3L1/15.png)

Initial response: We observe that the 4th experiment, where the cores have specialized in the tasks with the given IPCs, there is a reduced **instruction per cycle** observed in the experimental results, particularly for the cores with a higher CPI. This suggests that fewer instructions are run for that cycle, meaning that this leads to wasted cycles on other cores.

Note that in cases A and B, the processor pipeline was very well-utilized, suggesting a higher IPC. However, in cases C and D, there are a lot of contentions on certain cores. Cores 2 and 3 contributed very little to the aggregate IPC metric, executing very few instructions per cycle.

**Based on Fedorova's simulation alone,** CPI is a helpful metric for scheduling threads. However, realistic workloads suggest that threads do not have such dramatically different CPI values, so it's not a useful enough metric.

There are very helpful takeaways from the paper: primarily resource contention in SMTs, especially in the processor pipeline. We also know now how hardware counters can be used to analyze thread workloads to better inform the scheduler, as well as how schedulers must choose a set of tasks to reduce resource contention as well as how to implement load balancing when scheduling tasks.

All in all, LLC usage would have been a better choice in Fedorova's hypothesis.

## FINAL EXAM Q: Can you work through a scenario describing some workload mix (few threads,their compute and I/O phases) and for a given scheduling discipline compute various metrics like average time to completion, system throughput, wait time of the tasks...

For system throughput, take the total amount of time to finish all of the tasks, and divide by the num‐ ber of tasks. For average time to completion, take the sum of the differences between the zeroth time and the time that each task completes, and divide by the number of tasks. For wait time, take the sum of the difference between the zeroth time and the time that each task starts, and divide by the number of tasks.

Don’t forget that context switching takes time.

I/O bound threads that are waiting on requests cannot hide latency when there is no other thread to switch to, so don’t forget to factor in request time in these cases.

## FINAL EXAM Q: Do you understand the motivation behind the multi-level feedback queue, why different queues have different timeslices, how do threads move between these queues... Can you contrast this with the O(1) scheduler? Do you understand what were the problems with the O(1) scheduler which led to the CFS?

The motivation behind the multi‐level feedback queue was to have a single structure that maintained different timeslice levels that threads could be easily moved through. Queues have different times‐ lices because tasks have different needs. CPU‐intensive tasks are better off have large timeslices be‐ cause the shorter the timeslice, the more context‐switching interferes with their task. I/O‐intensive tasks are better off having a shorter timeslice. Since I/O‐intensive operations issue blocking I/O re‐ quests, it’s valuable to have them issue their request and then immediately be switched out, to hide the latency of their request. Maintaining a structure that has different queues for different timeslice values captures the complexity of the system. The scheduler can move threads through this structure based on preemption vs. yield observation. When a thread is consistently preempted, this is inline with the assumption that the thread is compute‐bound. The scheduler can move this task to the end of a queue with a longer timeslice. When a thread consistently yields the CPU, this suggests that the thread is I/O bound. This thread should be moved to a queue with a smaller timeslice.

The O(1) scheduler maintains 40 different timesharing priority levels. The scheduler assigns the small‐ est timeslice to the compute‐bound tasks, and the largest timeslice to the interactive tasks. The feed‐ back for a task was based on how long it spent sleeping at its current priority level. Tasks that didn’t sleep at all (CPU bound) had their priority decremented by 5, while tasks that slept more (I/O bound) had their priority increased by 5. In addition, each priority level maintained two queues, one active, one expired. The active queue contains the tasks that are actively being scheduled on the CPU. Each of these tasks must expire its entire timeslice before it is moved to the expired queue. Only after all tasks are expired do the active queue and the expired queue swap places.

The problem with this approach is that tasks had to wait until all of the other tasks, in all of active run‐ queues at all of the levels of higher priority, exhausted their timeslices. This introduced unacceptable jitter in applications that needed to be performant in realtime, such as Skype. As a result, the com‐ pletely fair scheduler was introduced. This scheduled maintained a balanced tree structure, where each the left subtree contained tasks that had ran for less time on the CPU, and the right subtree con‐ tained tasks that had ran for longer time on the CPU. The scheduler always picks the leftmost task on the tree ‐ the one that ran the least amount of vruntime. After some point of scheduling, the scheduler updates the vruntime of the task and either preempts it ‐ if there is a new task with the lowest vrun‐ time ‐ or continues to execute it. For tasks with lower‐priority, these updates happen more quickly while for tasks with higher‐priority these updates happen more slowly.

## FINAL EXAM Q: Thinking about Fedorova's paper on scheduling for chip multi-processors, what's the goal of the scheduler she’s arguing for? What are some performance counters that can be useful in identifying the workload properties (compute vs. memory bound) and the ability of the scheduler to maximize the system throughput?

The goal of the scheduler that Fedorova is arguing for is a scheduler that maximizes CPU utilization in hardware multithreaded environments. Normally, we look at a metric called instructions per cycle (IPC) to determine utilization. Tasks that are CPU‐bound typically have IPCs that are close to 1, while tasks that are memory‐bound typically have IPCs that are closer to 0. Hardware counters can provide IPCs counts for the execution contexts that they manage. Federova is suggesting that hardware incor‐ porates a new metric, cycles per instruction (CPI), which is the reciprocal of IPC. Compute‐bound tasks would still have a CPI of close to 1, while memory‐bound tasks would have a CPI much higher than 1. With synthetic workloads that consist of tasks with different combinations of CPIs, she showed that the IPC value for the most varied combination was the closest to 1. As a result, she concluded that ‐ given her synthetic workload ‐ CPI would be a helpful metric track and thus help improve platform utilization. Unfortunately, empirical values of CPI did not vary nearly as much as her synthetic work‐ loads.

The IPC performance counter is helpful in determining if a task is compute‐bound or memory‐bound. In addition, performance counters that track cache misses can be helpful. With high cache misses, a scheduler can assume a task is making lots of memory access and therefore is memory‐bound. With few cache misses, a scheduler can assume a process is more compute‐bound.