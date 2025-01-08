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

In the tournament barrier, there are N players with two players playing against one another across a number of rounds. $N$ players, with two players pitted against one another in every round, means $log_{2}(N)$ rounds. 

The tournaments, however, are **rigged**. How? Winners in each round are **predetermined**. In this case: 
![[L04C_03_16.png]]

- $P_0$ will win against $P_1$,
- $P_2$ will win against $P_3$,
- $P_4$ will win against $P_5$, 
- and $P_6$ will win against $P_7$.

Key rational behind match-fixing is this: if these processors execute on shared memory machine, the winning processor can simply **sit and wait** until it is signaled to have won the match. In the first match, for example, as $P_0$ is waiting for $P_1$ to signal that $P_0$ has won, it is **spinning on a static location** as it waits. **This is especially beneficial in a non cache-coherent NUMA machine**, because since the spin location for each of these "winning" processors is pre-determined, the spin location can be found in memory close to the winning processors on these NCC NUMA machines. 

As we continue with our first example, our winning processors–$P_0$, $P_2$, $P_4$, and $P_6$–will advance to the next round.

![[L04C_03_17.png]]

Once more, these matches are fixed, so $P_0$ and $P_4$ are actually just spinning in their pre-determined locations, waiting for the signal that they have won. Once $P_2$ and $P_6$ signal that $P_0$ and $P_4$ win this round, $P_0$ and $P_4$ will advance to the next round. 

![[L04C_03_18.png]]

$P_0$ will, once again, spin on its predetermined location, until $P_4$ has indicated that $P_0$ has won. This will propagate until $P_0$ is crowned champion. 

The important performance consideration from the tournament barrier is that **the spin locations** for each of the processes waiting for the winning signal is **statically determined** on every level. 
## 12. Tournament Barrier (cont) 
### Wake-up

![[L04C_03_19.png]]

$P_0$ will tell $P_4$ that it is time to wake up. Revisit tournament analogy--winner shakes hands with loser once the tournament is complete. For some arbitrary N where N is a binary power, then at every level, for a total of $log_{2}(N)$, a wakeup will take place.

![[L04C_03_21.png]]

Recall that **the positions of all processors are statically determined**. If $P_4$ knows that $P_0$ will shake hands upon wakeup, then $P_4$ will spin on a local variable closest to its processor. $P_0$ and $P_4$ then go down to the next level, and **shake hands with every processor** on each level. The **winners of every match** go around and wake up the **losers of every match**. 

Once **all processors have experienced the wakeup**, then the processors can move onto the **next phase** of the barrier. 

The primary takeaways from the tournament barrier algorithm is: the arrival **moves up the tree with match-fixing**, and the winners of each fixed match **spins on a statically determined location.** Similarly, the losing processor of each match will spin on a statically determined location **until the winning processor 'shakes hands' to wake them up.** 
## 13. Tournament Barrier Continued

There's a lot of similarities between the Tournament algorithm, the sense-reversal tree algorithm and the MCS algorithm. So let's discuss the **differences** between the tree barrier and the tournament barrier first. 

### Tree Barrier vs Tournament Barrier 

First, in the tournament barrier, **the spin locations are statically determined** whereas in the tree barrier, **the spin locations are dynamically determined** based on who arrives at a particular node in the barrier in the tree in that algorithm. 

Another important difference between the tournament barrier and the the tree barrier is that **there is no need for a fetch-and-free operation**. Because all that's happening at every level. 

Additionally, at every round of the tournament, there is spinning taking place. Spinning is reading, while signaling is writing. **As long as we have atomic read and write operations in the multi-processor that's all we need in order to implement the tournament barrier**. But in the tree-barrier, there must a **fetch-and-free** operation in order to atomically decrement the count variable. 

Now what about the total amount of communication that takes place? Communication amounts are similar due to the tree arrangements in both tree and tournament barriers. As you go up the tree, the amount of communication between nodes will decrease, because the tree is being pruned towards the root, so **the amount of communication in the tournament barrier** is exactly similar to the tree barrier–that is, $O(log_{2}(N))$. 

Now the other important thing that that I should mention is that at every round of the tournament you can see that there there's quite a bit of communication happening. In the first round going up the tree P1 is communicating with P0 P3 with P2 and so on. All of these red arrows are **parallel communications** that potentially **take advantage of any inherent parallelism** in the interconnection network. 

