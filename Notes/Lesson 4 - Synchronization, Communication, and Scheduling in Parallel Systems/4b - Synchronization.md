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

## 11. Ticket Lock

## 12. Spinlock Summary 

## 13. Array Based Queueing Lock 

## 14. Array Based Queueing Lock (cont) 

### Topic

## 15. Array Based Queueing Lock (cont) 
### Topic

## 16. Link Based Queueing Lock 

## 17. Link Based Queueing Lock (cont)
### Topic

## 18. Link Based Queueing Lock (cont) 
### Topic

## 19. Link Based Queueing Lock (cont) 
### Topic

## 20. Link Based Queueing Lock (cont)
### Topic