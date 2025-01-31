## 1. Scheduling First Principles

With that, we conclude discussion of synchronization and communication issues in parallel systems. The next part of the lesson will cover scheduling issues in parallel systems. You'll notice once again when we discuss scheduling issues, that the mantra is always the same: namely, **pay attention to keeping the caches warm to ensure good performance.** 

We're going to look at scheduling issues in parallel systems. Fundamentally the typical behavior of any thread or  a process running on a processor is to do the following: 
1. compute for a while and then make a blocking IO system call,
2. or it might want to synchronize with other threads that it is a part of in the application, 
3. or it might be that it is a compute bond thread in which case it might just run out of the time quantum that it has been given by the scheduler on the processor. 

But fundamentally, what that means is: this is a point at which the operating system, in particular the scheduler piece of the operating system, can schedule some other thread or process to run on the CPU. 

So how should the scheduler go about picking the next thread or process to run in the processors given that it had the choice of other threads that in can run at any point of time?

## 2. Quiz: Scheduler

Now I'll turn that into a question for you. How should the scheduler choose the next thread to run on the CPU? I'm going to give you four different choices. The first is first come first served. That is if I have a bunch of threads the scheduler says well which was the earliest one that was ready to run on the processor? That's going to be the one that I'm going to schedule. That is first come first served. The second possibility is it's going to assign static priority to all the threads so it's going to pick the threads that have the highest static priority to run on the processor. The third possibility is that the priority is not static but it is dynamic or in other words it changes over time. And so what the scheduler is going to do is pick the one that has the highest dynamic priority. And the fourth choice is its going to pick a thread whose memory contents are likely to be in the cache of the CPU. So these are the four choices and I want you think about it and come up with some thoughts as to what might be the right thing that the scheduler might do in picking the next processor to run.

---


If you picked any or more or all of the choices that I gave you you're not completely off base. Let me just talk through each of these choices and why it may be a perfectly valid choice for the scheduler in picking the next thread to run on the processor. First come first search says well you know there is an an order of arrival into the processor there's a fairness issue I'm going to pick the one that became runnable at the earliest so there is a first come first serve policy.

The second is well somebody paid a lot of money to run the program and so I'm going to give it a priority that it statically assigned with every process or thread. And I'm going to pick the one that has the highest priority so that's also a valid choice. 

The third possibility is a thread's priority is not static. It may be born with a certain priority but over time it might change. 

Why might it might the thread's priority change over time? Well for one thing operating systems typically tend to give priority to jobs or processes or threads what do we even want to call them that tend to be interactive, that tend to take a short amount of time on the CPU and then go off and IO or synchronization. Those kinds of threads are shortest amount of time that it takes on the processor the schedule may want to boost up the priority of the process and give it a higher priority, even if it was born with a smaller static priority. And that maybe a reason why it might choose a higher dynamic priority. That's a third choice. 

And the fourth choice is pick the one whose memory contents in the CPU cache is likely to be the highest. What that means is the that thread that has the cache warn for its working set is likely to do really well when it gets scheduled on a processor. And so it makes sense to suggest that this might be a good choice as well. So all of these four choices one can't argue for and against. 

But in this particular lecture what we're going to think about is particularly look at this last choice and that is picking the thread whose memory contents are likely to be in the CPU cache. Picking that as a choice why that makes a lot sense especially in a multiprocessor where there's going to be several levels of caches and and cohesivenesses and so on and so forth. We'll discuss more about that in the rest of this lecture but I wanted to warm you up with this particular quiz In which we have all these different choices. And one can as I said argue for or against every one of these choices and there are valid arguments both for and against. But this is the choice that we're going to focus on for the rest of this lecture.

## 3. Memory Hierarchy Refresher

Here's a quick refresher on the memory hierarchy of a processor. As you know between the CPU and the main memory there are several levels of caches. And typically these days you may have it up to three levels of caches between the CPU and the memory. And the the nature of the the memory hierarchy is that you can have something that is really fast a small amount of or really slow and big amount of. And and so all of these choices that are in between are really in between the two extreme choices of an L1 cache and the main memory. 

The disparity in the CPU cycle time and the main memory if you take the disparity between the CPU and the main memory it's more than two orders of magnitude today and it's only growing. So any hiccup that the CPU has in not finding data or instructions that it needs to execute the currently running thread in the caches and it has to go all the way to the memory is bad news in terms of performance. 