Finally, **the tournament barrier works even if the processor is not a shared-memory machine.** All that is required for the nodes to communicate is **message-passing**–there is no need for shared-memory communication between nodes. Even if the processor is a multiprocessor cluster, or a set of processes in which the only way they can communicate with one another is through message passing, is no shared memory no physical shared memory, the tournament barrier will work perfectly fine to implement the barrier algorithm. 

### MCS Algorithm vs Tournament Barrier

Now let's make a comparison of tournament to to MCS. Now because this tournament is arranged as a tournament there are only two processes involved in this communication at any point of time in the parallel. So this means that **the tournament barrier cannot exploit the spatial locality that may be there in the caches**. 

If you recall one of the virtues of the MCS algorithm is that it could exploit spatial locality. That is, multiple spin variables can be located in the same cache line, and the parent can spin on a location where multiple children will signal the completion of their process. This is impossible for the tournament barrier. 

Similar to MCS, the Tournament Barrier does **not** need a fetch-and-free operation. 

But the tournament barrier can perform even in a non-cache coherent NUMA machine, where message-passing is the only available method of communication between processes, while the MCS algorithm requires a cache-coherent machine. 

### Aside - What is a cluster? 

A cluster is a set of nodes in the multiprocessor that don't physically share memory, and can only communicate with one another through message passing. This is important to know because clusters are the work horses for data intensive computing today. The data centers and content distribution networks, which will be discussed later in the course, use this kind of computation cluster. And these computation clusters employ on the order of thousands or tens of thousands of nodes connected together through an interconnected network where they operate as a parallel machine with only message passing as the vehicle for communication among the processes.

## 14. Dissemination Barrier

The last barrier algorithm I'm going to describe to you is what is called a Dissemination Barrier. And it works by information diffusion in an ordered manner among the set of participating processes. And what you will see is that it is not pairwise communication as you saw in the tree barriers and the NCS barrier or the tournament barrier. But it is through information diffusion. 

The other nice thing about this particular barrier the dissemination barrier is that it is since it is based on **ordered communication among participating nodes**, it's like a well-orchestrated gossip protocol. **And therefore $N$ need not be a power of 2.** 

So what's going to happen is that there's going to be information diffusion that's going to happen among these processors in several different routes. And in each round what's going to happen is:
- a processor is going to send a message to another ordained processor. 
- The receiving processor is determined by the current round.
	- So, given round $k$, $P_{i}$ will send a message to $P_{(i + 2^{k})\ mod\ N}$ .

![[L04C_03_22.png]]

We can begin our example with 5 processors labeled above. At round 0, $k=0$. 
$P_0$  is going to be sending a message to $P_{(i + 2^{k})\ mod\ N}$ . Here, where $k=0$, ${(i + 2^{k})\ mod\ N} = (0 + 2^0)\ mod\ 5 = 1$, meaning that $P_0$ sends a message to $P_1$.  

 Note that, as we work out the receiving processor for each processor, the arrangement is cyclical. 

- $P_0$ sends a message to $P_1$
- $P_1$ sends a message to $P_2$
- $P_2$ sends a message to $P_3$
- $P_3$ sends a message to $P_4$
- and $P_4$ sends a message to $P_0$. 

That if before the neighbor for him is going to be in the cyclic order whoever is the next neighbor. So in this case since there is mod function that we using before is going to be sending Its message to processor P5 mod. N N being N N being 5 it will be sending the message to P0. 

Recall that this is simply round 0 of the communication. At every round **a processor is sending a message to an ordained processor based on their own number $i$**. 

Note that all of these communications take place in **parallel**. The processes **are not** waiting on one another. But how will these guys know that Round 0 is done? 

Well if you take any particular process here let's say P2 as soon as it gets a message from P1, and it has sent a message to P3, it knows that Round 0 is finished as far as P2 is concerned it can progress to the next round of dissemination. 

So each of these processes are independently making a decision that the round is over based on two things. 
- One is they have sent a message to the peer,
- and they have received the message from the ordained neighbor that they're supposed to get it from.

After confirming these two requirements, the processor can move on to the next round. 

