Note: This section refers specifically to the [[On Micro-Kernel Construction.pdf]] Liedtke paper on Micro-Kernel Construction. 

## 1. The L3 Microkernel Approach Introduction 

Both Spin and Exokernel were founded upon the assumption that **microkernel-based operating systems structure** is a **poorly performant system**. Why did they start with such an assumption? Well they used a popular microkernel of the time called Mach which was developed at CMU as an exemplar for microkernel-based operating system structure. 

But Mach had **portability** as an important goal. If we keep ***performance*** as the primary goal, what can we achieve with a micro-kernel? In this lesson we will look at L3–a micro-kernel based operating system design that provides a contrarian viewpoint to the spin and exokernal assumption.

## 2. Microkernel-Based OS Structure

![[L02D_01.png]]

Just to refresh your memory about micro kernel based operating system structure the idea is micro kernel the micro kernel is providing a small number of simple abstractions, such as **address-based and inter process communication.** All the system services that you normally expect from an operating system, such as the file system, memory manager, CPU scheduling and so on are implemented as processes **above** the microkernel.

In other words, these operating system services run at **the same privilege level** as **user-level applications**, and all of them have their own individual address spaces. **Only the microkernel runs at a different level of privilege provided by the processor architecture**. 

Since all the operating system services are implemented as server's processes on top of the microkernel, they may have to communicate with one another in order to satisfy a particular user's request. But in order to communicate with one another, they must **utilize an IPC** that is provided by the microkernel to fulfill what is needed by a particular system call from an application. 

## 3. Potentials for Performance Loss

![[L02D_02.png]]

**What are the potentials for performance loss when you have a microkernel based operating system design?** 

Recall that the main source of potential performance loss would occur at **border crossings**, as we have seen before. Border crossings have **both an explicit cost** as well as an **implicit cost** associated with it. 
- The explicit cost incurred by a border crossing occurs when an application being run at the user-level protection level of the processor requesting a service from the microkernel, which operates at a protection level beneath the application.  
	- In order to accomplish a particular service that an application has requested, the service has actually provided by several processes above the Micro Kernal and therefore there are border crossings involved going from the application to the Micro Kernal to the particular service you're talking about.

For example, consider a file system service. A system service like the file system service may have to consult other services such as a storage module, or a memory management module, in order to complete a request sent to the application. All of these situations require the execution of a **protected procedure call** between services that are part of the operating system. 

### An Aside - Protected Procedure Calls (PPC) and Implicit Costs of Border Crossing

> **Why must they be protected procedure calls?** Because these calls are going across address spaces. This action is more expensive than simple or normal procedure calls–in fact, protected procedure calls can be 100 times more expensive than standard procedure calls. This is because **each of these services, in a micro-kernel based design,** is assumed to be **implemented in its own address base** to **protect the integrity of each of these system services**.  why are protector procedure calls that much more expensive than normal procedure calls. 


> **Why are protected procedure calls more expensive than normal procedure calls?** This is where the ***implicit cost*** of border crossings is incurred, as we're losing locality both in terms of **address translations contained in the TLB** as well as the **contents of the cache** that the processor uses in order to access memory. Regardless as to whether the procedure call starts from the user address piece and goes to the kernel address piece, or between one hardware address piece representing a particular system service to the hardware address space of another system service, these repeated border crossings result in making protective procedure calls between the user space and kernel space that much more expensive.

## 4. L3 Microkernel

The key word when I describe the the performance loss in micro kernel-based operating system structure is the **potential** for performance loss. 

What L3 micro kernel does is by proof of construction they show that they can debunk the myths about micro kernel-based operating system structure. 
Now L3 micro-kernel being a micro-kernel has: a minimal set of abstractions, comprised of: 

1. address space,
2. threads,
3. an inter-process communication, and 
4. a service for providing unique IDs for subsystems that live on top of the micro-kernel.

The exact details of these mechanisms provided by L3 micro-kernel is not critical to understanding that the micro-kernel provides **the minimum set of abstractions** required of a micro-kernel system–namely address space, threads, inter-process communication, and UID. 

L3 argues that these are fundamental abstractions that **any subsystem** that lives on top of the micro-kernel or any subsystem that you want to implement in the general purpose operating system requires these facilities. And therefore L3 argues that micro-kernels should provide this as the minimal set of abstractions. How they actually provide it may differ from one micro-kernel to the other. But the important takeaway is that this is the minimal set of abstractions that a micro-kernel should provide. 

