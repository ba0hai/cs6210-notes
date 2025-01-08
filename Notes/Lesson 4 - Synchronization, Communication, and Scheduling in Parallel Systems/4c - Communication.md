## 1. Barrier Synchronization 

Recall that **barrier synchronization** is where $N$ threads need to know with respect to one another and wait at a barrier until all $N$ threads have arrived. This type of synchronization is very popular in scientific applications. 

A very simple implementation of the **centralized**, or **counting barrier**, is implemented below. 

### Simple centralized barrier 

```python
count = N # N = the number of threads that must arrive at the barrier 
decrement(count) # atomic instruction to decrement 
if count == 0: # Nth thread will reach this 
	count = N # set count = N
	break
else: 
	while count > 0:
		spin # the N - 1 threads will spin here
```

However, there is one significant problem with this algorithm. 

## 2. Quiz - Problems with Algorithm 

Recall that the $N$th processor must **decrement** the count atomically, before setting the count equal to $N$. At this point in the code: 
```python
decrement(count)
if count == 0: # here
	count = N 
```

It is likely that the other threads will move to the next barrier. In other words, once the other threads that were spinning on `while count > 0` detect that `count == 0`, they will move onto the next iteration of the barrier before the $N$th processor. 

## 3. Counting Barrier 

```python
count = N 
decrement(count) # other spinning threads will be signaled here,
if count == 0: 
	count = N # but we want them to be signaled here.
	break
else: 
	while count > 0:
		spin 
```

In short, we want to signal the remaining spinning threads **after** we have set `count = N`, not after we have decremented the `count` variable. In other words, **the other threads may not leave the barrier before the count is reset.** 

We can add another spin loop that will have our threads continue to spin before the `count` variable is set to $N$ like so: 

```python
count = N 
decrement(count) 
if count == 0: 
	count = N 
	break
else: 
	while count > 0: # once we have detected that the count = 0,
		spin 
	while count != N: # wait until it is = N. 
		spin 

```

However, there are now **two spin loops** for every barrier in the counting algorithm. We would prefer, ideally, to have only **one** spin loop for every barrier in our counting algorithm. This is where we consider the implementation of the **sense reversing barrier.**

Recall that in the counting barrier, our two spinning episodes were: 
1. Spin and wait for the count to become 0 
2. Spin and wait for the count to be set to $N$. 

The sense-reversing barrier will remove one of these episodes, specifically the first one, which is spinning and waiting for the count to become 0. In addition to the count, there is a **sense variable** that is shared by all of the processes that want to accomplish barrier synchronization. 

The sense variable will be true for **one** barrier episode, and **false** for another. 

![[L04C_03_01.png]]

Because we have one barrier at a time, the sense variable's value will alternate between barriers. 
## 4. Sense Reversing Barrier 

When a thread arrives at a barrier, it will: 
- decrement the count (just like in the counting barrier), and 
- spin on the **sense reversal**. 

Let's say that we are executing a true barrier, where the sense flag is set to true. Every thread that spins during the execution of the true barrier will thus spin on the sense flag to be set to false, which will indicate that all threads are ready to move to the next instance of the barrier. 

The $N$th thread present will: 
- reset count to $N$, and 
- reverse the sense variable. 

The sense variable becomes the signal for all threads to move on to the next instance of the barrier. 

The centralized barrier is simple and intuitive, while the sense-reversing barrier is an improvement on the centralized barrier in that it has reduced the number of spinning episodes from 2 to 1. The problem is, however, that there is a **single shared variable** between a large number of processors. In a large scale multi-processor, running large-scale scientific applications with many parallel threads passing through a barrier, **this produces a significant degree of network contention** on the interconnected network. Recall that **less sharing means the multi-processor is more scalable.** How can we reduce the amount of sharing taking place between shared variables, and thus create a more scalable barrier algorithm? 
## 5. Tree Barrier 

Limit the amount of sharing to a small number of processes $K$. Given $N$ processors that want to perform barrier synchronization, break them up into groups of $K$ processors. This will be a **tree solution**, which means that there will be $log_{K}(N)$ levels in the tree in order to achieve the barrier. That is, given 8 processors and $K=2$, there will be $log_{2}(8)=3$ total levels in our tree barrier. 

