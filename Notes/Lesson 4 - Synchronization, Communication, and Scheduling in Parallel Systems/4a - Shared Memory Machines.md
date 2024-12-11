Flashback to HPCA!!! Onur Mutlu has incredible lectures on ALL of these topics. 
## 1. Shared Memory Machine Model 

There are 3 different structures for the shared memory machine. The common things in all the structures is that there will be CPUs, memory, and an interconnection network. 
The entire address space defined by the memories is also accessible from any of the CPUs. There is also a cache associated with every CPU. 

### Dance Hall Architecture 

![[L04A_01_01.png]]

This is where you have CPUs on one side, and memory on the other side of an interconnection network. 

### Symmetric Multiprocessor Architecture (SMP) 
Also known as SMP architecture, or a Symmetric multiprocessor. 

![[L04A_01_02.png]]

The interconnection network from the dancehall architecture has simplified considerably, and is now a simple bus that connects all the CPUs to talk to main memory. The access time from any of the CPUs to the memory is the same, hence it is symmetric. Note that there are individual caches attached to every CPU. 

### Distributed Shared Memory Architecture (DSM)  

![[L04A_01_03.png]]

In this distributed shared memory arch, you have a piece of memory associated with each CPU. Each CPU is able to access all of the memories through the interconnection network--however, access to the memory closer to the CPU is going to be faster than memory that is farther from a given CPU, which will require communication through the interconnection network. 

## 2. Shared Memory and Caches 

Let's discuss shared memory and private caches. 
To simplify the discussion, refer to the SMP architecture, with the system bus that connects all process communication with shared memory. 

![[L04A_01_02.png]]

The caches associated with each CPU serves the exact same purpose in a multiprocessor like this, as it does in a uniprocessor. That is, the CPU, when it wants to access some memory location, it will check if the information is present in the cache before it goes and fetches the information from main memory to store in the cache for later use. 

In a multiprocessor, this function remains exactly the same. However, ther eis a unique problem with the multiprocessor--the fact that there are private caches associated with each one of these CPUs, and the memory itself is shared across all of these processes. 

![[L04A_01_04.png]]

Let's say that there is a memory location $y$ that is currently in the private caches of all the processes. Maybe $y$ is a hot memory location. Say that process $p_1$ decides to write to the memory location $y$ in its cache. Now, $p_1$'s cache contains the memory location, $y'$ . **What will happen to the memory location $y$ that exists in the caches of processes $p2 \xleftrightarrow{} p_n$?** This is the cache coherence problem. 

If any of these processes happen to have this memory location $y$ in their private caches, they should obtain $y'$, and not the old value of $y$. **Who is responsible for maintaining cache coherence across all processes?** 

Consider the partnership between hardware and software. In other words, the hardware and software must agree on what is called the **memory consistency model.** The memory consistency model is a contract between hardware and software with regards to the behavior that a programmer can expect when writing a multithreaded application running on a multiprocessor. 

For example--in a uniprocessor, there's a compiler-writer that knows about the instruction set provided by the CPU. When the architect builds the CPU, he doesn't know how the instruction will be used, but there is an expectation that the instruction set's semantics will be adhered to in the implementation of the processor so that the compiler writer can use that instruction set to compile high level language programs. 

Similarly, when writing a multithreaded application, there needs to be a contract between the hardware and software when processors access the same memory location. 
## 3. Quiz - Processes 

Assume that you have two shared memory locations $a$ and $b$, both of which are initialized to 0. Assume that you have access to two other shared memory locations $c$ and $d$. 
Assume also that there are two processes $p_1$ and $p_2$, each of which are running on different processors, and are running the following code snippets:

|        | Process $p_1$ | Process $p_2$ |
| ------ | ------------- | ------------- |
| Line 1 | `a = a + 1;`  | `d = b;`      |
| Line 2 | `b = b + 1;`  | `c = a;`      |
|        |               |               |

We know that both processes are operating completely independently of each other. However, these memory locations that processes $p_1$ and $p_2$ are accessing are all shared memory locations. We don't know the relative ordering between all the instructions that are executed between $p_1$ and $p_2$. In uniprocessor programs, there is the expectation that instruction order is exactly what the processor is executing. But in multiprocessor programs, relative ordering of instructions between processes needs to be defined. 

In this example, what are the possible values you expect to see for $d$ and $c$ after processes $p_1$ and $p_2$ have completed execution? 
- $c=d=0$
- $c=d=1$
- $c=1$, $d=0$
- $c=0$, $d=1$

---

- $c=d=0$ `CORRECT`
	- It is possible for process $p_2$'s instructions to complete before process $p_1$.
- $c=d=1$ `CORRECT`
	- It is possible for process $p_1$'s instructions to complete before process $p_2$.