Now let's talk about the system services. As mentioned in the previous slide, the system services have to be: 

1. in distinct protection domains,
2. protected from one another, and
3. protected from the applications that live on top of the operating system. 

There is also the boundary between the applications and the servers providing the services and the micro-kernel.
![[L02D_03.png]]

Given the base structure of a micro-kernel system, what is more performant about the L3 micro-kernel? The key distinction the L3 micro-kernel makes is: while these operating system services must operate within their own protection domain, **they do not necessarily require distinct hardware address spaces**. 

L3 establishes by proof of construction that there are ways to construct a micro-kernel based operating system providing these services efficiently, given the knowledge of the hardware platform. In other words L3's argument is that **efficient implementation** of the micro-kernel impacts micro-kernel performance. Micro-kernel performance is not the consequence of a micro-kernel based operating system structure. 

However, to fully understand how L3 micro-kernel goes about systematically debunking the myths about micro-kernel based operating system structure, we have to understand the strikes against the micro-kernel.

## 5. Strikes against Microkernel 

![[L02D_04.png]]

### Border crossing

The first strike against the micro-kernel based operating system structure is the **border crossing cost** going between kernel and the user and vice versa, incurred every time a user-level process makes a system call. 

### Address space switches 

The second strike against a microkernel based design is **address space switches**. With the assumption that each system service is living in its own hardware address space, whenever an application needs any system service, that may involve the servers (which are situated above the microkernel) having to communicate in order to facilitate a particular service that was requested by the application. Recall that **protected procedure calls** are the basis for cross protection domain calls. 

![[L02D_05.png]]
So here's the protection domain–a file system. Here is another protection domain the storage module. 

If the file system has to get some service out of the storage module in order to serve the original request from the application process, that communication is implemented as a protected procedure call. Recall that **going across hardware address spaces** at minimum involves **flushing the TLB of the processor** in order to **make room for the TLB entries of the destination domain**. 

### Thread switches

The third strike against microkernel-based design is the cost for doing **thread switches**. Thread switches are mediated by the kernel. 

![[L02D_06.png]]

If the file system needs to make a protected procedure call to the storage module in order to complete an application level request for an operating system service, this can involve the microkernel mediating the filesystem in order to execute some functionality in the storage module. A thread switch with an IPC is required to perform such an operation. 

In other words, the basis for a protected procedure call is **thread switches** and **interprocess communication** which has to be **mediated through the kernel**, and kernel mediation can be expensive. That's the third strike against a microkernel. 

---

In short, all of the aforementioned strikes are explicit costs associated with providing an application level service in a microkernel based operating system, and all of them are a result of the fact that the application has to first make a request to the microkernel. 

An application may create a request to the microkernel, and the microkernel may have to pass that request on to server processes that operate above the microkernel, and the server processes may have to talk to one another through a protected procedure call, which has to be mediated by the microkernel via thread switching and interprocess communication. 

----
### Implicit costs of all of the above

![[L02D_07.png]]

In addition to all of these explicit costs there could be a fourth cost which is the implicit cost, incurred due to the memory subsystem and the loss of locality that can happen when crossing address spaces. 

When the file system makes a protected procedure call into the storage module, we are changing locality **from the address space of the file system** to **the address space of the storage module.** Therefore, the **thread** that starts executing inside the storage module may not find the cache **warm**, meaning **the contents of the cache may not be reflecting the working set of the storage module** that needs to get executed right now in order to satisfy the request coming from the file server. 

And **this loss of locality** can be incurred for both the address translation in the TLB in the **new address space** as well as for **data and instructions that are contained in the** cache. Both **translation** and **cache access** are impacted by crossing address spaces.

All of these strikes against a microkernel-based design seems pretty solid. Now how does L3 go about debunking the myths about microkernel-based design?

## 6. Debunking User Kernel Border Crossing Myth 

![[L02D_09.png]]


By proof of construction and additional calculations, L3 asserts that 107 processor cycles is the minimum number of cycles required by instructions involved in performing a border crossing on a particular processor architecture. Performance results for the L3 micro-kernel shows that border crossing is accomplished within 123 processor cycles. These processor cycles include:

