Note: [[Engler_Exokernel An Operating System Architecture.pdf]] This section refers directly to the Engler paper on Exokernel.

## 1. Exokernel Approach to Extensibility 

Having seen SPIN's approach to extensibility, we will now look at Exokernel's approach to operating system extensibility. 

The name Exokernel itself comes from the fact that the kernel exposes hardware explicitly to the operating system extensions living above it. The basic idea in Exokernel is to **decouple authorization of the hardware from its actual use.** 

![[L02C_01.png]]

### Authorization of the hardware 

Let's say a student wants to do research in a lab. The professor may interview the student before giving the student a key to the lab, which provides the student resources to perform the research they need. Exokernel functions in a similar way. 

In Exokernel, the library operating system will ask for a resource. Exokernel will validate the request for the resource from the library, and bind the request to the specific hardware resource. In other words, Exokernel **exposes the hardware that was requested by the Library** OS through **creating a secure binding between the ask and the actual hardware resource**. 

Once Exokernel has established this binding **it creates an encrypted key for the resource** and gives it to the requesting library operating system. 

Similar to the analogy that I gave you of a student using my lab resource, **the semantics of how the resource is going to be used by the library is entirely up to the library.** It is not the responsibility of Exokernel to define exactly  **the semantics of how a particular hardware resource is used**, so long as the library operating system is staying within the usage restrictions defined by Exokernel. 

Once a library operating system has asked for a resource and Exokernel has created the binding for that resource to the requesting library operating system, then the operating system is now ready to use the resource. 

### How the resource is used

Basically what the library operating system will do is **present the encrypted key** that it has received authenticating that use of the resource for this library to the Exokernel. In other words Exokernel will be able to **validate** whether **the key presented to it is the key that was presented for this particular libraries operating system**. So in other words the key cannot be forged, and the key cannot be passed around. 

If I gave a key to this library operating system that key if it is presented to the Exokernel by this library operating system it's a valid key. If it is a valid key, but it is not the operating system to which Exokernel gave the key, then that request would be denied. 

So with a valid key any time the library operating system can present the key to the Exokernel, Exokernel will validate it, and then the library operating system is free to use resource for which it has this valid key. This is similar to a doorman in an apartment building checking when a resident comes in whether the resident is a bona fide occupant of the residence. 

Once inside his apartment what the resident does is not something that the doorman cares about. Exactly the same thing is being done by Exokernel as a doorman for using the hardware resource for which a valid key exists with a library operating system. **Establishing the secure binding is a heavy duty operation**. That's where Exokernel comes in the middle of saying: Can I give access to a specific resource being requested by a particular library operating system? 

Once such a secure binding has been established the actual use of the hardware is going to be much cheaper.
## 2. Examples of Candidate Resources 

If Exokernel has to validate the key every time for the library to use it, there is valid concern that the repeated validation process would impact performance. However, this would depend on what is defined as a resource. Let's look at some examples of a candidate resource. 

### A TLB Entry

TLB entry is going to establish a mapping between a virtual page to a physical page. The process of mapping the virtual page to the physical page is done by the library. 

Now once the mapping has been done by the library, it presents the mapping to the Exokernel, along with the encrypted key that it has for a particular TLB entry. 

Exokernel validates the key, and puts this mapping into the specific TLB entry of the hardware TLB, as inserting an entry into the hardware TLB is a privileged operation. The library operating system cannot perform this operation alone because **it doesn't have the same privilege as Exokernel.** Once the encrypted key for this TLB entry is presented to Exokernel, then Exokernel, on behalf of the operating system, **is putting that mapping that has been established by the library operating system into the specific TLB entry of the hardware TLB**. 

Once this entry has been put into the TLB, the process that is going to be using that virtual page when it is running can use this multiple times **without Exokernel intervention**. Even though inputting an entry into the hardware TLB required the intervention of Exokernel, once that entry has been put in, processes of that library operating system when they are running on the CPU can **access the TLB and perform the translation any number of times** because these accesses take place under hardware control, thus Exokernel is not involved in these specific accesses. 

So even though Exokernel intervention is required in order to perform certain operations against the hardware, the actual use of a hardware resource is not affected by the fact that Exokernel is in the middle between the hardware and the library operating systems. 

