## 1. Lesson Summary 

## 2. Synchronization Primitives 

What is a lock? If you have multiple threads executing, and they share one data structure, it is important that the threads do not overwrite one another's work. A lock is something that allows a thread to make sure that when it is accessing some piece of shared data, it is not being interfered with by another thread. 

So a thread can acquire a lock. One the lock is acquired, the thread knows that it can access some data that is shared by other threads. Once T1 knows it has access to this data structure, it can make updates to the data structure, and after it is finished, it can release this lock. 

Locks come in two flavors--a mutual-exclusion lock, and a shared lock. 

### Mutual exclusion lock

A **mutual-exclusion lock** (also known as an exclusive lock), which is that it can be used by only one thread at a time. Mutual-exclusion locks guarantee that no other thread will interfere with the data protected by the lock except for the thread that has acquired the exclusive lock at present.

### Shared lock 

There are also **shared locks**. This lock is something that allows multiple threads to access data at the shared time. Under what conditions, however, would this be meaningful? Say that there are records in the database that multiple threads want to inspect at the same time, but under the guarantee that no data will change while the records are being inspected. A shared lock guarantees multiple readers access to data with the assurance that no other thread will modify the data while it is being accessed by readers. 

### Barrier Synchronization 

The idea behind barrier synchronization is that there are multiple threads performing a computation, but require knowing the statuses of the other threads involved in the computation at some given time. They require the guarantee that all other threads have reached a particular point in their respective computations so that they can all move to the next phase of the computation. 

![[L04A_02_01.png]]

In this example, is possible that thread $T_1$ and $T_2$ has arrived at the barrier, but the other threads have not. However, until all thread $T_n$ arrived at the barrier, no other phase of the computation can proceed. 

Now that we understand the basic synchronization primitives required on a shared memory machine, we can start looking at how to implement them. 
## 3. Quiz - Programmer's Intent 

In the instruction set architecture of a processor, instructions are atomic by definition. In other words, if you think about reads and writes to memory, they are implemented as loads `LW` and stores `SW` . During the execution of either a load or a store instruction, the processor **cannot be interrupted**--this is the definition of an atomic instruction. 

The question here is, if we have a multi-threaded program, where there are two processes performing the following functions: 

|        | $p_1$              | $p_2$                     |
| ------ | ------------------ | ------------------------- |
| line 1 | `modify struct(A)` | `wait for modifications;` |
| line 2 |                    | `use struct(A);`          |

This is an example of a **producer-consumer relationship**, where one process is producing data, and another process is consuming it after verifying that the data has been produced successfully. 

Is it possible to achieve the programmer's intent that has been embodied in this code snippet? If so, illustrate how so. 

---

The answer is yes, it is possible. The solution is very simple--we can implement a new `flag` variable. Simply, we can set the flag to 0--`flag = 0`. 

Then, the processes will signal the modification of data between one another using this flag variable. Here, process 1 will look like: 
```cpp
mod(A);
flag = 1; // signal p2
```

And process 2 will look like: 
```cpp
while (flag == 0); 
use (A); 
flag = 0; // re-initialize the flag 
```

The producer will modify the data structure, and once it is done with the modification, it will set the flag equal to `1`. Process 2 is waiting for the flag to be set to `1`, and once it is set to `1`, process 2 will exit the spin loop.

Now, let us analyze this solution and see why it works with simply atomic reads and writes. 
## 4. Programmer's Intent Explanation 

Process 1:
```cpp
mod(A);
flag = 1; // signal p2
```

Process 2:
```cpp
while (flag == 0); 
use (A); 
flag = 0; // re-initialize the flag 
```

Note that all of these commands are simple read/write accesses. Process 1 is modifying data using loads and stores, while process 2 is simply reading and using a value with the same loads and stores. 

However, note that there is a difference between the ways in which the processes are using the `flag` variable, as opposed to the memory variable `A`. The `flag` variable is being used as a **synchronization variable**, but it is being accessed by the same read/write accesses available in the processor. 