- TLB misses, which are incurred due to the change in user and kernel space, 
- as well as cache misses incurred due to the execution of new code in the micro-kernel which debunks the myth that border crossing is **inherently expensive** in micro-kernel operating system design. 

123 cycles is extremely close to the 107 processor cycles calculated as hand-coded machine instructions required to cross between user and kernel spaces. By comparison, CMU's Mach operating system on the same hardware takes 900 cycles as opposed to the 123 cycles taken by L3 for border crossing. 

Spin and Exokernel used Mach as the basis for decrying microkernel-based design, stating that border-crossing in microkernel based design is prohibitively expensive. But what L3 has shown is that it need not take this much time it can be done in much shorter amount of time. 

## 7. Quiz: The L3 Microkernel Approach: Cycles

**Why did Mach take roughly 800 or more cycles than L3 microkernel for doing this border crossing between user and kernel?** Is it because:
- L3 uses faster processor, 
- Liedtke is smarter, 
- Mach's design priorities are different from L3 microkernels,
- or because microkernels are slow by definition? 

---

The right choice is **the design priorities of Mach.** 

In particular, Mach's design priority was not just extensibility of an operating system, but also **portability**. And we will talk more about how that portability design consideration comes into play. However, it is not the structure of the micro kernel that impacts border-crossings--it is only the design priorities of a given micro-kernel. 

## 8. Address Space Switches 

![[L02D_10.png]]

**What is involved in an address space switch?** Recall the components of a virtual address–that is, the index and the tag. An index is used access the TLB, and the tag that is contained in that particular entry of the TLB is matched against the tag coming from the virtual address. If they match, then we got a hit, and this particular virtual address is contained in the TLB entry, which returns a physical frame number that corresponds to the virtual page number. (It might help to revise the translation process again).

Now what happens on an address space switch–that is, a context switch going from one address space to another address space? Do we have to flush the TLB on an address-space switch? 

On a context switch the virtual address to physical address mapping will change for the new process that is going to be scheduled. In other words, if the TLB contains the translations for a particular process that is currently executing on the CPU, does the TLB need to be flushed now that the **virtual address to physical address mapping will be different for this new process** after a context switch? The answer to this is: **it depends.**

Whether the TLB needs to be flushed or not upon an address space switch depends on whether the TLB has a way of **recognizing that the virtual to physical address translation contained** in the buffer is **flagged by the distinct process** for which those translations have been put into the TLB. 

For example, if the TLB has **address space tags** in addition to the tag for disambiguating one virtual address from another virtual address, then there is no need to flush the TLB. An address space tag inside of a TLB contains **the process ID** for a particular TLB entry.

One example of an architecture that uses address space tagged TLBs is MIPS. However, on the other hand, an architecture may not use an address space tagged TLB.Intel 486 and Intel Pentium are examples of such architectures. In this situation, **all entries in the TLB must be flushed at context switch.** 

In the Intel architecture actually the TLB is split into two parts–a user part and a kernel part, where the kernel part is common regardless of which process is running. The kernel component of the TLB does not need to be flushed, but the user portion of the TLB has to be, because the virtual address to physical address mapping is going to be different for the new process that starts to run on the processor.

## 9. Address Space Switches With As Tagged TLB 

Recall that the virtual addresses are being generated on behalf of a particular process and the process has a process ID that is uniquely assigned by the operating system for that process. 

So when we make an entry into the TLB what we are storing in the TLB is not only the tag that is associated with a particular virtual address **but also the PID of the process** for this entry in the TLB. So in other words every entry in the TLB is flagged with the process ID that that particular entry corresponds to. **So how does address translation work in this address space tagged TLB?** 

Well similar to a normal TLB, we're going to take this virtual address and split it in two parts: the index and the tag. The index part is what let's us look up a particular entry in the TLB. And from the TLB we get two tags. **One tag is the address space tag.** This is signifying which process created this particular entry. So what we want to do is we want to **compare the PID of the process that is currently generating this virtual address** against **the tag that is contained at that entry**. And this matching hardware is going to say yes or no. If it says no then we are done then this entry does not correspond to the virtual address that we're trying to translate here. 

On the other hand if it ***does*** match, if this entry does correspond to this process then we want to ask the question **whether the tag that is associated with this entry is the same as the tag that is contained in the virtual address.** So that's a second level of matching that goes on. And only if both the process ID matches and the tag matches do we have a hit in the TLB. 