### Packet Filtering

Let's say that the operating system wants to install a packet filter that needs to be executed every time a network packet arrives on behalf of a library operating system. Predicates for looking at this incoming packet are loaded into the kernel by the library operating system. 

Now this is a heavy-duty operation because you're doing it with the help of Exokernel. But once those predicates have been loaded into Exokernel by the library operating system on every packet arrival, Exokernel will automatically check it using those predicates. 

So those are examples of candidate resources that tell you that **establishing the binding may be expensive** but **using the binding once established does not incur the intervention** by exokernal Exokernel, thus binding usage can happen at hardware speeds.

## 3. Implementing Secure Bindings 

Now let's talk about the mechanisms that are there in Exokernel for implementing these secure bindings. There are three methods. 

### Hardware Mechanisms

Recall the example of the TLB entry. Other examples of hardware mechanisms include:
- retrieving a physical page frame from exokernel, 
- or retrieving portion of the frame buffer that is being used by the display. 

These are all examples of specific hardware resources that can be requested by the library operating system and can be bound to that library operating system by Exokernel and exported to the library operating system as an encrypted key. Once the library operating system has the encrypted key for that resource, the library operating system can use that resource any time it wants. 

### Software Caching

The second mechanism that Exokernel has is **software caching on behalf of each library operating system**, specifically the shadow TLB. 

Caching the hardware TLB in a software cache for each library operating system is meant to **avoid the context switch penalty** when Exokernel switches from one library operating system to another. 

Basically what will happen is that at the point of context switch, Exokernel will dump the hardware TLB into a software TLB data structure that is associated with that specific library operating system, and similarly load the software TLB of the library operating system to which it is switching to into the hardware TLB. 

We will talk about these mechanism in much more detail shortly but at this point I wanted to mention that this is second mechanism that exists in Exokernel for establishing a secure binding between a library operating system and the hardware. 

### Downloading code into the kernel 

The third mechanism that exokernel has for establishing a secure binding on behalf of an operating system is downloading code into the kernel. This is simply to **avoid border crossing** by inserting specific code that an operating system once executed on behalf of it. Recall the example of the packet filter.  This is an example of downloading according to the kernel that needs to be **executed on behalf of a particular guest operating system**. This is very similar to the SPIN idea of extending the kernel with logical protection domains that I created and dynamically linked in. 

Similarly in Exokernel, a library operating system can securely download code into the kernel that will get executed under specific conditions that are laid down by the library operating system.

## Quiz: Exokernel vs Spin 

In Exokernel, there is the mechanism of downloading code into the kernel. SPIN has a similar functionality which is to extend logical protection domains. 

The question to you is **which one of these two mechanisms compromises protection more?**

---

The correct answer is Exokernel. 

Now so long as SPIN's logical protection remains follow Modula-3 language-enforced compile time checking and run time verification, then there is no violation of protection in SPIN. 

But we cannot say the same about Exokernel because it is **arbitrary code** that is being **downloaded into the kernel** by a **library operating system** and, which may end up compromising protection more than the SPIN mechanism. 

Having said this, it is not always possible to live within Modula-3 enforced protection domains even in SPIN. In order to do certain things in the hardware SPIN may have to step outside the protection boundaries of Modula-3. It's not always possible to do this within the confines of language-enforced protection domains. However, if you just think in terms of the logical protection domains as defined by SPIN as modula 3 objects. Those have strong guarantees of protection compared to arbitrary code that we can download into Exokernel.

## 5. Default Core Services in Exokernel 

Recall that memory management and CPU management are core services that any operating system has to provide. SPIN had its own approach to dealing with those core services. We will do the same analysis for Exokernel as to how it does memory management and CPU scheduling first memory management.

### Memory Management in Exokernel 

Let's see how Exokernel will handle a page fault incurred by a library operating system. 

![[L02C_02.png]]

So in this picture that I'm showing you here is an application thread running and maybe this application thread belongs to a specific library operating system. So long as this application thread is doing normal memory accesses where all its virtual addresses have been mapped to physical page frames, then the thread is executing at hardware speeds on the CPU. Life is good. 

![[L02C_03.png]]