Atomic read/write operations are good for doing simple co-ordination among processes.. But how can we implement a synchronization, such as a mutual-exclusion lock? Are atomic read/write operations sufficient for implementing a mutual-exclusion lock? 
## 5. Atomic Operations

Below is a very simple implementation of a mutual exclusion lock. 

```cpp
lock(L) {
	if (L == 0) {
		L = 1;
	}
	else {
		while (L == 1); // wait
		// go back to check if (L == 0)
	}
};

unlock(L) {
	L = 0;
}

```

In terms of the instructions the processor will execute in order to get this lock, it will need to be able to: 
- check if the lock is currently available, 
- and indicate that the lock is or is not available
To release the lock, we will simply set the lock `L` to 0, to indicate that the lock has been released. 

Can we implement this simple lock using only atomic reads and write alone? 
If we look at the set of instructions that the processor has to execute in order to acquire the lock, it has to: 
- read `L` from memory, 
- check if `L` is `0`, 
- store that new value `1` into the memory location `L`
All of these steps must be executed **atomically**, so that nothing can interfere with the execution of these steps. 
While read and write instructions are atomic individually, **a group of read and write instructions is not atomic.** What that means is that reads and writes **cannot be the only atomic operations** if we want to implement a lock algorithm. As such, we will need a new semantic for an atomic instruction that can do all of these steps in one atomic operation. 

In other words, we want to **read from memory**, **modify the value**, and **write it back to memory** in a single atomic instruction. This is needed in order to ensure we can implement a lock algorithm. This will be referred to as an atomic read-modify-write instruction--`RMW`. 

### Atomic `RMW` instructions

There are a number of existing atomic `RMW` instructions. Generically, the following instructions are referred to as fetch-and-$\varphi$ instructions, meaning that the operation is going to fetch an old value from memory, and then perform some modification $\varphi$ to this memory in an atomic manner. 

#### `test-and-set(<mem_loc>)`

This will atomically return the current value in the memory location before setting the memory location value to 1. 

#### `fetch-and-inc(<mem_loc>)`

Atomically fetches the old value from memory and increments the current value in the memory by `1`, or any other value. 

## 6. Scalability Issues With Synchronization 

There are a number of scalability issues with synchronization primitives, particularly in lock and barrier algorithms. The sources of inefficiencies include: 
- latency 
	- This is the time spent by a thread in acquiring an available lock. 
- waiting time 
	- This is the time spent by a thread in waiting to retrieving access to an unavailable lock. Waiting time is a function of the application that is out of the control of an operating system designer. 
- contention 
	- If several threads are waiting to access a lock, they are **contending** for lock access. How long it takes in the presence for one thread to obtain the lock and the others to give up access to the lock is the inefficiency incurred by contention. 

Latency and contention are the primary concerns of operating systems designers when implementing synchronization primitives. 

## 7. Naive Spinlock

When a processor is **spinning**, this means the processor is doing no useful work, but is simply waiting for the lock to be released. 

The first naive spinlock implementation is referred to as the naive spin on test-and-set. 

### Spin on test-and-set 

```cpp 
lock(L) {
	while (test_and_set(L) == locked); // spin on test and set. 
}

unlock(L) {
	L = unlocked;
}
```

Say that we have three threads, $T_1$, $T_2$, and $T_3$, contending for some shared memory location $L$, which has been initialized to `unlocked`. When we call the `lock` primitive, it executes the `test_and_set` atomic instruction, which returns the old value of $L$ and returns the old value from $L$ and set it to the new value, which is `locked`. 

If one thread finds that `test_and_set` returns `locked`, it will wait for `test_and_set` to return `unlocked` on the memory location $L$. This is where the thread spins. 

Assume in our example that $T_1$ has obtained lock $L$ before threads $T_2$ and $T_3$. When $T_2$ and $T_3$ try to obtain the lock, they will execute the same `lock` algorithm and discover that $L$ is locked. Therefore, $T_2$ and $T_3$ are effectively stuck spinning until $T_1$ calls the `unlock` function. 
## 8. Quiz - Problems With Native Spinlock

What are the potential problems with this naive implementation of the spin on test-and-set spinlock? 