So in other words there is two level of matching going on. And therefore **when we do a context switch from one process to another process** there is no need to flush the TLB on the context switch because **the TLB may contain some translations on behalf of the previously executing process**. And it **may contain some translations that correspond to the new process** that has been scheduled on it. And the **hardware disambiguates these entries** by doing this **second level of matching** of **process ID against the address space tag** that is contained in the TLB. 

![[L02D_11.png]]

But if the memory management hardware does not support address space tag, then what do we do? And this is a case for instance in the Intel architecture that the TLB does not have address space tags.

## 10. Liedke's Suggestions for Avoiding TLB Flush 

Liedtke the author of the L3 microkernal suggests tricks for exploiting the hardware and avoiding TLB flushes even if the TLB is not address space tagged. 

And in particular Liedtke's suggestion is that **take advantage of whatever the architecture offers**. For example the architecture may offer **segment registers** in x86 and PowerPC both of them offer segment registers. What the segment registers do is give an opportunity for the operating system to **specify the range of virtual addresses** that can be **legally accessed** by the **currently running process**. 

![[L02D_12.png]]

What that means is even though we are given a linear virtual address space, that linear virtual address space can be carved out among several different protection domains by using segment registers. So here is the linear address space provided by the hardware. It starts from zero to a max and that max is of course **decided by the number of bits you have** for addressing in the hardware architecture. If it's a 32 bit architecture you have 2 to the 32 as the maximum addressing capability of that particular processor. If you have 64 bits you have 2 to the 64 as a maximum address space that's available in that particular architecture. So that's the linear address space that is provided by the hardware. 

If the architecture such as the PowerPC offers **segment registers** to bound the range of virtual addresses that can be legally generated by a running process, then use segment registers to define a protection domain. 

![[L02D_13.png]]

So here is one protection domain $S_{1}$ and it uses segment registers to say that this particular protection domain can generate virtual addresses starting from here to here. Any other virtual address generated by $S_1$ is illegal. The hardware will check that because the segment registers are a hardware-provided facility for bonding the range of legal virtual addresses that can be generated by this protection domain. 

![[L02D_14.png]]

Similarly another protection domain $S_2$ can use the segment registers to carve out a different portion of the linear hardware address space. So for $S_2$ the bounds for legal virtual addresses that can be generated by a thread that is running in this protect domain starts and ends at different addresses shown above. 

Therefore, we can take the hardware address space that's available, and use segment registers provided by the hardware to carve out the hardware address space into these regions. **This means that there is no need for TLB flushing on a context switch.** Why? When the TLB is accessed for a particular virtual address, the TLB segment register will act as a first line of defense and define whether a given address is within the bounds of legal addresses generated by a given protection domain. This is because the segment bounds are **hardware enforced.** 

These segment bounds are effective when protection domains are fairly small, meaning that the amount of memory space that is needed by any given protection domain is not the entire hardware address space, which allows us to carve out the available hardware address space among multiple co-resident protection domains in the hardware address space using these concept of segment registers.
## 11. Large Protection Domains 

But what if the protection domain is so large that it needs all of the hardware address space? Some examples: 
- Maybe the file system code base is so big that it needs the entire hardware address space.
- Or the code base for the storage module is so big that it may occupy the entire hardware address space.

Under such circumstances, **the TLB must be flushed on context-switch**. 

![[L02D_15.png]]

If you go from this service to the aforementioned filesystem to the storage module, a TLB flush is necessary because the **memory footprint** of each service occupies the entire hardware address space that's available on the processor. This means that the **segment registers overlap**, which further necessitates a complete TLB flush. 

Flushing the TLB at the point of a context switch is an **explicit cost,** which can be avoided given a smaller protection domain. However, if our protection domain is, as illustrated above, extremely large, then flushing the TLB will not only incur an explicit cost, but also a **significantly larger implicit cost.** 

**What constitutes an implicit cost?** Here, it is the **loss of cache locality** going from the filesystem to the storage module–that the cache is not going to have the working set of the storage module when we switch from the filesystem to the storage module. That impact is much more significant that the explicit cost. 

For example, Liedtke shows that on the Pentium architecture in which they implemented L3, there were 32 entries for kernel translations and 64 entries for user translations. It would take 864 cycles to flush out all of the entries for this TLB. 