But life may not be always good. Because this thread my incur a page fault. And when it incurs a page fault the page fault is first fielded by Exokernel. 

![[L02C_04.png]]

Exokernel knows which library operating system is currently executing on the CPU, but it has no knowledge of processes within a library operating system. All it knows is that this library operating system is doing something on the CPU... and it knows that there is a page fault incurred and it can kick it up to the library operating system through a **registered handler** (see below). 

Because the library operating system knows about processes whereas Exokernel has no knowledge about that and it services the page fault. And servicing the page fault may involve requesting Exokernel for a page frame to host the specific page that is missing. 

And if it does that as we detailed before that will involve the library asking Exokernel for a page frame and Exokernel creating a binding for a page frame, and then returning an encrypted key for the page frame. 

Assume for the moment that the library operating system has page frames. In servicing the page fault it establishes a mapping between the virtual piece that was missing and the page frame that contains the contents of the virtual page. Once the library OS does that, the mapping between the virtual page and the frame that it corresponds to has to be presented to the Exokernel. 

![[L02C_06.png]]

So the library presents that mapping to the Exokernel along with the TLB entry where it wants this mapping to be placed in the hardware TLB. 

Remember that when the process runs, the CPU is consulting the hardware TLB to check for a valid mapping between the virtual page number and a physical frame. Recall that the page fault was incurred because the mapping did not exist and the whole point of this exercise is for the library operating system to reestablish a mapping. But the library OS cannot do that directly into the TLB--so it **presents** a mapping to Exokernel along with **the encrypted key** that represents the capability that the library operating system has to a specific TLB entry where it wants this to be placed. 

Exokernel will validate the encrypted key presented by the library operating system. If valid, Exokernel will then **install the mapping in the hardware TLB**.

Recall once more that this is a a privileged operation, which means that it can only be performed in the kernel-mode of the processor. This is the reason for the **red line** between the library operating system that runs at the non-privileged level and Exokernel that runs at the privileged level to do certain operations such as installing an entry into the TLB. 

After the entry had be installed in the TLB, if the same process is run by the library operating system and it generates the same virtual address, a valid mapping will then be discovered. 

## 6. Secure Binding 

During secure binding, the Library OS has the ability to **drop code into the kernel** to avoid performing border crossings. Obviously, however, this can become a serious security loophole. Even in SPIN, we've noticed that a core service may have to step outside the language enforce protection mechanism in order to control hardware resources. 

In short: while both SPIN and Exokernel initially allow extensibility, **they may necessarily restrict who will be allowed to do such extensions**. Not any arbitrary user. It has to be **a trusted set of users.**

## 7. Memory Management using S TLB 

Software caching is a mechanism that's available in Exokernel for establishing secure binding. Software TLB is one specific example of using the software caching. 

During a context switch, one of the biggest sources of performance loss is the fact that **locality is lost** for the newly scheduled process. Since the address base occupied by each library operating system are **completely different** when we switch from one library operating system to another we have to flush out the entire TLB. 

When we run this other Library OS2, it is not going to find any of its virtual addresses in the TLB. In order to mitigate that overhead Exokernel has this mechanism called software TLB. 

The idea is quite simple the software TLB is sort of a snapshot of the hardware TLB for each of the operating systems. 

![[L02C_08.png]]

So this software TLB if a data structure in the exokernel that represents the mappings for Library OS1. Similarly this data structure represents the mapping for Library OS2. 

![[L02C_07.png]]

Assume that Library OS1 is running, and the TLB entries correspond to valid mappings for Library OS1. Assume that Exokernel decides to switch from Library OS1 to Library OS2. 

At that point, Exokernel will **dump the TLB into the software S-TLB OS1.** Not all, but ***some subset*** of the TLB mappings will be dumped into S-TLB OS1.

Let''s say that we are switching from Library OS1 to Library OS2. In that case, Exokernel will pre-load the TLB with the S-TLB data set that is associated with Library OS2. 

As a result, when the library operating system starts running on the CPU, it will find some of its mappings already present in the hardware TLB. That's the idea in Exokernel of having the S-TLB data structure associated with every library operating system to mitigate the loss of locality that happens when you do context switch in terms of address translations. 