- Too much contention 
- Does not exploit caches
- Disrupts useful work 

---

All three are correct.

- Too much contention 
	- First of all, with this naive implementation, there is going to be too much contention for the lock when the lock is released. All three threads in our instruction is performing the test-and-set instruction in an attempt to acquire the lock. If there are thousands of processors, everyone is going to execute this test-and-set instruction,  which leads to excessive **contention on the network** when attempting to access this shared variable.
- Does not exploit caches
	- A shared memory multiprocessor has private cahes associated with every single processor in the architecture. While there is a private cache associated with every processor, and a value from memory can be cached within it, the `test_and_set` instruction cannot access the private cache, because the `test_and_set` instruction cannot atomically perform read-modify-write instructions as well as include an access to the cache. By definition, a test-and-set instruction is not going to exploit caches. It is going to bypass the cache and go directly to memory.  
- Disrupts useful work 
	- This is also a good answer because when a processor releases the lock, the processor wants to go on and do some useful work. Similarly, if there are several processes trying to acquire the lock, if the lock is unavailable, all other processes will continue to contend for the lock instead of continuing to perform useful work. 

## 9. Caching Spinlock (spin on read)

![[L04A_02_02.png]]

While a test-and-set instruction is forced to go to memory in order to preserve atomicity for read-modify-write instructions, we can make an improvement to our lock algorithm such that contending threads can access their cache instead of spinning on the shared memory location.

Assume that we have a shared-memory machine through which hardware is maintaining cache coherence. The waiting threads can spin locally on the cached value of the lock $L$ instead of making a direct call to main memory.  When spinning on the local cached value of $L$, we can assume that, through cache coherence, any changes to $L$ made on main memory will be propagated to the private caches of each processor, which will notify the waiting threads. 

```cpp
lock(L): 
	while (L == locked); // retrieve L from main memory and spin on the cached value of L. 
	 
```

By spinning on the locally cached value of $L$, there is no longer any contention on the network. 
If the one processor with the lock releases it, **all contending processors will notice the updated value of the lock** as cache coherence will maintain that changes made to shared memory is propagated across the private caches of all processors. 

## 10. Spinlocks with Delay 

There is another approach to reducing contention for a given lock: implementing a delay. Each processor will wait before it contends for a lock, even if the processor detects that the lock is available. 

### Delay after Lock Release 

```cpp
while ((L == locked) || (test_and_set(L) == locked)) {
	while (L == locked); // locally spinning until the lock has been released
	delay(d[P_id]); // once lock is released, we delay for some time assigned to processor ID
}
```

Here, the delay is a function of processor ID, meaning that each processor delays lock acquisition for a different amount of time. However, because this is a static delay time, some time can be wasted on the delay process. We can, instead, perform a dynamic delay. 

### Delay with exponential backoff (?) 

```cpp
while (test_and_set(L) == locked) {
	delay(d);
	d = d * 2;
}
```

When the lock is not highly contended for, the processor will not delay for very long. But with repeated checks (and failures to acquire) the lock, the processor will delay the acquisition for a little bit longer. 

### Summary 

Because these spinlock algorithms do not require cache access, they can be implemented on a non-cache coherent processor. Generally speaking, if there is a lot of contention, then static assignment of delay might be better than exponential delay with backoff--but in general, any delay implementation improves a naive spinlock algorithm. 
## 11. Ticket Lock

If multiple processors are waiting on the lock, then the lock should be acquired by the processor which has waited the longest amount of time. However, the spinlock does not keep track of how long each thread has been waiting for the lock--the threads are **indistinguishable** from one another. Therefore, spinlock **does not preserve fairness.**
### Ensuring fairness in lock acquisition 
The ticket lock algorithm is simply the implementation of a ticketing system. 
```cpp
struct lock { // add new data fields to the lock
	int next_ticket;
	int now_serving;
};
release_lock(L) {
	L->now_serving++; // every thread performs a release_lock after it is done
}
acquire_lock(L) {
	int my_ticket = fetch_and_inc(L->next_ticket); // acquire a lock by marking the current position. this is the "get_ticket" step of the ticket lock.
	loop: // delay like in spinlock with delay. 
		pause(my_ticket - L->now_serving); // pausing for an amount of time determined by the ticket and now_serving value
		if (L->now_serving == my_ticket) return; // acquire the lock when the ticket == now_serving
}
```