So what this suggests is that in picking the the next thread to run on the CPU, it'll probably be a very good idea if the scheduler picks a thread whose memory contents are likely to be in the caches. If not in the L1 cache at least in the L2 cache. If not in the L2 cache at least in the L3 cache. So that it doesn't have to go all the way to the memory in order to get the instructions and data for the currently running thread. So that's an important point to think about.

## 4. Cache Affinity Scheduling

### What are the shortcomings of cache-affinity scheduling? 

So that brings us to this concept of cache affinity scheduling. 

Basically the idea is very very simple. And and that is if let's say that a particular process at P1. I had this thread T1 running for a while and it got descheduled at some point of time becuase it made an I/O call it tried to synchronizes another thread. Whatever it is. Or time quantum. Expired for T1 any of those situations will result in T1 one getting descheduled and then the schedule is going to use the process of for perhaps running some of the thread, but finally at some point of time if T1 gets ready to be scheduled again, it makes a lot of sense for T1 to be scheduled on the same processor. Why? Because it used to run on this processor P1 and therefore the the memory contents of T1 T1 that needed to have its execution. We're in the cache of P1 and therefore when T1 gets ready to run again if I schedule T1 on the same processor it is likely that T1 will find its working set in the caches of P1. That's the reason why it makes sense to say well look let's look at the affinity Of a particular threat to a processor. Cache affinity of a particular threat to a processor. So the cache affinity for this thread is likely to be higher for P1 because it ran on P1 before got descheduled and is rescheduled on when it is time to reschedule it if you reschedule it on the same processor good chance that T1 will find its working set. In the memory hierarchy the caches of processor P1. But can something go wrong? Well what can go wrong is the following. When T1 was descheduled the scheduler may have decided that okay P1 is now available for doing business for some of the thread so it scheduled T2 and it scheduled T3 And eventually when T1 gets ready to run again it's ready to run again but in between it's running on the processor here and running on the processor again here along this timeline. Two other intervening threads ran on P1. So watch out. The cache may be polluted by the contents of threads T2 and T3 So far as T1 is concerned. 

So when T1 runs again it's quite possible that it may not find a lot of its memory contents in the cache because these two guys that got in the middle of its running on the process at T1 may have polluted the cache and gotten rid of a lot of this stuff that used to belong to T1 and therefore even though we made this choice that well when T1 is ready to run let's put it in on P1. But it used to run before. And that way we can ensure that T1's working set is probably in the cache of process of B1. But unfortunately these intervening threads may have polluted the cache. So that's something that you have to watch out for. 

So the moral of the story is that you want to exploit cache affinity in scheduling threads on processors. But also you have to be worried about any intervening threads that may have run on the same processor and may have polluted the cache as a result. So that's something that you have to watch out for. So now that I've introduced the idea of cache affinity for a processor we'll just pick to a particular thread we are now ready to discuss different scheduling policies that an operating system may choose to employ

## 5. Scheduling Policies

### First Come First Serve 

The first scheduling policy is a very simple one first come first serve. And what this is saying is that you look at the order of arrival of threads into the scheduling queue of the scheduler and pick the one that is the earliest to become runnable again. And that's the one that you're going to schedule. So what this is saying is well basically we will give importance to fairness for threads as opposed to affinity–we ignore affinity altogether in favor of fairness. We'll pick the thread that became runnable at the earliest that's the one that we're going to pick as the next one to run on the processor. That's first come first served. 

### Fixed processor

The second scheduling policy is called fixed processor or in other words for every thread when I schedule the thread the first time I'm going to pick a particular processor. And I'm always going to stick to that. So the processor on which $T_i$ will run will always be a particular fixed processor. And the the the way we choose the initial processor on which to schedule $T_i$ may depend on the load balance, that is making sure that all the processors in the multiprocessor are equally stressed in terms of using the resources for running the available threads that are there in the system. And that's how I pick a particular processor for the life of this thread the processor on which Ti is going to run is always fixed. So that's fixed processor scheduling. 

### Last Processor