![[L04C_03_02.png]]

What happens when we arrive at a barrier? 

![[L04C_03_03.png]]

On a micro level, the algorithm works exactly like the sense-reversal algorithm--that is, if these two processes share a data structure (the `count` and `locksense` variable), then if $P_1$ arrives at the barrier, it will decrement the `count` variable. 

The `count` variable corresponds to the `K` processes synchronizing between each other, so it will initialized at `2`. $P_1$ will decrement `count` from `2` to `1`, exactly as it would during the sense-reversal algorithm.

Some time later, $P_0$ arrives at the barrier and decrements `count` to 0. Note, however, that $P_0$ does not flip the sense flag, as **the barrier has not been completed yet**. $P_0$ will thus **move up the tree**. 

![[L04C_03_04.png]]

Note that $P_0$ alone moves up the tree to level 1, while $P_1$ remains in level 0 waiting for the sense variable to diverge. Recall that the sense flag will only be flipped when ALL other processors have arrived at the barrier.  All that $P_0$ will do right now is decrement the count, see that it is 0, and move up from level 0 to level 1 of the tree. 

![[L04C_03_05.png]]

Remember that the count and locksense variables at level 0 are shared by 2 processes: $P_0$ and $P_1$. But now that $P_0$ has moved up to the next node in the tree, it is at level 1, which is being shared by 4 processes: $P_0$, $P_1$, $P_2$, and $P_3$. The count at lock 1 is decremented from 2 to 1, and waiting for the one other process--any of $P_1$, $P_2$, or $P_3$ -- to arrive at the same node. 

## 6. Tree Barrier (cont) 

Eventually, $P_3$ will arrive at its corresponding barrier on level 0, decrement the `count` to 0, and then move up to level 1. 

### Arriving at Level 1

![[L04C_03_06.png]]

Eventually, after $P_0$, $P_1$, and $P_2$ have arrived at the barrier on level 1, what remains is $P_3$ moving up to the same barrier. Once $P_3$ has reached the barrier, it will decrement the `count` variable to 0 and finally switch the `locksense` from `false` to `true`, before waiting for the other processors at the barrier on level 2. 

![[L04C_03_07.png]]

On the other side of the barrier, $P_4$ and $P_7$ have arrived at their corresponding barriers on level 0. $P_6$ is last to arrive to the corresponding barrier on level 0, so it is the first processor to move to level 1, where it has decremented the count and is now waiting on the arrival of processors $P_4$, $P_5$, and $P_7$. 

**Which processor will be the second one to arrive at level 1 after $P_6$?**  Naturally it will be $P_5$, as it has yet to reach its barrier at level 0. The **last** processor to arrive at some given level $K$ will be the **first** processor in that barrier to move to the barrier at level $K+1$. Eventually, $P_5$ will reach level 2 first, and decrement the count. 

![[L04C_03_08.png]]

After $P_5$ has decremented the `count` variable at level 2 from 1 to 0, this indicates that **all of our processes have reached the same barrier.** As such, it is now time to notify every processor that it is time to move to the next barrier. 

Let us review the steps that a processor must take when it arrives at a barrier. 

1. $P_x$ arrives at a barrier on level 0. 
2. $P_x$ decrements the `count` at the barrier: `barrier->count --`. 
	1. If `count != 0`, spin on the `locksense` flag. 
	2. But if `count == 0`, check for `barrier->parent`. 
		1. If there is a `barrier->parent`, recurse. 
		2. If there is no parent, recursion ends because **we are at the root of the tree**. 

Therefore, when at the root of the tree barrier, if `count==0`, this means that **all processors have reached the barrier**. 

## 7. Tree Barrier (cont) 
### Wake up other processors 
![[L04C_03_09.png]]

$P_5$ will now flip the `locksense` flag at the root of the tree. What happens after the `locksense` flag is flipped? 