But the **loss of locality** incurred when a service goes from one service to another in terms of cache effects is going to be much more significant, because a service that occupies a large address space is expected to be doing more work within the subsystem, and the repeated implicit cost incurred by this work will dominate the cost incurred by the singular action of flushing the TLB.  
## 12. Upshot for Address Space Switching 

Determining whether an address space switch is taking place between small protection domains or large protection domains is extremely important when maximizing efficiency. For example, switching between  small protection domains can be made more efficient by careful construction of the services. 

On the other hand, if the switch is from one large protection domain to another large protection domain, the explicit cost of switching from one hardware address space to a different hardware address space is **less significant** than the implicit cost incurred by the address space switch. Implicit costs are measured by **the loss of locality**, specifically in: 
- TLB misses that will inevitably take place once the new protection domain is executed,
- For translations, 
- And the cache effects, particularly the fact that the cache will not be warm with the data and the instructions for the new protection domain.

These implicit costs will prove to be much more significant than just the switching cost going from one large protection domain to another large protection domain. 

So this is the way the address-based switching myth is debunked by the L3 microkernel by construction.

## 13. Thread Switches and IPC 

The third myth about microkernel based design is that the **thread switches and interprocess communication can be very expensive**. 

A thread-switch takes place when the microkernel executes one thread in a particular protection domain, but needs to execute another thread in a different protection domain. To measure the explicit cost of a thread-switch, we can ask how much time does it take for this thread switch to be effected by the microkernel. **The explicit cost that is involved in thread switching is saving  the volatile state of the processor** –the registers of the CPU that have been modified by $T_1$, for example. The contents of these registers has to be stashed away in the **thread context block** before the second thread can be scheduled to run on the processor. 

L3 shows by construction that the thread switch time in L3 microkernel is as competitive as SPIN and Exokernel. Once again, L3 debunks the myth that thread switching is more expensive on a microkernel-based OS structure compared to SPIN, Exokernel, or even a monolithic operating system.

## 14. Memory Effects 

The next myth is regarding memory effects. And that myth concerns the assertion that the lost of locality in a micro-kernel base design is much more significant than in a monolithic structure, or the structure advocated by SPIN and Exokernel. 

### Memory Hierarchy 

But before we talk about the memory effects, recall the **memory hierarchy**. 

![[L02D_16.png]]

There is the CPU, the TLB, L1/L2/L3 caches, main memory, and the virtual memory that resides on the disk. The L1, L2, and L3 caches are **physically tagged.** 

**Memory effects** refer to the following: given the following hardware address space and this of course is much bigger than the amount of space that's available in these caches. And in fact we know that the entire hardware address space may not even be in physical memory. Because In a demand-paged system when a process is executing the pages that it needs will be demand-paged from the virtual memory that's on the disk and brought into physical memory. When the processor is executing instructions then the instructions and data contained in physical memory move into the memory hierarchy close to the CPU so that the CPU can access the **working set** of the **currently running thread** by getting it from the cache that is closest to it. That's the hope in this memory hierarchy. 

What we mean by memory effects is answered by the following question: When we context switch between protection domains, **how warm are the caches**? 

![[L02D_17.png]]

Say that we have the following smaller protection domains: $P_1$, $P_2$, $P_3$, and $P_4$. Liedtke's suggestion for smaller protection domains is **NOT** to put each of these in its own hardware address space, but rather **pack them together in the same hardware address space** and then **force protection for these processes from one another through segment registers.** 

Therefore, when working with smaller protection domains, even between context switches from process to process, **there's a good chance that the cache hierarchy is going to contain the working set of the newly scheduled small protection domain**. In short, memory effects can be **mitigated** by **carefully structuring the protection domains** in the hardware address space. Therefore, **debunking the myth with respect to address space switching** also helps in **reducing the ill effects of implicit costs associated with address space switching** because of these small protection domains, since in occupying a small memory footprint, they occupy a small memory footprint in the caches as well. Across small protection domains, there's a good chance that the **locality for the newly scheduled small protection domain** is going to be **contained in the cache hierarchy.** 