The third scheduling policy is what is called a last proccessor scheduling policy. The idea here is the processor is going to pick among the set of threads that are available to be run at any point of time. It is going to pick a thread that used to run on it. In other words if $T_i$ the last time it had any cycles from the system was on a particular processor. Then when this processor is coming around looking for work it'll see oh $T_i$ is there he used to run on me. I'm going to pick that guy to run on me again. And as you can imagine this is giving preference to the fact that there could be affinity for $T_i$ to this processor because it used to run on this. So that is what is called last processor scheduling and of course when a processor is looking for work and it looks at the run queue does not find any thread that used to run on it and of course it has to pick some thread right? So the inclination is to pick the thread that had run on this processor before. If such a thread is not available then you're going to pick something else. So the idea behind this is that **you want to make sure that if this processor is going to pick a thread to run on it the likelihood of this thread finding its memory contents in this processor is high**. That's that's what we're trying to shoot for in this last processor. 

### Minimum Intervening Policy

The next couple of scheduling policies I'm going to tell you about. It requires more sophistication in terms of the information that the scheduler needs to keep on behalf of every thread. You know in order to make a scheduling decision. The next scheduling policy is what is called Minimum Intervening scheduling policy. MI for short. And in MI what we're going to do is the following. We're going to keep track, for every thread its affinity with respect to a particular processor and pick the processor for running this thread in which this thread has the highest affinity.

## 6. Minimum Intervening Policy

### How do we calculate the affinity index?

The **affinity index** of a particular thread $T_i$ is defined as 

If you look at the timeline for a particular process of $P_j$ it might look like this–that $T_i$ was running here, got de-scheduled, and then there were a couple of other threads that ran on on $P_j$, $T_x$, and $T_y$. So now if I want to think about the affinity for $T_i$ with respect to this processor $P_j$. That affinity if we're going to schedule $T_i$ now on $P_j$ the affinity number that I want to compute for this guy is 2–indicating the number of intervening threads that ran on Pj between the last time Ti ran on it and if I schedule Ti now at this point of time. And so clearly this number that I'm talking about, the affinity number, **the smaller the number the higher the affinity**. 

When we say the affinity number is 2, it means there are 2 intervening threads that ran on $P_j$ between the time $T_i$ got dibs on $P_j$ now at this point of time and at this point of time. That's the idea behind this **affinity index**. So what we want to do is, in a minimum intervening scheduling policy, you want to keep this information about the affinity for $T_i$ to run on every processor. If I have a multiprocessor with 16 processors, then there is an **affinity index** associated with every one of those processors for Ti. It might be that on processor one, $T_i$ has an affinity index of 2, while on processor two it has an affinity index of 4 and so on and so forth. And what we want to do is, when it comes time to scheduling $T_i$, I want to pick a processor on which **the affinity index is the minimum**. So the **minimum affinity index** indicates that there is **the minimum number of intervening threads** on this particular processor. That's the processor on which I want to run $T_i$. That is amplifying the chance that $T_i$ is going to find its memory contents–the working set in the caches. That's the idea behind minimum intervening scheduling policy.

## 7. Minimum Intervening Plus Queue Policy

So that's your minimum intervening scheduling policy–that is ensuring that the processor that is picked for $T_i$ to run on has the highest affinity for $T_i$. 

There's a variant of minimum intervening which is called limited minimum intervening policy. That is, the limited minimum intervening policy will **limit the number of processors** for which a task maintains affinity information. This is especially important in situations where an arbitrarily large number of processes (IE: 1000 or more) are running in the multiprocessor. Naturally, affinity indeces of 2 or 3 are kept, but larger infinity indeces like 20 or 30 will be discarded in the policy. 

We can define the limit for the number of processor associations per task by some value $A$. For each associated task $x$ on the ready queue, LMI will compute the number of tasks executed on that processor since $x$ last executed. If the queue contains associated tasks, LMI runs the task with the minimum value–otherwise, it executes the first task on the ready queue and replaces the task's worst processor association with an association to the processor on which the task just ran.  Just keep the top candidates. That's the idea behind limited minimum intervening scheduling policy. 

The last policy I'm going to introduce you to. It's called **Minimum Intervening Plus Queueing**. The idea is still the same that I want to look at whether Intervening Threads that ran on a particular processor with respect to this thread that I am trying to schedule at this point of time, but when I make a scheduling decision that $T_i$ is going to run on a particular processor It may be that this particular processor $P_j$ may already have some other threads that are going to run on it and that's the idea behind minimum intervening plus queue. 