First, recall that $P_3$ is spinning on the `locksense` on level 2. Once the `locksense` flag is flipped, $P_3$ will be released from its spin cycle. Wakeup starts from the root. In this case, $P_3$ and $P_5$ will now flip the `locksense` variables at level 1, which will release the processors that are spinning on the `locksense` at every lower level of the tree. 

At level 2, there are $K-1$ processors waiting--$P_3$ was waiting to be released by $P_5$. At level 1, there are $K * (K - 1)$ processors waiting to be released. $P_0$ and $P_6$ are both waiting to be released by $P_3$ and $P_5$. And at level 0, there are $K^2 * (K-1)$ processors waiting to be released. This makes sense--$P_1$, $P_2$, $P_4$, and $P_7$ are all spinning on their respective `locksense` variables, waiting to be released by the other processors above them. 

Once the corresponding `locksense` flag has been flipped at every level, all of the other processes will be released and this stage of the tree barrier is complete. 

## 8. Tree Barrier (cont) 

The tree barrier is a fairly intuitive recursive algorithm that builds on the same centralized sense-reversal barrier. It breaks up all of the processes into groups of size $K$ in order to reduce contention on shared resources across all of the processors. It also allows for scaling up to a large number of processes, since the amount of sharing is limited to the size of $K$. 

There are, however, many drawbacks of the tree barrier. 

### Drawback 1 - Non-static spin location 

![[L04C_03_10.png]]

Note that the spin location is not statically determined for each processor. For instance, if tyou take this particular execution above, $P_0$ happens to arrive later than $P_1$. When $P_1$ arrived at level 0, it spins on `locksense`, but $P_0$ will move up. In another execution of this program however, $P_0$ could be the first processor to arrive at a given barrier, and $P_1$ will be the processor that moves up to the next barrier. **The spin location is thus dynamically defined by the arrival pattern established by some given run of the program** and can change on subsequent program executions. 

### Drawback 2 - Ary-ness of a tree 

Additionally, if a given tree is 4-ary, or 8-ary, or even greater, the amount of contention for shared data structures will be significant, which leads to more contention on the network. 

### Drawback 3 - Cache Coherence 

In a non-cache coherent multiprocessor, the spin variable that we have to associate with a specific processor may take place on a **remote** memory, and not a local, private one allocated to our processor's cache. Recall that a distributed shared-memory architecutre, or a NUMA (non-uniform memory architecture), means a processor accessing a variable on local memory will always be faster than a processor's access to a variable on remote memory. 

If spinning must be performed on remote memory in a non cache-coherent system, this leads to much higher contention for the shared variable over the network. 

## 9. 4-ary Arrival 

The MCS tree, or the 4-ary arrival tree, is a 4-ary tree. There are two data structures associated with every parent, and this data structure is `haveChildren` and `childNotReady`. 

`haveChildren` (HC) is a **vector** data structure associated with every node.

![[L04C_03_11.png]]

For example, $P_0$ has 4 children, $P_1$ $P_2$ $P_3$ and  $P_4$ . If we look at $P_1$, it has 3 children--$P_5$, $P_6$, and $P_7$. Thus, we have a total of 8 processors. 

The `haveChildren` vector contains a `true` value when a child occupies 1 of the available 4 allotted child nodes of a given node in the tree, and `false` when no child occupies the child node. It goes without saying that the `haveChildren` vector is false for all nodes $P_2$ to $P_7$. 

The `childNotReady` data structure is a way by which each process has a unique spot in the parent to signal their arrival to a barrier. 

![[L04C_03_12.png]]

For each child, there is a unique spot to indicate their arrival to a barrier, and it is stored in the `childNotReady` vector. 

The black arrows in this structure shows us the arrangement of the trea, and the parent-child relationship for the 4-ary arrival tree. However, the red arrows indicate the specific spot where a particular child will indicate their arrival status to the parent. The 4th spot is empty in $P_1$ which indicates that there are only 3 children nodes that $P_1$ must wait for, rather than $4$. 

### Algorithm 

The algorithm for barrier arrival will work like this. When each of these processes arrive at a barrier: 
1. The process will access its allocated, statically-determined location in the `childNotReady` data structure belonging to the parent. 
2. The parent node will check whether the spots in the `childNotReady` vector has been populated. 
3. Once all children have arrived at the barrier, then similar to the original tree barrier algorithm, $P_x$ will move up by accessing its parents `childNotReady` data structure and setting its corresponding bit. 