## 15. Dissemination Barrier Continued

![[L04C_03_22.png]]

How many communication events are taking place at every round? An order of $N$ communication events do take place at every round, since all $N$ nodes are participating in this barrier.  So now you can quickly see what's going to happen in the next round and the next round, $k$ is going to be equal to one. 

![[L04C_03_23.png]]

So in round zero for instance what we did was $P_0$ is sending a message a neighbor that is one distant from it because $k=0$. But now, in round 1, $k=1$, so $P_0$ is going to be sending a message to a neighbor that is 2 nodes from itself. For $P_0$, the receiving node will be:

$${(i + 2^{k})\ mod\ N} = (0 + 2^1)\ mod\ 5 = 2$$


$P_2$. And similarly, $P_1$ will send a message to $P_3$, and $P_2$ will send a message to $P_4$. At $P_4$, the cyclic arrangement reveals itself once again, and $P_4$ sends a message to $P_1$. Once again, an order of $N$ messages are being exchanged among these processes to indicate that this round is complete. 

Once every processor has sent a message to its ordained neighbor and received one from another neighbor, this round of the barrier is complete. For $P_2$'s case, $P_2$ will know that round 1 is over after it has received a message from $P_0$ and sent a message to $P_4$.  Only after will $P_2$ be able to move on to the next round of the dissemination barrier.

 Recall that these communications happen in parallel, so the dissemination barrier can exploit parallelism to efficiently perform this communication. 

Now, onto the next round. 

![[L04C_03_24.png]]

$k = 2$ and therefore what we're going to do is every one of these processors is going to be choosing a neighbor that is 4 nodes away. 

$${(i + 2^{2})\ mod\ N} = (0 + 2^2)\ mod\ 5 = 4$$

Now, $P_0$ will send a message to the node $P_4$. 
- $P_1$ will send a message to $P_0$, 
- $P_2$ will send a message to $P_1$, 
- $P_3$ will send a message to $P_2$, 
- and $P_4$ will send a message to $P_3$. 

Notice that **every node has now received a message $\lceil{log_{2}(N)}\rceil$ nodes**. For example, $P_0$ has now received a node from $P_1$, $P_3$, and $P_4$, which indicates that **the barrier has been reached**. 

## 16. Quiz - Barrier Completion Question

So given that there are N processors participating in a dissemination barrier algorithm, how many rounds are required before the barrier is complete? 
- $N*log_{2}(N)$
- $log_2(N)$
- $\lceil{log_{2}(N)}\rceil$
- $N$
---

The correct answer is  $\lceil{log_{2}(N)}\rceil$ . With $N=5$, at the end of 3 rounds, every node has received a message from every other node in the system, which means that the barrier has been reached. 

## 17. Dissemination Barrier Continued

At the end of round 2, every processor has heard from every other processor in the entire system, which means that all of the processors now know that every other processor has reached the barrier after $\lceil{log_{2}(N)}\rceil$ rounds. There is **no distinction** to be made between barrier arrival and wake-up in the case of the dissemination barrier, because **information diffusion** occurs at the end of the $\lceil{log_{2}(N)}\rceil$ rounds. This is very different from the binary wake-up process performed by the tree, MCS, and tournament barrier. 

In every zone of communication taking place between the processors, every processor is receiving exactly one message in every round of the barrier. During the information diffusion taking place throughout the dissemenation barrier, every processor receives $\lceil{log_{2}(N)}\rceil$ messages. 

Once every processor has received $\lceil{log_{2}(N)}\rceil$ messages, the processor understands that the barrier is complete and that it can move to the next round.

---

> What is a **message** in a dissemination barrier? On a shared memory machine, a message is essentially a **spin location**. And because these processors are pre-ordained, the spin locations for each processor is **statically determined**. So at every round of the tournament, the **spin location** where a processor spins on as it is waiting to receive a message can be **statically determined.** Here, this message is really a signal from its ordained peer for that particular round of the dissemination barrier. Recall that **static determination of spin location** becomes extremely important if the multiprocessor happens to be an NCC NUMA machine. Under such circumstances, the spin location can be determined by the memory address closest to a particular processor, which makes communication and spin location determination more efficient.

---