Of course when the library operating system starts running and it does not find a mapping for a virtual address in the TLB, Exokernel is going to handle the missing virtual page translation in the TLB as a page fault and pass it up into the library operating system, which will be handled as in the earlier slide. 

## 8. Default Core Services in Exokernel (cont) 

### CPU Scheduling 

In order to facilitate this core service, Exokernel maintains a linear vector of time slots. 

![[L02C_09.png]]

So time is divided into these epochs $T_1$, $T_2$, $T_3$ and so on. Every time quantum has a beginning and an end. And these time quantums represent **the time that is allocated** to the **library operating systems that live on top of Exokernel**. 

The time quantum is bound by the **begin** and **end** markers for each library operating system. Each library operating system gets to mark its time quantum at startup in this linear vector of time slots. 

![[L02C_11.png]]

So for instance OS1 may say that I get this time slot I get this time slot maybe some of the time slot and so on. And similarly OS2 marks its spots in the linear vector time slots. 

So CPU scheduling in Exokernel is essentially looking at this linear vector of time slots and asking the question: In this time quantum, which library operating system should be running on the processor?

Let's say OS1 is now running on the CPU. When the time quantum reaches its end, control is transferred by Exokernel to the library operating system to **save and restore context as needed**. 

The time that is allowed for a library operating system to do the saving and restoring of context at the point of a context switch is bounded. If the operating system takes more time than allotted to perform a context switch, exokernel will remember the OS that his exceeded the time-quantum, deduct time from the associated OS's time quantum when it is next scheduled.  There is a penalty associated with exceeding the time quantum. 

The time quantum is bound–during this time quantum, OS1 has complete control of the processor and Exokernel is not going intervene during the execution of OS1 unless if a process on OS1 incurs a page fault. In that case, Exokernel will have to intercept the page fault and pass it up to the operating system. 

Otherwise, during this time quantum the operating system is entirely at liberty to use a processor for running whatever processes it wants to. 

At the end of the time quantum the time-interrupt goes off. Exokernel feels it and kicks it up to the operating system and tells the operating system to clean up and save any context it wants so that the CPU can be reallocated to the next library operating system. And that's where the time is bounded as to how much time the library operating system can take in order to do that saving of the context.

## 9. Revocation of Resources 

Notice so far that Exokernel supports no extractions. It **only has mechanisms for securely giving resources to the Library Operating System**, where the resources may be a space resource memory time resource or specific hardware resource like area of the graphic display and so on. 

Therefore Exokernel needs a way of **revoking** or **reclaiming** resources that have been allocated to a library operating system. Of course exokernel keeps track of what resources have been allocated to different library operating systems. At any point of time it can revoke the resources that had been given to a library operating system. 

We can revisit the last example of a student being given a key to the lab. When they graduate and return the key to the lab, that is an example of **revoking lab access** from the requestor.  


Similarly Exokernel has mechanisms for revoking resources from a library operating system. So specifically for instance if you think about the CPU Exokernel has no idea what the library operating system is using the CPU for. Compare with SPIN, for instance, which has an abstraction of `strand` that represents the library's notion of a thread. Exokernel has a `revoke` mechanism for exactly this purpose of revoking resources that have been allocated to a library operating system, and the revoke call–which is an upwards call into the Library OS–will give the library operating system a **repossession vector** that lists the resources being revoked by Exokernel. For instance, if Exokernel gave page frame access to the Library OS before, it can **reclaim** these page frames for later use. 

![[L02C_12.png]]

When the **repossession vector** is given to the Library OS, it is now the responsibility of the Library OS to **clean up the resources** listed in the **repossession vector**. 

![[L02C_13.png]]

For example, if Exokernel wants to reclaim frame number 20 and page frame number 25 from the library operating system, then the Library OS will store the contents of those page frames onto the disk. Storing these contents is the **corrective action** that the library operating system will have to take when it is informed of the resources pending reclamation. 

To reduce the number of responsibilities of Library OS, the Exokernel also allows a library to **seed auto-save options** for resources pending revocation. For example, if Exokernel decides to revoke page frames from the Library OS, the Library OS could have seeded the Exokernel ahead of time with the corrective action of **dumping page frame contents to disk** upon revocation of the page frame resource. 