The root node, $P_0$, is waiting on all of its children $P_1$, $P_2$, $P_3$, and $P_4$, by spinning on the `childNotReady` data structure and waiting for each spot to be populated by its children upon arrival at the barrier. Because of the arrangement of the data structure, all of $P_0$'s children know what the statically-determined index of the `childNotReady` data structure to populate. 

Note that **each processor is assigned a unique location** due to the nature of this algorithm. Additionally, on a cache coherent infrastructure, parent nodes only have to spin on a 1-word memory location that corresponds to `childNotReady`, not necessarily on four separate discrete memory locations.

## 10. Binary Wakeup 

![[L04C_03_13.png]]

Note that, while the MCS tree barrier is a 4-ary tree, the wakeup tree that corresponds to the MCS tree barrier is a **binary** tree. 
Once again, every processor is assigned a unique location. The data structure used in the wake-up tree is a `childPointer` data structure, which is a way by which a parent can reach down into the children and indicate that it is time to "wake up". 

Depending on the particular location in this wakeup tree, they may or may not have children--for example, $P_0$ has two children, $P_3$ has only 1 child, and $P_4$, $P_5$, $P_6$, and $P_7$ do not have children. Each of these processes is also spinning on a unique, **statically-determined** location, where they will receive a signal upon wake-up. 

The key point here is that in the construction of this tree, all processes are assigned a position in the tree, and are associated with a statically-determined memory location. 

![[L04C_03_14.png]]

In the above image, the red arrows represent these statically-determined memory locations associated with a given process in the binary wake-up tree. Once the wake-up signal has been handed down to all of the children of the tree, we can assume that this first stage of the barrier has been completed. Through the statically-assigned memory location, contention on the network is limited. Additionally, by packing the variables into a single data structure, we are minimizing contention as well. 

## 11. Tournament Barrier 

![[L04C_03_15.png]]

Rigged tournament -- pre-determine who is the winner in this round 

$N$ players means $log_{2}(N)$ rounds 

$P_0$ will win, $P_2$ will win, $P_4$ will win, $P_6$ will win

Key rational -- if processors execute on shared memory machine, the winner will sit and wait for the process to come over 

$P_2$ will wait for $P_3$ etc 

Spin location where $P_0$ is waiting for $P_1$ to tell him he has lost the match is static 

In a shared-memory multiprocessor, the location for each of these processes , the winners so to speak, is pre-determined. Helpful when we have a NCC NUMA machine. Locate the spin location in memory close to the processors. This is the idea behind match fixing. 

![[L04C_03_16.png]]

$P_0$, $P_2$, $P_4$, and $P_6$ will advance to the next round 

$P_2$ and $P_6$ will notify that $P_0$ and $P_4$ win this round so $P_0$ and $P_4$ will advance to the next round

![[L04C_03_17.png]]


Then, $P_4$ will signal that $P_0$ has won the tournament 

![[L04C_03_18.png]]

Once $P_0$ is crowned champion, then everyone is at the barrier
## 12. Tournament Barrier (cont) 
### Wake-up

![[L04C_03_19.png]]

$P_0$ will tell $P_4$ that it is time to wake up. Revisit tournament analogy--winner shakes hands with loser once the tournament is complete. 

For some arbitrary N where N is a binary power, then at every round (of which there are $log_{2}(N)$) wakeup will take place.

![[L04C_03_21.png]]

Note that the positions of all processors are statically determined. If $P_4$ knows that $P_0$ will shake hands then $P_4$ will spin on a local variable closest to its processor. Important for NCC NUMA machines because it is more convenient for $P_4$ to be spinning on a location closest to it. 

## 13. Tournament Barrier (cont) 

### Topic

## 14. Dissemination Barrier 

### Topic

## 15. Dissemination Barrier (cont) 
### Topic

## 16. Quiz - Barrier Completion 

## 17. Dissemination Barrier (cont) 
### Topic

## 18. Performance Evaluation 