The `acquire_lock` algorithm is implemented in three steps: 
1. First, **acquire a ticket.** Here, this is done by retrieving and incrementing the next ticket in the `lock` structure. 
2. Now, **spin on the lock.** We spin until the `my_ticket` value is equal to the `now_serving` value. `now_serving` is updated every time the lock is released in the `release_lock` function. 
3. When `L->now_serving == my_ticket`, we can successfully acquire the lock. 

While the algorithm preserves fairness, every time the lock is released, the `now_serving` value is going to be updated by the cache coherence mechanism, which causes contention on the network. We have not reduced the contention that can happen on the network when a lock is released because the `now_serving` is repeatedly updated across the private caches of all the processors in the system. 
## 12. Spinlock Summary 

Our first two spinlock algorithms--read with test-and-set, as well as test-and-set with delay--reduces contention for the resource but does not guarantee **fairness**, and our final spinlock algorithm, the ticket lock, guarantees fairness but does not reduce contention on the network. 

To further illustrate the limitations of spinlock algorithms, consider the following example. Say that, in a set of N threads ($T_{1}...T_{n}$). Say that $T_1$ has acquired a lock. As per the spinlock algorithms, to some extent the remaining threads $T_2$ to $T_n$ are now waiting on the lock to be released. However, **only one thread will be able to acquire the lock after $T_1$ has released it.** Why should more than one thread contend for the lock? 

Ideally, when $T_1$ releases the lock, it will signal ONLY one other lock between $T_2$ and $T_n$ to retrieve the next thread, rather than signalling all $n-1$ threads at the same time.  It is this idea that lays the foundation for **queueing locks**. 
## 13. Array Based Queueing Lock (Andersen's lock)

![[L04_13_A.png]]

This is also known as **Andersen's lock**. 

First, associated with each lock $L$ is an **queue of flags** `flags`. The size of this queue `flags` is equal to the **number of processes in the SMP**. If you have an $N$-way multiprocessor, then you have $N$ elements in the circular flags array. This flags array serves as a circular queue for enqueueing the requesters that are requesting access to lock $L$. 

Each element in this `flags` queue can be in one of two states: 
1. `has_lock` - whichever processor that happens to be waiting on this slot is waiting on a particular slot has the lock $L$.
2. `must_wait` - if the processor can only use `must_wait` as an entrypoint into the `flags` queue, then this processor must wait on the lock $L$. 

There can be exactly **one** processor that can be in the `has_lock` state, while all other processors are in the `must_wait` state. This is because the lock $L$ is mutually exclusive. In order to initialize the lock, we have to initialize the queue data structure `flags`, which marks **one** slot as `has_lock` while marking the others as `must_wait`. 

One important note: the slots are **not** statically associated with any processor. While there is a unique spot available **for every waiting processor**, they are not statically assigned to the processor at run-time. 

## 14. Array Based Queueing Lock (cont) 

### The `queuelast` variable

The `queuelast` variable points at the next open slot in the `flags` queue. Each time a processor requests a lock, the `queuelast` variable is incremented once. 

|             |                                      |                                                          |      |      |      |      |      |      |      |
| ----------- | ------------------------------------ | -------------------------------------------------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| **idx**     | 0                                    | 1                                                        | 2    | 3    | 4    | 5    | 6    | 7    | 8    |
| **state**   | `HL`                                 | `MW`                                                     | `MW` | `MW` | `MW` | `MW` | `MW` | `MW` | `MW` |
| **pointer** | $P_1$<br>current <br>lock <br>holder | `queuelast`<br>future requ<br>-estors must<br>queue here |      |      |      |      |      |      |      |

Say that some processor $P_x$ arrives and requests the same lock. Every time that some processor $P_x$ requests lock acquisition, we update the **pointer** and the `queuelast` variable to point to the next space in the array that the next contending processor will occupy. 