And as always in every one of these barrier algorithms you have to do sense reversal, to indicate to every processor that they must continue to the next phase of the barrier. 

 Let's talk about some of the virtues of the dissemination barrier. 
 
 The first thing that you'll notice is in the structure there is **no hierarchy**. In the tree algorithm, the root of the tree automatically produces a hierarchy in terms of the organization of the tree, but in the dissemination barrier there's no such thing. 
 
 Additionally, the dissemination barrier can work inside of NCC NUMA machines, as well as clusters. That's also a good thing. 
 
 And there is no waiting. Every processor is **independently making a decision to send a message** as soon as it arrives at the barrier. Is ready to send a message to its peer for that particular round. And of course every processor can move to the next round only after it has received a corresponding message from its peer for this particular round. So as soon as that happens it can move on to the next round of the dissemination barrier. 
 
 And **all processes** will realize that the barrier is complete when they received $\lceil{log_{2}(N)}\rceil$ messages in the entire structure of this algorithm. Because the communication in every round is fixed at $N$ messages in every round, and since there are $\lceil{log_{2}(N)}\rceil$ , rounds the communication complexity of this algorithm is on the order of $N*log_{2}(N)$. 
 
 When we compare that to the communication complexity of the tournament or the the tree barrier. In both of those cases the communication complexity was only $log_{2}(N)$ because of the hierarchy structure–communication shrinks as we progress towards the root of the tree. Therefore, the amount of communication in those algorithms is only order of $log(N$). Where as in the **dissemination barrier**, since there is no hierarchical structure, the total amount of communication in this algorithm is order of $N*log(N)$. 

## 18. Performance Evaluation

We covered a lot of ground discussing different synchronization algorithms for parallel machines both mutual exclusion lock and barriers but now it's time to talk a little about performance evaluation. As OS designers of course we're always concerned about the performance of these algorithms because all the applications that sit on top of the processor is going to be using the algorithms that you've designed. And so the performance of these algorithms are very very critical in determining how good the applications are going to be performing. 

### Spin algorithms

![[L04C_03_25.png]]

So we looked at a whole lot of spin algorithms from a very simple spin on test and set to spin with delay and spin. Algorithms that respect the order of arrival of fairness if you will. Starting from ticket lock and the queue-based locks all of these are different kinds of spin algorithms that we looked at. 

![[L04C_03_26.png]]

And we also looked at a whole number of barrier synchronization algorithms starting from a simple shared counter to a tree algorithm to an MCS tree. A tournament and dissemination. 

![[L04C_03_27.png]]

And I also introduced you to several different kinds of parallel architectures. Shared memory multiprocessor that is cache coherent. Which may be a symmetric multiprocessor or it could be a non-uniform memory access multiprocessor. And you can also have non-cache coherence shared memory multiprocessor. And of course the last thing that I mentioned to you is the mul message passing clusters. So these are the different flavors of architectures that parallel machines can be built today. 

![[L04C_03_28.png]]

And the question you want to ask is if you implement the different types of. Spin algorithms that I've been discussing with you. Which would be the winner on these machines? Well the answer is not so obvious. **It depends on the architecture.** It is extremely important for OS designers to take these different spin algorithms and implement them on these different flavors of architectures. **The winner may not always be the same on different types of machines.** The same logic applies to barrier algorithms. As a result, the better question to ask is **which would be most appropriate to implement on these different flavors of architectures**? 

Recall that, when understanding the performance evaluation that is reported in any research paper, **trends** are more important than the absolute numbers, as absolute measurements may lose significance as technology evolves overtime. Because these kinds of architectures that I mentioned to you they're still relevant to this day. And therefore what you want to ask is the question if different types of spin algorithms and barrier algorithms. When you implement it on different kinds of architectures which one of those algorithms are going to be the winners? 

That completes the discussion of synchronization algorithms for parallel machines. I encourage you to think about the characteristics of the different spin lock algorithms and the barrier synchronization algorithms that you studied in this lesson. And we also looked at two different types of architectures. **One was a symmetric multiprocessor the other was a non-uniform memory access architecture**. Given the nature of the two architectures try to form an intuition on your own on which algorithm will win in each of these styles of architectures. **Verify whether the results that are reported in the MCS paper matches your intuition.** 

Such an analysis will help you very much in doing the second project.