Scheduling decisions are thus not only based on the affinity index of Ti with respect to a particular processor, but also the queue for this particular processor. Why do we need to know do that? If $T_i$ s going to be scheduled on on this particular processor $P_j$, maybe there's a scheduling queue associated with $P_j$ which already has some number of threads to be run. That is, even though the processor $P_j$ is selected based on cache affinity, there is a chance that other threads are going to run before $T_i$ is scheduled to run.

![[Pasted image 20250129184730.png]]
So this was when $T_i$ ran last and I might find the definitive for $T_i$ with respect to Pj is 2, so the decision to put $T_i$ on on $P_j$ is feasible. However, if the scheduling queue of $P_j$ has $T_m$ and $T_n$ already populated, then by the time that $T_i$ gets to run on the process of $P_j$, $T_m$ and $T_n$ would have run on the processor and modified the contents of the cache, rendering the affinity index calculated beforehand inaccurate. 

Therefore, in order to account for the queue of a given processor $P_j$, we perform the following calculation: 
$$
T_i \rightarrow P_{j}^{(I+Q)_{min}}
$$
...where $I$ is the affinity index, and $Q$ is the size of the scheduling queue associated with this particular processor $P_j$. So that's the last scheduling policy. 

In short, we have introduced five different scheduling policies: 

- FCFS, where fairness is approved over affinity,
- Fixed processor, where $T_i$ is associated with $P_{fixed}$,
- Last processor, where $T_i$ is associated with $P_{last}$,
- Minimum intervening, where $T_{i} \rightarrow P_{j}^{I_{min}}$
- and minimum intervening plus queuing, where $T_i \rightarrow P_{j}^{(I+Q)_{min}}$

> **Note:** for minimum intervening and minimum intervening plus queueing, we are implementing the **limited** versions of these scheduling policies, that is, we do not store the affinity indeces for every process in the system in order to ensure the efficient performance of an operating system. 

## 8. Summarizing Scheduling Policies

|                       | Scheduling Policy              | Definition                             |
| --------------------- | ------------------------------ | -------------------------------------- |
| **Neutral**           | FCFS                           | Fairness is prioritized over affinity  |
| **Thread-centric**    | Fixed processor                | $T_i$ is associated with $P_{fixed}$   |
|                       | Last Processor                 | $T_i$ is associated with $P_{last}$    |
| **Processor Centric** | Minimum intervening            | $T_{i} \rightarrow P_{j}^{I_{min}}$    |
|                       | Minimum intervening plus queue |  $T_i \rightarrow P_{j}^{(I+Q)_{min}}$ |

So to summarize the scheduling policies I already mentioned that first come first serve simply ignores affinity and pays attention only to fairness. 

And these next two policies that I introduced to you fixed processor and last processor the focus is on cache affinity of a thread with respect to a particular processor. That's what we're focusing on. And Fixed Processor was the last processor. 

The next two policies they focus not only on cache affinity but also **cache pollution**. In particular it asks the question **how polluted is a cache going to be by the time $T_i$ gets to run on the processor**? That's the question we're asking (no period) In both minimum intervening as well as minimum intervening plus queuing in terms of making a scheduling decision. And that should be clear from the discussion up until now the amount of information that the scheduler has to keep grows as you go down this list. The order of arrival is all that you care about you put them in priority order in the queue and you're done with it. And the scheduler has to do a little bit more work in each one of these cases and corresponding with the amount of information that this schedule has to keep for every one of these scheduling policies is going to be more. 

But the result of scheduling decision is likely to be better when you have more information to make the scheduling decision. 

Another way to think about the scheduling policy is that **the fixed processor and the last processor is thread-centric** in saying what is the best decision for a particular thread with respect to it's execution. Whereas MI and minimum intervening plus queuing are processor-centric saying that what thread should a particular processor choose in order to maximize the chance that the amount of cache contents is going to be relevant for the currently scheduled thread? 

So that's what we're looking at. Now that I've introduced to you these scheduling policies it's time for a quiz.

## 9. Scheduling Policy Question

![[Pasted image 20250129190333.png]]

Information that we have available is that on some processor $P_u$, the queue contains a task $T_x$. On another processor $P_v$, the queue contains four threads in its queue $T_m$, $T_q$, $T_r$, and $T_n$. 