We already mentioned that if the protection domains are large you cannot avoid **cache pollution**, whether it is a monolithic kernel, the exokernel or the SPIN. If the memory footprint of the system service is very big, then its contents will **pollute the cache** when we visit that particular system service. Even with a monolithic kernel that has subsystems occupying a significant portion of the hardware address space, the implicit costs in the memory hierarchy will still be felt even when no context switches are being performed, **because the cache is physically tagged**. And therefore when context switch from large protection domains or large subsystems in the context of a monolithic kernel, **cache pollution is unavoidable**. So the only place where a monolithic kernel can win or an Exokernel can win or a SPIN can win is in small protection domains. A microkernel can also win for a small protection domain by packing multiple small protection domains in the same hardware address space.

Why were the memory effects so bad in Mach? Recall that border crossing on Mach incurred over 800 cycles, as opposed to the 120 or so cycles incurred by border crossing on an L3 micro-kernel. **What made the Mach micro-kernel perform so poorly?**

## 15. Reasons for Mach's Expensive Border Crossing 

The Mach microkernel is **architecture independent** to allow that Mach microkernel to run on several different processor architectures. Therefore, the **Mach microkernel is a victim of extensive code bloat.**

In particular in the Mach microkernel there is both an architecture independent component and an architecture-specific component of the microkernel. Together, these components produce significant code bloat, which also means that the Mach microkernel has a **large memory footprint**. 

As illustrated before, **larger memory footprints** lead to **loss of locality**, and a loss of locality implies **higher numbers of cache misses** and **cache pollution**.  This is the reason for a longer latency incurred by border crossing in Mach as opposed to the theoretically smallest number of processor cycles (107) required to perform a border crossing between user to kernel domains. 

Recall that Liedtke implemented the L3 microkernel and showed that it took only 123 processor cycles to do a border crossing, which he calculated would require at minimum 107 processor cycles. In other words, **Mach kernel's memory footprint** is the reason for expensive border crossing. By proof of construction L3 has shown that you can have very minimal border crossing overhead even when operating under the principles defined by micro-kernel construction. Alternatively, **Mach sacrificed performance in exchange for portability.***


## 16. Thesis of L3 for OS Structuring 

So L3 Microkernel shares to debunk the myth about Microkernel-based operating system structure. It goes beyond that and it has a thesis for how operating systems should be structured. 

### Principle 1 - Minimal abstractions in the micro kernel 

The first principle advocated by L3 is that the Microkernel should have minimal abstractions that includes support for address spaces threads interprocess communication and generating unique IDs. **Why do we need these abstractions in the microkernel?** 

The argument is that these four abstractions that I mentioned–address space, threads, IPC, and UID–are abstractions that are needed by any subsystem that provides a functionality for end users in an operating system. Therefore the principle of optimizing the common case suggests that **these abstractions should be part of any microkernel**. 

### Principle 2 - Microkernels are process-specific in implementation

The second thesis coming out of L3 microkernel experience is that microkernels are process specific in implementation. In other words if you want an efficient implementation of microkernel you have to take advantage of whatever the hardware is offering you. Which suggests that **micro-kernels by the very nature are non-portable if high performance is the primary goal.** 

### Overall Thesis

What L3 is advocating for, in terms of these two principles, is packaging the **minimum viable set of kernel abstractions** and **processor-specific implementation**. With these two principles, efficient processor-independent abstractions can be built at the upper layers. All of the services that we associate in a monolithic kernel like a UNIX operating system such as file system network protocols scheduling memory management, can be implemented in a **processor independent way** on top of a **microkernel that provides the right set of abstractions** and **exploits whatever the hardware gives** in terms of capabilities to achieve an efficient, processor specific implementation. 

That's the thesis of L3 microkernel: implement processor specific kernel and processor -independent abstractions at higher layers of the operating system stack.

## 17. The L3 Microkernel Approach Conclusion 

Research on the structure of operating systems in the mid' 90s as exemplified by the three research papers we studied in this course module led to fascinating innovations in the operating system structure. 

It is important to note that all three systems–SPIN, Exokernel, and the L3 microkernel were done contemporaneously. In this sense they **mutually informed** each other and laid the **basis for several innovations** in operating system structuring. 

For example, many modern operating systems have internally adopted a microkernel based design. Similarly, technologies such as **dynamic loading of device drivers into the kernel** have come out of the research into the extensibility of operating system services. The next lesson we're going to look at is a logical progression of the idea of extensibility namely virtualization.