Seeding allows the Exokernel to work on behalf of the Library OS. As a result, the amount of work required of the Library OS to perform a corrective action during revocation is minimal. 

## 10. Quiz: Code Usage by Exokernel 

Give a couple of examples of how the **downloading code mechanism** provided by Exokernel may be used by the Library OS. 

Recall that this mechanism works in the following way: the Library OS may have a piece of code it wants to run, and present this code snippet to Exokernel to download. Give some examples of how a Library OS may use this core downloading facility provided by Exokernel.

---

Here are a couple of examples:

1. Packet filter for **de-mux** of network packets 
2. Run code in Exokernel on behalf of Library OS not currently scheduled (e.g., **garbage collection** for an app)

There are many other examples that may exist, but **packet filtering** is an extremely critical performance component of any operating system. Therefore, Library OS might install a packet filter for demultiplexing incoming network packets so that Exokernel can hand packets intended for a particular library operating system by running this code on behalf of the library operating system. 

A second example would be things that a library would like Exokernel to do on its behalf even when it is not currently scheduled. For instance, the Library OS may request the execution of a garbage collection mechanism for an application. And that's something that can be installed as a code that is downloaded into Exokernel and executed on behalf of that library operating system. 

## 11. Putting it All Together 

How can Library OSes function within the conditions defined by the protection boundary for Exokernel, yet continue to execute the applications belonging to the OS directly on hardware resources without obstructing the responsibilities of Exokernel (and vice-versa)? That is, how does Exokernel achieve extensibility, protection, and performance? 

![[L02C_14.png]]

So one of the hooks for getting good performance is what I mentioned earlier and that is called that a performance critical for library operating system is something that can be downloaded securely into the exokernel so that piece of code becomes sort of the extension of exokernel. For a particular library operating system. This may be for OS one this maybe for OS two and so on and so forth. 

So now with this setup at any point of time some application process of some library operating system is running on the CPU. 

Remember that Exokernel has no knowledge of the **processes within any of these library operating systems**. All it knows is existence of these library operating systems and the fact that Exokernel has been the broker in giving some hardware resources, or capabilities for some hardware resources I should say, to specific library operating systems, and it has also been the broker for downloading some code specific to library operating systems into the exokernel code base itself. 

![[L02C_15.png]]

Let's say that this particular thread belongs to this library operating system as long as this thread is well behaved by which I mean it's not doing anything funky. Doing normal program execution. Accessing memory. For which mapping exists in TLB and so on. Life is good. Address translation happens on every memory access entirely in the CPU and the process is making forward progress at memory speeds without intervention from exokernel or any of the library operating system. 

But there could be this discontinuities in the execution of this process. For example let's say that this processor thread makes a system call like opening a file. Or worse yet it may incur a page fault meaning that not all of the pages for this particular process is currently in the mapping available to the hardware in the TLB and therefore there's a page fault. Even worse this thread could do something stupid such as divide by 0 or something like that that causes an exception. And lastly the thread is not doing anything to cause the discontinuity but there is an external interrupt that came in and that is going to cause a discontinuity to this execution of this process on the CPU. All of these are examples of discontinuities in the normal execution of this process. All such discontinuities essentially **result in the CPU incurring a fault or a trap** and the trap is fielded by Exokernel. 

When such discontinuities occur exokernel has to pass the program discontinuity to the appropriate Library OS that is living on top of it. Now Exokernel knows, based on the linear vector of time slots mentioned earlier, which library operating system is currently running on the CPU. the correct library operating system to which it has to pass the discontinuity that occurred right now for the currently executing process. 

![[L02C_16.png]]

To facilitate a finer-grain association between these different kinds of discontinuities and the specific functionality in the Library OS for dealing with those discontinuities, Exokernel maintains state for each currently existing library operating system. 

We will discuss the state maintained by Exokernel on behalf of every Library OS next.

## 12. Exokernel Datastructures 

![[L02C_17.png]]

### $PE_{OS}$ Data Structure and Handler Entry Points

To facilitate the bookkeeping needed for the different types of program discontinuities that I mentioned Exokernel maintains a **PE data structure** ($PE_{OS1}$) that corresponds to each of the library operating systems. The PE data structure contains **the entry points** in the Library OS for dealing with the different kinds of program discontinuities. 