Given a particular thread $T_y$, such that the affinity of $T_y$ with respect to $P_u$ is 2 and the affinity of $T_y$ with respect to $P_v$ is 1. 

> **Note:** Recall that the affinity index is precisely the intervening thread index, and that the smaller the index the higher the affinity.

There's a number of intervening tasks that have run on the process of $P_u$ and $P_v$ respectively since the last time $T_y$ had a chance to run on these processors. The scheduling policy is minimum intervening plus queue. 

Given this scheduling policy, when $T_y$ is ready to be put on a queue, which queue will the scheduling policy decide to place $T_y$? Is it $P_u$'s queue, or $P_v$'s queue?

---


First, calculate the affinity that $T_y$ has on $P_u$. 

$$
(I + Q)_{min} = (2 + 1) = 3
$$

Then, calculate the affinity that $T_y$ has on $P_v$. 

$$
(I + Q)_{min} = (1 + 4) = 5
$$

The overall min of I plus Q for Ty with respect to $P_u$ is 3. Let's do the same thing for Ty on Pv. In the case of $P_v$, its affinity apparently is higher because there's only 1 intervening task that ran since the last time $T_y$ ran on it, but we have to also account for $P_v$'s queue. 

When $T_y$ gets put on Pv's queue, is scheduled behind the 4 threads set to run in the queue, hence the affinity index is lower than that of $P_u$. Therefore, $P_u$ will lead to the least amount of cache pollution for the execution of $T_y$. 

## 10. Implementation Issues

### Queue-based global queue 

Now that we looked different scheduling policies let's discuss the implementation issues of these scheduling policies in an operating system. One possibility is the operating system can maintain a global queue of all the threads that are runnable in the system. And what these processes might do is. When they're ready for work they'll go to this global queue and pick the next available thread from this queue and run that on itself. And the way we organize the queues is orthogonal to the scheduling policy itself. But if the policy is something like FCFS it makes sort of logical sense to have. A global queue and let the processes pick from the queue the head of the queue is the earliest arriving thread and therefore first come first serve policy we use this global queue policy. This global queue becomes very infeasible as an implementation vehicle. When the size of the multiprocessor is really big. Because then it's a huge data structure that all these guys have to access centrally and so on. 

### Affinity-based local queue

So typically what is done is to keep local queues with every processor. And these local queues are going to be based on affinity. And the particular organization of the queues. In each of these processes. These local queues for each of these processes is going to depend on the specific policy that you're going to use. So if it is last processor or is it is it fixed processor is it a minimum intervening or is it minimum intervening plus queuing, all of those things will decide how these local queues are going to be maintained. 

But important point I want to get across is that, in implementing the scheduling policies, **you have to have a ready queue of threads from which the processor will pick the next piece of work to do**. And **the organization of these queues** will be based on the **specific scheduling policy** that you might choose to employ for the scheduler. And it might be that processor $P_2$ runs out of work completely nothing in its local queue. In that case it might pull its peers' queues in order to get some work from other guys and run it in that processor. Now that's something that might be done and that is what is called **work stealing** in the scheduling literature. So that might be something that is commonly employed in a multiprocessor scheduler.

So I mentioned that the way these queues are organized is based on policies that scheduler picks which might be affinity-based or might be fairness-based and so on. But in addition to the policy specific attribute, it might also use additional information in order to organize its queue. 

In particular a priority of a thread is determined by three component: 
- affinity for a particular processor,
- priority, 
- and age (in which seniority is prioritized)

Now one component is the affinity component assuming it's an affinity based scheduling policy. But in addition to that it might also use additional information. So for instance every thread may be born with a certain priority so that is the base priority that. A particular thread has when it is started up and as I mentioned it might depend on whether you know they usually give a huge amount of money you know to run this particular thread. So that is the base priority that you associate with the thread and and of course then you take the affinity into a into account. And in addition to that there is age coming in. And this is sort of like a senior citizen discount. If a if a thread Ti has been in the system for a long long time you want to get it out of the system as quickly as possible. So what you do is equivalent to giving a senior citizen discount. You boost the priority of the thread by At certain amount so that it gets to to be at the head of the queue and it will get scheduled on the process of p2. 