- $c=1$, $d=0$ `CORRECT
	- It is possible for process $p_1$ to execute line 1, followed by $p_2$ executing lines 1 and 2, before $p_1$ executes line 2. 
- $c=0$, $d=1$ `POTENTIALLY CORRECT`
	- If $d=1$, this would mean that line 2 on $p_1$ has finished executing. However, if $c=0$, this would mean line 1 on $p_1$ hasn't finished executing--but how can this be possible? Recall that communication between these processors is taking place on a shared bus. If messages between the processors go out of order, then yes--this can potentially take place. However, **this is not intuitive**, and should **not be permitted by the memory consistency model**. The agreement between the programmer and the system should not allow for this outcome to take place. 

## 4. Memory Consistency Model 

Assume that you are given the following memory accesses to $a, b, c$ for two different and completely independent processes. 

|     | Process $p_1$ | Process $p_2$ |
| --- | ------------- | ------------- |
| 1   | `read(a)`     | `write(a)`    |
| 2   | `write(b)`    | `read(b)`     |
| 3   | `write(c)`    | `write(c)`    |
| 4   | `read(b)`     |               |

There is no guarantee made thus far for the ordering of these accesses on main memory. Therefore, it is completely possible that in one execution of $p_1$ and $p_2$, the write access in line 1 process $p_2$ performed on memory location $a$ happens after reading memory location $a$ takes place on $p_1$. 

The ordering on the accesses happening in line 1 are acceptable to the programmer, but the programmer needs to know what to expect for these memory accesses. Therefore, we must implement the memory consistency model. 

### Sequential Consistency Model 

One expectation we have as a programmer is that the accesses we have on a particular processor is going to be exactly the order in which we have written it. For example, for $p_1$, a write on memory access $b$ in line 2 will be read in the read access on memory location $b$ at line 4. This is the program order. 

We expect that the program order will be maintained by the execution on that processor. In addition to that, there is an interleaving of memory accesses between $p_1$ and $p_2$. However, we have no way of controlling the interleaving, and so the interleaving of memory accesses can be defined as arbitrary. 

These are the two properties of the sequential consistency model. 

#### Constraints for Sequential Consistency 

There are really two constraints for sequential consistency. The first is the program order requirement, which means that it must appear as if the memory operations of a process become visible—to itself and others—in program order. 

The second requirement, needed to guarantee that the total order or interleaving is consistent for all processes, is that the operations appear atomic; that is, it appear that one is completed with respect to all processes before the next one in the total order is issued. 

The tricky part of this second requirement is making writes appear atomic, especially in a system with multiple copies of a block that need to be informed on a write. **Write atomicity**, included in the definition of sequential consistency above, implies that the position in **the total order at which a write appears to perform should be the same with respect to all processors**. 

It ensures that nothing a processor does after it has seen the new value produced by a write becomes visible to other processes before they too have seen the new value for that write. 

In effect, while coherence (write serialization) says that writes to the same location should appear to all processors to have occurred in the same order, **sequential consistency says that all writes (to any location) should appear to all processors to have occurred in the same order**. 

So, in order to preserve sequential consistency, there are three conditions that must be sufficed: 

1. Every process issues memory requests in the order specified by the program. 
2. After a `write` operation is issued, the issuing process **waits for the write to complete** before issuing its next operation. 
3. After a `read` operation is issued, **the issuing process does the following**: 
	1. It **waits for the read to complete,**
	2. and it **waits for the write whose value is being returned by the read to complete** before issuing the next operation. That is, if the write whose value is being returned has performed with respect to this processor, then the processor should wait until the write has performed with respect to **all processors**. 

We can illustrate the third condition with the following image: 
![[L04A_01_05.png]] 

Notice that `write A` must be complete before  `read A` is executed. Additionally, `write B` must complete before `read B` can execute. However, `write A` can happen before `write B`, and `read A` can happen before `read B`. 
## 5. Memory Consistency and Cache Coherence

Let us return to our earlier example. 

|        | Process $p_1$ | Process $p_2$ |
| ------ | ------------- | ------------- |
| Line 1 | `a = a + 1;`  | `d = b;`      |
| Line 2 | `b = b + 1;`  | `c = a;`      |
|        |               |               |

- $c=d=0$
- $c=d=1$
- $c=1$, $d=0$
- $c=0$, $d=1$

Once again, we see that choices 1 to 3 are possible. However, the 4th choice is **impossible** under sequential consistency. Memory consistency models are what the application programmer needs to be aware of to develop code with the knowledge that it will execute correctly on a shared memory machine. As operating system designers, we  need to make sure that code **runs quickly**. In order to do so, we have to understand how to implement the consistency model efficiently. 

![[L04A_01_06.png]]

If memory consistency represents the shared memory model presented to the programmer, then cache coherence is how the system is implementing the model in the presence of private caches. In order to make sure that the consistency model is implemented correctly by the cache coherence mechanism, we have to understand that cache coherence is a hardware/software trade-off. 

For example, in non-cache coherent (NCC) multiprocessor systems, hardware may be giving shared address space, but not a guarantee that caches are coherent. There is a shared address space available for all the processors, and there is private caches for holding data from main memory, but if data is modified, then system software must maintain cache coherence. 

Alternatively, hardware can provide the shared address space, but also maintain cache coherence between processors--this is a CC processor. 
## 6. Hardware Cache Coherence 

There are two possibilities for hardware cache coherence, which is a write-invalidate and/or a write-update scheme. (Remember HPCA?) 

Assume once again that this is taking place on a Symmetric Multiprocessor Architecture (SMP).  

### Write-invalidate 

### Write-update 



## 7. Scalability 

![[L04A_01_07.png]]
**Parallelism isn't perfect.** While the expectation that more processors leads to improved performance, there is also overhead in terms of maintaining cache coherence when you have sharing happen between shared data. The advantage in adding more processors is the ability to exploit parallelism, but the disadvantage is the increased overhead incurred by the system due to increased communication between the processors for maintaining cache coherence. 

The actual performance of a shared memory machine falls somewhere between the expected performance and the overhead incurred by the processors. Note, however, that these trends are not always linear.

In short--reduce shared information between threads as much as possible if you want a performant shared memory machine. **Shared memory machines scale well when you don't share memory.** As operating system designers, we have no control over what the application programmer does. However, all operating system designers can do is ensure that user shared data structures are kept at a minimum in the implementation of the operating system itself.