For example, exceptions that are thrown by the currently executing process has a **specific entry point** in the library operating system. Following this logic, external interrupts are going to be handled by **interrupt handlers** which have **specific entry points** into the Library OS. If a process on the Library OS performs a system call, it does so within a **protected entry context**, which is the entry point in a library operating system for system calls that are made by the currently running process that belongs to that specific Library OS. 

This last entry may correspond to **the addressing context** of the handler for the page fault service in the library operating system. In short, this PE data structure–which is unique for every library operating system–contains the **handler entry points** for different types of events that are fielded by Exokernel. 

> **Note:** The PE data structure, which corresponds to the Exokernel action of calling the appropriate handler entry point when a particular event is triggered, is very similar to the event handler mechanism that we discussed in the spin operating system. 

### Software TLB and Handler Entry Points

Recall the $STLB_{OS}$ that Exokernel also maintains on behalf of every operating system. The idea that $STLB_{OS1}$ corresponds to Library OS1 and $STLB_OS2$ corresponds to Library OS2 represent what is called a **guaranteed mapping**. Recall that on a context switch, Exokernel will **save part of the current hardware TLB state** into the $STLB_{OS}$ that is associated with the particular operating system. **What role does the handler entry point play in this process?** 

Here, the handler entry point is specifying to Exokernel the **set of guaranteed mappings** that a particular Library OS wants Exokernel to maintain on its behalf every time the Library OS is scheduled. And that's the set of TLB entries that are dumped into the software TLB data structure at the point of a context switch. Not all of the hardware TLB entries but **only the entries that have been guaranteed to be kept** on behalf of a particular operating system are dumped into a software TLB data structure and repopulated into the hardware TLB when the same OS is scheduled on the hardware. 

Earlier, it is mentioned that an external interrupt is another source of discontinuity for the currently executing process. Note that an external interrupt ***may not always*** be for the currently scheduled operating system. It may be for some other operating system– for example, maybe some Library OS2 scheduled a disk I/O, and the interrupt that came into Exokernel is on behalf of Library OS2 and not the other Library OS that is currently executing on the CPU for this operating system. 

Exokernel **must** be able to associate an incoming interrupt with the corresponding operating system anticipating the interrupt. **Downloading code into the kernel** is one mechanism that Exokernel provides allows first-level interrupt handling on behalf of a library operating system.

## 13. Performance Results of SPIN and Exokernel 

How can we evaluate the results of SPIN and Exokernel? In other words, how do we prove that we are achieving extensibility and performance? 

> Quote from lecture: "How do you make sense out of the quantitative results from a data research paper? When you research papers remember that **absolute numbers are meaningless**. It is the **trends that are meaningful**."

 While doing any performance study of a new system that you're proposing you have to identify what the competition is. Remember that spin and exokernel were done in the early to mid 90s. For SPIN and Exokernel, the competition is a **monolithic operating system** and a **microkernel based operating system**. And the competition at that time for both SPIN and Exokernel was UNIX as a monolithic example and Mach from CMU as a micro kernel example. 
 
 Performance questions always center around space and time. For example: how much better, timewise, is an extended kernel (either SPIN or Exokernel) compared to a microkernel-based approach in terms of performance? 
 
  Because we know that an extended kernel may have to incur **loss of locality** and **border crossing overhead** and so on compared to a monolithic kernel, we may also want to ask: **Is the extended kernel at least as good as a monolithic kernel?** If we ask another question: What is the code size of implementing a standard operating system as a monolithic operating system (IE: Unix) versus a micro-kernel based operating system, or an extended kernel operating system? This would be considered a **space** question. And so we can have time questions as well as space questions when we propose a new way of doing any particular system component. 
  
The key takeaway from the SPIN and Exokernel performance papers is that the performance results reported by both SPIN and Exokernel is **MUCH** better than the Mach microkernel. 

However, for a protected procedure call–that is when you go from one protection domain to another–what is the performance results of the extended kernel? Once again, note that both SPIN and Exokernel exceed the Mach microkernel, and that both SPIN and Exokernel deal with system calls as performantly as a monolithic kernel does.