So basically the priority attribute is what determines the position in the queue in the particular thread. And as I said three attributes that go with it is a base priority that you may associate with a thread then it is first created, the affinity it has for a particular processor and also the senior citizen discount that it might give to a particular thread depending on how long it's been on the system.

## 11. Performance

So having discussed several different scheduling policies we have to talk about performance. Now the figures of merit that is associated with the scheduling policy are threefold: **throughput, response time, and variance.** 

### Throughput

The first scheduling policy figures of merit is what is called throughput. And as the name suggests what this is saying is **how many threads get executed or completed in per unit time.** It's a system centric metric. It doesn't say anything about the performance of individual threads how soon they are performing their work and getting out of the system but it is asking the question: what is the throughput of the system with respect to the threads that need to run on it? 

And the next two metrics are user-centric metrics. 

### Response time

The response time is saying if I start up a thread how long does it take for that thread to complete execution? And that's what response time is saying and variance of responsing's time is saying. Well does the time that it takes for me to run my particular thread vary depending on when I run it on the system? Why will it vary? well for instance if you think about a first come first serve policy if I have a very small job to run and if it gets the processor immediately it's going to quickly complete its execution. 

But suppose when I start up my particular thread, there are other threads in the system ahead of me that are going to take a long time to execute . then I'm going to see a very poor response time. So depending on from run to run the same program may experience different response times depending on the load that is currently on that system. And that's where the **variance** of response time comes in. Clearly from a user's perspective I want response time to be very good and variance to be very small as well. 
### Variance 

Now when you think about first come first serve scheduling, the figure of measure that is really good about it is the fact it is fair. But it doesn't pay attention to affinity at all. And it doesn't give importance to small jobs vs big jobs. It's just doing it first come first serve and therefore there's going to be a high **variance** especially if it is small jobs that need attention of the processor and there are long-running jobs on the processor connecting.

## 12. Performance Continued

![[Pasted image 20250129192342.png]]

Now if you look at the memory footprint of a process, and the amount of time it takes to load all of its working sect into the cache, the bigger the memory footprint the more time it's going to take for the processor to get the working set of a particular thread into the cache so that the cache is warm enough and the process of the thread can do its work without having to have those hiccups where it has to go to the memory in order to fetch the contents in the cache. 

What this suggests is that it's **important to pay attention to cache affinity** in scheduling. And so the variance of cache affinity scheduling that we talked about are all excellent candidates to run on a multiprocessor. And in fact the minimum intervention or minimum intervening scheduling policy and the minimum intervening scheduling with queuing both of those are very good policies to use when you have a **fairly light to medium load on a multiprocessor**. Because that is the time when it is likely that a thread when it is run on a processor, if it has an affinity for that particular processor, the contents of the cache are going to contain the memory contents for that particular thread. 

But on the other hand if you have a very heavy load in the system, then it is likely that by the time a thread is run on a processor on which it supposedly having an affinity, **all of the cache may have been polluted** because the load is very heavy. So in between the time that a thread got run on a particular processor, next time it runs on the same processor maybe its cache contents have been highly polluted by other threads, and therefore if the load is very heavy then maybe a **fixed processor scheduling** may work out to be much better than the variants of minimum intervening scheduling policies. 

So the moral of the story is that you really have to pay attention to both **how heavily loaded your processor is or system is** and also **what is the kind of workload that you are catering to**. Both those things play a part in deciding what what we will be best scheduling policy and it may not always be the case that the same scheduling policy applies in all circumstances. 

So a real agile operating system may choose to vary the scheduling policy based on the load as well as the current set of threads that need to run on the system. 

Another interesting wrinkle to taking a scheduling policy is the idea of **procrastination**. Why would procrastination help? First of all, how do we implement procrastination? Well what the processor can do is it is actually ready to do do some work. But what it does is it it's going to insert an idle loop because it has it's looking in the the scheduling queue and it sees that there is no thread in the scheduling queue that has run on it before. And therefore if it schedules any one of those threads though all of those threads are not going to find any of their working set in the cache of the processor, the processor says okay now let me just spin my wheels for a while not schedule anything. **It is likely that a thread that has its cache content in that processor becomes runable again**. And then you can schedule that and that might be a win in terms of performance. 