|             |                                      |       |           |           |            |                                                          |      |      |      |
| ----------- | ------------------------------------ | ----- | --------- | --------- | ---------- | -------------------------------------------------------- | ---- | ---- | ---- |
| **idx**     | 0                                    | 1     | 2         | 3         | 4          | 5                                                        | 6    | 7    | 8    |
| **state**   | `HL`                                 | `MW`  | `MW`      | `MW`      | `MW`       | `MW`                                                     | `MW` | `MW` | `MW` |
| **pointer** | $P_1$<br>current <br>lock <br>holder | $P_x$ | $P_{x+1}$ | $P_{x+2}$ | $P_{mine}$ | `queuelast`<br>future requ<br>-estors must<br>queue here |      |      |      |

```cpp 
Lock(L){
	myplace = fetch_and_inc(L->queuelast); // mark my place in the array
}
```

The `Lock` algorithm for the array-based queueing lock will follow as such. 
When we make a lock request, we mark our place in the `flags` array using an atomic instruction (to prevent race conditions between contending processors), `fetch_and_increment` on the `queuelast` variable. By calling `fetch_and_inc` on the `queuelast`, we not only retrieve our own place in the queue (through the `fetch` operation), but we also increment the `queuelast` variable to point to the next available spot in the array. 

If the architecture does not support `fetch_and_increment`, we will have to simulate the operation using `test_and_increment` instructions instead. 

Once we have marked our position in the `flags` array, we will now **wait for our turn**. In other words, we are spinning on our thread's assigned slot until the state changes from `must_wait` to `has_lock`. 

```cpp
Lock(L){
	myplace = fetch_and_inc(L->queuelast); // mark my place in the array
	while(flags[myplace % N] == MW); // stop spinning once MW becomes HL
}

```
## 15. Array Based Queueing Lock (cont) 
### Topic

What happens when the lock is released? Let's take a look at the `unlock` operation. 

```cpp
Lock(L){
	myplace = fetch_and_inc(L->queuelast); // mark my place in the array
	while(flags[myplace % N] == MW); // stop spinning once MW becomes HL
}

unlock(L){
	flags[current % N] = MW; 
	flags[(current + 1) % N] = HL;
}
```

What happens here is that when $P_1$ is finished working on the lock, it will call the `unlock` operation on the lock, which will do two things: 
1. Changes `flags[P_1]->state` to `MW`
2. Changes `flags[P_1 + 1]->state` to `HL`

This updates our array to the following state: 

|             |           |                                     |           |           |            |                                                          |      |      |      |
| ----------- | --------- | ----------------------------------- | --------- | --------- | ---------- | -------------------------------------------------------- | ---- | ---- | ---- |
| **idx**     | 0         | 1                                   | 2         | 3         | 4          | 5                                                        | 6    | 7    | 8    |
| **state**   | `MW`      | `HL`                                | `MW`      | `MW`      | `MW`       | `MW`                                                     | `MW` | `MW` | `MW` |
| **pointer** | $P_1$<br> | $P_x$<br>current <br>lock<br>holder | $P_{x+1}$ | $P_{x+2}$ | $P_{mine}$ | `queuelast`<br>future requ<br>-estors must<br>queue here |      |      |      |

Recall that the `flags` array is a **circular** queue. This means that the state of `HL` is 'circled' around all of the available slots of the array from start to finish. When the predecessor of $P_{mine}$ acquires the lock, the state of the array looks like: 

|             |           |       |           |                                        |            |                                                          |      |      |      |
| ----------- | --------- | ----- | --------- | -------------------------------------- | ---------- | -------------------------------------------------------- | ---- | ---- | ---- |
| **idx**     | 0         | 1     | 2         | 3                                      | 4          | 5                                                        | 6    | 7    | 8    |
| **state**   | `MW`      | `MW`  | `MW`      | `HL`                                   | `MW`       | `MW`                                                     | `MW` | `MW` | `MW` |
| **pointer** | $P_1$<br> | $P_x$ | $P_{x+1}$ | $P_{x+2}$<br>current<br>lock<br>holder | $P_{mine}$ | `queuelast`<br>future requ<br>-estors must<br>queue here |      |      |      |

