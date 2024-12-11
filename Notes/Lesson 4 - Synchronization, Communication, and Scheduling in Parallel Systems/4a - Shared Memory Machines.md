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

![[Pasted image 20241211110212.png]]

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

We expect that the program order will be maintained by the execution on that processor. In addition to that, there is an interleaving of memory accesses between $p_1$ and $p_2$. However, we have no way of controlling the interleaving, and so the interleaving of memory accesses can be defined as arbitrary. These are the two properties of the sequential consistency model. 

Note to self--include notes on sequential consistency from HPCA. This explanation isn't sufficient. 


## 5. Memory Consistency and Cache Coherence

## 6. Hardware Cache Coherence 

## 7. Scalability 