So in other words procrastination may help boost performance. We saw that already in the synchronization algorithms where we talked about **inserting delays** in the scheduling algorithm in order to reduce the amount of contention in the interconnection network. It's the same sort of principle. Often times you'll see in system design procrastination actually helps in boosting performance. It helps in the synchronizational rhythms it helps in scheduling and later on when we talk about file systems you'll see that it helps in the design of file systems also.

## 13. Cache Affinity and Multicore

So let's talk about cache affinity and modern multicore processors in modern multicore processors you have multiple cores on a single processor, and in addition to the multiple cores that are in a single processor the processors themselves are also hardware multithreaded. 

What hardware multithreading means is that if a thread that is currently running on a processor on a core $C_1$ and is experiencing a long latency operation, for instance it misses in the cache and therefore has to go out in order to fetch the contents from memory that's a long latency operation, the hardware may switch to one of the other threads and run those. So in other words it wants to keep this core busy. There's only one execution engine in this core but it has four threads that it can run on this core. Depending on what these threads are doing if they are involved in long latency operations meaning they are going out they're not switched out of the processor in terms of operating system scheduling. It is just that these are the the threads that have been scheduled to run on this core and the hardware is switching among these threads by itself without the intervention of the operating system. It is automatically switching among these threads depending on what these threads are doing. 

If this thread does the memory access which is going outside the processor, then the hardware is going to say well you know this guy is going to be waiting for a while so let me switch to this guy and let him do its do its work because it's possible that what he needs is available in the cache. And if this thread also makes a long latency operation like a memory access then the hardware can switch to yet another thread, and another. So if all of these guys are waiting on memory then of course the core is not able to going to be going to be able to do anything useful until at least one of these memory accesses are complete. So that's the idea behind hardware multithreading. It is not very unusual for modern multicore processors to employ hardware multithreading. 

![[Pasted image 20250129192335.png]]

In this example I'm showing you there are four cores and in each core I have four hardware threads. So it is a four way hardware multithreaded core. And I'm showing you two levels of caches L1 and L2 cache. L1 cache is specific to this particular core C1 shared by these threads. Similarly L1 cache here is specific to this core C2 shared by the threads that are on it. On the other hand this L2 cache is common for all the cores. So if there is a miss in any of these L1 caches the hope is that we were able to find it in the L2 cache. If the processor. Has only these two levels of caches L1 cache and L2 cache. This thing in L2 cache is really bad news because then you're going all to all to the chip. It's a long latency memory operation. And modern multiprocessors may in fact even employ even more levels of caching. In addition to L1 and L2 there may be an L3 cache. It's normal to have modern processors having at least three levels of caches on the chip. And L1 cache associated with core and a shared L2 cache and a shared L3 cache. So that's the structure that you might see in modern mulitprocessors. 

So what we have to think about now is cache affinity and the modern multi core processes and how the operating system should make its scheduling decisions. 

So here again there's a partnership between the operating system and the hardware. The hardware is providing these hardware threads inside each core. And what the operating system is doing is picking which threads that it has in its pool of runnable threads and map them on to the threads that are available in the hardware. Clearly the scheduling decision what it tries to do is to make sure that most of the threads that are scheduled a particular core may find their working set in the L1 cache if possible and similarly the threads that are scheduled on this may find its working set in the L1 cache of C2 if possible and so on. 

And also the other thing that the operating system may try to do is if you just take the universal all the threads That are currently scheduled by the operating system to run on all these four cores. **You want to make sure that working set of all these threads are likely to be found in the L2 cache** because if you're missing the L2 cache bad news because then you're going outside the chip and that's a very long latency memory operation. And of course you can extend this idea if there is a third level of cache but to make things concrete let's just stick to two levels of caches L1 cache and L2 cache and. 

The criterion for operating system is to make sure that the threads that are currently scheduled on the processors that's available all the cores that are available what it wants to try to do is make sure that all the threads will find the contents in the L2 cache. Because missing in the L2 cache is going to be a long latency the memory operation.

## 14. Cache Aware Scheduling
![[Pasted image 20250129192657.png]]

So let me briefly introduce here the idea of cache aware scheduling when you have these multithreaded multi-core processors. And to make things concrete let's assume that you have a pool of ready threads and in this case I'm going to tell you that the pool of ready threads I have is 32. So I have 32 ready threads and I have a four way multi-core per CPU. Meaning that there are four cores in the CPU and each core is four way hardware multithreaded. Or in other words at any point of time the operating system can choose from this pool of ready threads 16 threads to be run on the processor. 