Once $P_{x+2}$ performs the `unlock`, it will set the state of the slot $P_{mine}$ to `HL`, indicating that the processor $P_{mine}$ has now acquired the lock and can enter the critical section to perform its necessary modifications. 

### Advantages of Array-Based Queueing Lock 

#### 1. There is only one atomic operation performed per critical section.
Only one atomic operation, `fetch_and_increment`, needs to be executed in order to acquire the lock. 

#### 2. Fairness is preserved. 
Lock acquisition is provided sequentially based upon the order of threads entering the `flags` array.

#### 3. Reduced network contention. 
Threads spin on their own private copy of their own variable in their own caches. There is less network contention as only one thread is notified when the lock is released (as opposed to a large pool of threads), compared with the ticket lock algorithm. 

### Disadvantages of Array-Based Queueing Lock 

#### 1. The size of the data structure is the same size as the number of processors in the multiprocessor. TLDR: Excess space complexity.

Space complexity for this algorithm is O(N) for *every* lock you have in the program. A large-scale multiprocessor with dozens of processors can lead to excessive memory overhead. 

In any well-structured multi-threaded program, even though we may have lots of threads executing in lots of processors, but only a small subset of these processors may contend for a lock. However, **this algorithm nevertheless anticipates the worst-case**, and **instantiates a data structure that may far exceed the size of the subset of processors** that contend for the lock. Do note that this is a result of using a **static**, and not a dynamic, data structure to maintain the sequence of threads. 
## 16. Link Based Queueing Lock (MCS lock)

A link-based queueing lock, or the MCS lock, utilizes a **linked-list** representation for our queue. We begin with the following implementation of the `qnode` data structure: 
```cpp
class qnode {
	qnode next; // points to the successor
	bool locked; // whether or not processor acquired the lock 
}
```

Every processor that requests access to a lock creates a new instance of `qnode`. 
`locked` indicates whether or not the processor has acquired the lock, while the `next` field points to either `null` or the next processor that has requested access to the lock, and our lock is also of type `qnode`. 

### Scenario 1 - Empty Linked List 
![[L04A_02_03.png]]

If no requests for the lock have been issued yet, our linked list will look like: 

```
[!LOCK] -> null
```

Here, the `->` symbol represents the `next` pointer. 
### Scenario 2 - Single Request 
![[L04A_02_03.png]]
If only a single request has been made to acquire the lock, the linked list will take on the following updated appearance: 

```
[LOCK] -> [P1 (R)] -> null
```

In this scenario, processor 1 has acquired the lock, and so it is running (as indicated by the `R` symbol). More specifically, processor 1 has created an instance of `qnode`, and set its `next` field to `null`, to indicate that there is no one after it. At the same time, it has set the `lock`'s `next` pointer to itself. Thus, processor 1 can now access the critical section. 
## 17. Link Based Queueing Lock (cont)
### Scenario 3 - Waiting on a request 

![[L04A_02_04.png]]

Say that while processor 1 has acquired the lock, another processor 2 has requested access next. 
Processor 2 will **have to update the `next` pointers** of all the processors in the queue, as well as that of the `next` pointer in the `lock`. 
We have to update the `next` pointer in processor 1 to point at the next processor 2, so that processor 1 will signal processor 2 once it has released the lock, and we update the `next` pointer in `lock` to point at `P2`. Here are the contents of each `qnode` object at this point: 
```
LOCK: 
	next = P2 
	locked = True

P1: (running)
	next = P2 
	locked = True 

P2: (spinning)
	next = NULL 
	locked = False
```

The `next` pointer in `lock` is always pointing to the **last** member of the linked list queue. Once `P2` has made these necessary changes, then it can continue to spin and wait for the previous processor `P1` to signal that it has released the lock. 
## 18. Link Based Queueing Lock (cont) 
### The Lock Algorithm

A number of 

```cpp
class Lock(){
	
}
```

## 19. Link Based Queueing Lock (cont) 
### Topic

## 20. Link Based Queueing Lock (cont)
### Topic