Because that's the number of hardware multi-threads that are available if you pool together all the four cores. So each core has four multi hardware multi-threads together they have 1600 threads that can be run on the CPU at any point of time. And the the job of the operating scheduler is to pick from the available pool of ready threads 16 candidates to be scheduled on the CPU. So how does the operating system choose the 16 threads to be run on the CPU at any point of time? 

What the operating system should try to do is it should schedule some number of cache frugal threads and some number of cache hungry threads on the different course so that together the sum of all the cache hungriness of the 16 threads that are executing at any point of time in the CPU is less than the total size of the L2 cache. And as I said L2 cache in this simple example I gave you two levels of caching a caching that is associated with each of these cores and an L2 cache that is sitting outside of these cores but it is it is common to all the four cores. But of course you can generalize this and say it is the last level cache or in other words you want to make sure that this the universe of threads that are scheduled at any point of time on the CPU the sum total of the cache requirements of the universal thread scheduler on the processor is less than the total capacity of the last level cache in the CPU. Because if it's missing the last level cache on the CPU you're going outside the chip out of memory long latency operation bad news. That's the thing that you're trying to do. 


So we're going to categorize threads as either cache frugal threads ($C_{ft}$) or cache hungry threads ($C_{ht}$). So cache frugal threads are one in ones that require only a small portion of the cache to keep them happy. On the other hand, a cache hungry thread is one that requires a huge amount of cache space in order to keep it happy meaning that the working set of cache hungry threads is much bigger than the working set of cache frugal threads. 

Now how do we know which threads are cache frugal and which threads are cache hungry? Well that's something that we can know only by **profiling the execution of the threads overtime**. The assumption is that many of these threads get to run on the CPU over and over again so overtime you can profile these threads and figure out whether a particular thread belongs to this category of cache frugal thread or this category of cache hungry thread. And the criterion that you want to use in picking the set of threads to be populated in the CPU at any point of time from the pool of available threads is to make sure that the sum of the cache requirement of all the cache frugal threads is that there are N cache frugal threads and there are M cache hungry threads then the cumulative cache requirement of all the threads put together is less than the total size of the L2 cache. That is: 

$$
\sum_{1}^{n}C_{ft} + \sum_{1}^{m}C_{ht} \leq sizeof(LLC)
$$

And then I told you we can generalize this L2 cache to the last level cache that is the cache that is sitting at the last level inside the CPU beyond which you had to go out of the chip go out to memory and so that last level cache becomes the determinant in saying whether the size of that last level cache is within bounds of the cache requirements of all the threads that I want to schedule. So this is the set of threads that I want to pick where in this particular case since the total number of hardware threads that I have available to me is 16 I want to make sure that M the cache hungry threads, N the cache frugal threads, is 16 and this inequality is satisfied as well. So that's what we want to shoot for in picking the set of threads to run on the processor at any point of time. 

### Thread profiling and performance

I mentioned that we have to profile these threads or monitor these threads as they're executing in order to figure out their cache occupancy over time so that we can categorize these threads as cache frugal or cache hungry, and the more information the scheduler has the better decision it can take in terms of scheduling. Be we have to be careful about that. In order for the system to do this monitoring and profiling clearly the operating system has to **lose some work** in the middle of these threads doing useful work. And I always maintain that a good operating system gives you the resources that you need and gets out of the way very quickly. So you have to be very careful about the amount of time that the operating system takes in terms of doing this kind of monitoring and profiling and this information is useful in scheduling decisions, **but it should not be disrupting useful work** that these guys have to do in the first place. Or in other words **the overhead for information gathering** has to be kept minimal so that the OS does not consume too many cycles in doing this kind of overhead work accounting for making better decisions in in terms of scheduling.

## 15. Conclusion

Since it is well known that processes scheduling is np complete we have to resort to heuristics to come up with good scheduling algorithms. the literature is ripe with such heuristics as the workload changes And the details of the parallel system namely how many processors does it have how many cores does it have how many levels of caches does it have and how are the caches organized. There's always a need for coming up with better heuristics. In other words we've not seen the last word yet on scheduling algorithms for parallel systems.

