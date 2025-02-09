## 1. CPU and Device Virtualization Introduction 

As we said at the outset of this course module virtualizing the memory subsystem in a safe and performance conscious manner is the key to the success of virtualization. What remains to be vertualized are the CPU and the devices. That's what we'll talk about next. Memory virtualization is sort of under the covers. When it comes to the CPU and the devices the interference among the virtual machines living on top of the hypervisor becomes much more explicit. The challenge in virtualizing the CPU and the devices is giving the illusion to the guest operating systems living on top that they own the resources. Protected and handed to them by the hypervisor on a need basis. 

There are two parts to CPU virtualization we want to give the illusion to each guest operating system that it owns the CPU that is; it does not even know the existence of other guests on top the same CPU. If you think about it this is not very far removed from the concerns of a time shared operating system which has to give the illusion to each running process that that process is the only one running on the processor. 

The main difference in the virtualized setting is that **this illusion is being provided by the hypervisor at the granularity of an entire operating system**. That's the first part. The second part is **we want the hyperviser to field events arising due to the execution of a process that belongs to a parent guest operating system in particular during the execution of a process on the processor**. There are going to be discontinuities that occur. And those program discontinuities have to be fielded by the hypervisor and passed to right guest operating system.

## 2. CPU Virtualization 

![[Pasted image 20250119174416.png]]

To keep things simple let's assume that there's a single CPU. Each guest operating system is already multiplexing the processes that it is currently hosting on the CPU in a non-virtualized setting also. So each operating system has a ready queue of processes that can be scheduled on the CPU but there is this hypervisor that is sitting in between a guest operating system. It's already queued and the CPU. 

![[Pasted image 20250119174905.png]]

So the first part of what the hypervisor has to do is to give an illusion of ownership of the CPU for each of the virtual machines, so that each virtual machine can schedule the processes that it currently has in it's very queue on the CPU. If you look at it from the hypervisor's point of view it has to have a precise way of accounting the time that a particular VM uses on the CPU from the point of view of billing the different customers. That is the key thing that hypervisor is worried about that it gave the CPU for a certain amount of time to this VM certain amount of time to this VM and so on. Now for the time that has been allocated for instance this virtual machine. How this virtual machine is using the CPU time , what processes it is scheduling on the CPU, the hypervisor doesn't care about that. So commensurate with whatever is a scheduling policy enforced in a particular guest operating system, that guest operating system is free to schedule its pool of processes on the CPU for the time that has been given to it by the hypervisor. 
Now similar to the policy that we discussed for memory allocation, one straightforward way to share the CPU among the guest operating systems is to give a share of the CPU–a proportional share of the CPU for each guest operating system commensurate with the service agreement that this virtual machine has with the hypervisor. This is called a **proportional share scheduler** and it is used in the VM-ware ESX server. 

Another approach is to use a **fair share scheduler**, which is going to give a fair share of the CPU for each of the guest operating systems–that is, an equal share to each of the guest operating systems, running on top of the hypervisor. Both of these strategies proportion share scheduler or a fair share scheduler are straight-forward conceptual mechanisms, and you can learn more about them from the assigned readings for this course. **In either of these cases the hypervisor has to account for the time used on the CPU on behalf of a different guest during the allocated time for a particular guest.** And this can happen for instance if there is an external interrupt that is feeded by the CPU that is intended for this VM while a process that belongs to this VM is going to be executing on the CPU. So this is accounting that the hypervisor would keep to make sure that. Any time that was stolen away from a particular VM in order to service an external interrupt that did not belong to this VM it'll get rewarded for later on by the accounting procedure in the Hypervisor. We have discussed this issue already in the concept of operating system extensibility specifically when we discussed the Exokernel approach to extensibility. So it's not new problem. It is just that this problem occurs in the hypervisor setting also.

## 3. Second Part (Common to Full and Para) 


The second part of CPU virtualization which is common to both full and para-virtualized environments is being able to deliver events to the parent guest operating system. And these events are events that are generated by a process that belongs to the guest operating system currently executing on the CPU. Let's see what is happening to this process when it is executing on the CPU. 
![[Pasted image 20250119175003.png]]
Once this process has been scheduled on the CPU during its normal program execution everything should be happening on hardware speech what is that mean? Well. The process is going to generate virtual addresses that has to be translated to machine page addresses. And we talked at length about how the hypervisor can ensure that the virtual address translation to the machine page address can be done at hardware speeds by clever tricks in the hypervisor and or the guest operating system. In a fully virtualized environment the hypervisor is responsible for ensuring that the virtual address gets translated directly to the machine address without the intervention of the guest operating system. And para-virtualized environment once again with cooperation between the guest and the hypervisor we can ensure that the page table that is used for translating virtual address to physical addresses is something that had been installed on behalf of the guest operating system by the hypervisor so that the translation can happen at hardware speeds. 

This isthe most crucial part of ensuring good performance for the currently executing process in a virtualized environment–how address translations are dealt with. That's why I kept harping on the fact that virtual memory managing the virtual memory is the thorny issue in a virtualized setting. So we know that this is in good hands we know how to handle that. Let's look at other things that can happen to this process during the course of its execution. 

![[Pasted image 20250119175101.png]]

One thing that this process may do. Is execute a system call. For instance it might want to open a file. And that's really a call into the guest operating system. That's something that this process may do. 

![[Pasted image 20250119175118.png]]

Another thing that can happen to this process is that it can incur a page fault. Not all of the virtual address piece of the process is going to be in the machine memory. And therefore if some virtual page cannot be translated then it's going to result in a page fault. 

![[Pasted image 20250119175137.png]]

Or the process may throw an exception. For instance it might do something silly like divide by zero which can result in an exception.

![[Pasted image 20250119175154.png]]

And lastly though no fault of this process there could be an external interrupt when this process is executing. So these are the kind of discontinuities these are called program discontinuities that is affecting the normal execution of this process. And the first three things that I mentioned syscall page fault and exceptions are due to this process in some shape or form. **But the fourth one the external interrupt is something that is happening asynchronously and unbeknownst to what this process intended to do while running on the processor.** 

And all of these discontinuities have to be passed up by the hypervisor to the guest operating system. **So the common issue to both full and para virtualized environment is that all such program discontinuities for the currently running process have to be passed up to the parent guest operating system by the hypervisor.** Whether it is a para virtualized or a fully virtualized operating system, nothing special needs to be done in the guest operating system for fielding these events from the hypervisor, because all of these events are going to be packaged as software interrupts by the hypervisor and delivered to the guest operating system. Any operating system knows how to handle interrupts. 

So all the hypervisor has to do is to package these events as software interrupts and pass it up to the guest operating system. There are some quirks of the architecture that may get in the way and the hypervisor may have to deal with that. Recall that syscalls and page faults and exceptions are all things that need to be handled by the guest operating system. So it probably has entry points in it for dealing with all these kinds of program discontinuities. Now some of the things that the guest operating system may have to do to deal with these program discontinuities may require the guest operating system to have privileged access to the CPU. That is, certain instructions of the processor can only be executed in the kernel mode or the privileged mode. 

But recall that **the guest operating system itself is not running in the privileged mode**. It is above the red line which means it has no more privilege than a normal user-level process. This is a problem especially in a fully virtualized environment because **the fully virtualized environment the guest operating system has no knowledge that it does not have the privileges**. So when the guest OS tries to execute some instruction that can be executed only in privileged mode the expectation is that it'll trap, contact the hypervisor, and the hypervisor will then do the work that the fully virtualized guest operating system was intending to do. 

But here is where the quirks of the architecture come into play. Because unfortunately **some privileged instructions fail silently in some architectures when they're executed at the user level**. And this is a problem for the fully virtualized environment where the binary is unchanged. In the para-virtualized environment because we know that the para-virtualized guest is not running on bare metal we know that it does not have the same privilege as the hypervisor. And therefore we can collaboratively make sure that anything that the guest has to do in privileged mode, it can take the help of the hypervisor to do it. **But in a fully virtualized setting the guest has no knowledge**. So the only mechanism that will save the day is if the guest tries to execute a privileged instruction it'll trap into the hypervisor and the hypervisor can do the necessary thing. However because some privileged instructions when executed at the user level don't trap but fail silently. 

This happens in older versions of x86 architecture for privilege instructions executed in user mode. **Therefore the hypervisor has to be intimately aware of the quirks of the hardware and ensure that there isa  workaround such quirks**. Specifically in a fully virtualized setting **the hypervisor has to look at the unchanged binary of the operating system** and look for places where these quarks may surface and do binary rewriting in order to catch those instructions when and if they're executed on the CPU. 

Having said that I should mention that newer versions of Intel's architecture and AMD architecture for the x86 constructions have included virtualization support so that these kinds of problems don't occur. As a result of servicing the events that are delivered with a hypervisor the guest may have to do certain things, which may result in communication back to the hypervisor. In the case of a fully virtualized environment **communication from the guest to the hypervisor is always implicit via traps**. For example as a result of page fault servicing the guest may try to install a new entry into the page table. When it tries to do that that's a trap that will come in to the hypervisor and the hypervisor will take the appropriate action. In a para-virtualized environment the communication from the guest to the hypervisor is explicit so that API's that the hypervisor supports for the guest OS to communicate back to the hypervisor. 

What maybe reasons for that? Well. We talked about memory management done by the guest operating system in a pair of virtualized environment such as Xen where the guest operating system may have to tell the hypervisor here is a page table entry. Please install it in the page table for me. So that is the kind of communication that may have to come back from the guest operating system down to the hypervisor.

## 4. Device Virtualization Intro 

That completes discussion of issues that the hypervisor has to deal with for virtualizing the CPU. The next issue is **virtualizing devices**. Here again we want to give the illusion to the guest that they own the I/O devices.

## 5. Device Virtualization 

In the case of full virtualization the operating system thinks that it owns all the devices already. And the way devices are virtualized is the familiar trap and emulate technique. That is for the devices that the operating system thinks it owns when it tries to make any access to those devices, it's going to result in a trap into the hyperviser and the hyperviser will emulate the functionality that the Operating System intends for that particular device. 

In this sense there is not much scope for innovation in the way devices are virtualized in a fully virtualized environment. There are lots of details that the hyperviser has to worry about once the guest traps into the hypervisor to ensure the legality of the I/O operation and also whether the guest is allowed to make those I/O operations and so on but nothing fundamental conceptually is there in terms of device virtualization in a fully virtualized environment.

Para-virtualized setting is much more interesting. **The IO devices seen by the guest operating system are exactly the ones that are available to the hypervisor.** That is the set of hardware devices that are available in the platform are exactly the one that the para-virtualized guest operating system is going to be able to manipulate. 

This gives an opportunity for **innovating the interaction between the guest operating system and the hypervisor** in particular ways in which we can make **device virtualization more efficient** when we have this para-virtualized environment. 

So for one thing it is possible for the hypervisor to come up with clean and simple device abstractions that can be used by the para-virtualized operating system. Further through APIs it becomes possible for the hypervisor to expose shared buffers to the guest operating system, so that efficiently data can be passed between the guest operating system and the hypervisor and to the devices without incurring the overhead of copying multiple times, data from one address space into another. And similarly there can be innovations in the way event delivery happens between the hypervisor and the guest operating system. 

So in order to do device virtualization we have to worry about two things. 
- One is how to transfer control back and forth between the hypervisor and the guest, because devices being hardware entities, they need manipulation by the hypervisor in a privileged state. 
- And there is a data transfer that needs to be done because the hypervisor is in a different protection domain compared to the guest operating system. 

So these are two things that one has to worry about in device virtualization and we'll see how both control transfer and data transfer at accomplished in both the fully virtualized and the para virtualized settings.

## 6. Control Transfer

So control transfer in a fully virtualized setting happens implicitly from the guest to the hypervisor. How? 

When the guest operating system executes any privileged instruction–because it thinks it can do it–it'll result in a trap and hypervisor will catch it and then do the appropriate thing. That's how control is transferred from the guest to the hypervisor implicitly. 

And in the other direction control transfer happens as we already mentioned via software interrupts or events from the hypervisor to the guest. 
In a para-virtualized setting the control transfer happen explicitly via hypercalls from the guest into the hypervisor. I gave the you the example of page table updates that the guest operating system may want to communicate to the hypervisor through the API calls. When it executes the API call corresponding to that that results in control transfer from the guest into the hypervisor. 

And similar to the full virtualization case, in para virtualization in the other direction that is going from the hypervisor to the guest, it is done through software interrupts. So that's how control transfer is handled in both the fully virtualized and paravirtualized environments. 

The additional facility that you have in a paravirtualized environment is the fact that **the guest has control via hypercalls on when event notifications need to be delivered**. In the case of full virtualization since the operating system is unaware of the existence of the hypervisor, events are going to be delivered as and when they occur by the hypervisor to the guest. But in a para virtualized environments, the guest via hypercalls can indicate to the hypervisor that "leave me alone don't send me any event notifications now" or it can say "now is a good time to send me event notifications." So that level of control exists in a paravirtualized environment which doesn't exist in a full virtualized environment. **So this is sort of similar to an operating system disabling interrupts.** That's exactly the same facility that's available in a para-virtualized environment at the granularity of the operating system. The operating system can say that I don't want any event notifications. I'll come ask you when I need some event notifications.

## 7. Data Transfer

How about data transfer. Well once again when you think about full virtualization data transfer is implicit. Any data movement that has to happen between they hypervisor and the fully virtualized outputting system happens implicitly. 

In a para virtual setting for example in Xen there's an opportunity again to innovate because **you can be explicit about the data movement from the guest operating system into the hypervisor and vice-versa**. 

There are two aspects to resource management and accountability when it comes to data transfer. **The first is the CPU time**. So when an interrupt comes in for instance from a device hypervisor has to demultiplex the data that is coming from the device to the domains very quickly upon an interrupt. And there is CPU time involved in making such copies. And the hypervisor has to account for the computation time for managing the buffers on behalf of the virtualized operating system above it. 

The CPU time accountability is crucial from the point of your billing and data centers and therefore hypervisors pay a lot of attention to how CPU time is accounted for and charged to the appropriate virtual machines. 

**The second aspect of a data transfer is how the memory buffers are managed.** There is a space issue. The first is a time issue. CPU time issue. The second is a space issue. Meaning how are the memory buffers allocated? And how are they managed either by the guest operating system or by the hypervisor? 

As I said in the context the full virtualization that is little to scope for innovation. But in the context of paravirtualization there's a lot of scope for innovation in the way memory buffers are handled between the guest operating system and the hypervisor. Specifically looking at Xen as an instance of para virtualization, let's look at the opportunities for innovation in the guest OS to Xen communication. 

![[Pasted image 20250119180342.png]]

Xen provides asynchronous IO rings, which is basically a data structure that is shared between the guest and the Xen for communication. Any number of these data structures, that is, any number of these I/O rings can be allocated for handling all the device I/O needs of a particular guest domain. The I/O ring itself is just a set of descriptors. What you see here they're all descriptors that are available in this data structure. And it's a ring data structure that's why it's called IO ring. And we'll talk about how it is used in a minute. 

The idea is requests from the guest can be placed in this IO ring by populating these descriptors. Every descriptor is a unique I/O request coming from the guest operating system. So every request is going to have a unique ID associated with that particular request coming from the guest operating system. 

And recall what I said earlier that is the I/O ring is specific to a guest. So every guest can declare a set of I/O rings as data structures for its communication with Xen. 

Now in responding to the request from the guest, Xen, after it completes processing the request, it'll place a response back in the same I/O ring in one of these descriptors and that response is going to have the same unique ID that was used to communicate the request in the first place. 

So it's sort of a symmetric relationship between the guest and Xen in terms of request and response for things that the guest wants Xen to get done on its behalf, and for Xen to communicate back to the guest that has completed what was requested of them. 

![[Pasted image 20250119180358.png]]

So the guest is the producer of requests. So it has a pointer into this data structure that says where it has to place the next request. So this is a pointer that is manipulated meaning modified by the guest, but it is readable by Xen. So there's only one guy that can modify that is a guest. But Xen can look at this pointer. It's a shared pointer in that sense. 

![[Pasted image 20250119180422.png]]

For example the guest operating system may have placed new requests and that's why it has moved its pointer to here indicating that it has placed new requests. This is the next empty spot where it can place new requests after these two requests. 

![[Pasted image 20250119180452.png]]

The consumer for the request produced by the guest operating system is Xen. And it is processing requests in the order in which they've been produced by the guest operating system. Therefore it has the pointer into this IO ring data structure saying where exactly it is presently servicing requests from this guest operating system. 

So right now the pointer is indicating **that Xen is yet to process these two requests that have been queued by the producer**–the producer being the guest operating –into this I/O ring data structure. So this pointer that the request consumer has is private to Xen. It is just telling Xen where it is in processing requests from the producer. And the difference between these two pointers is telling how many requests are outstanding so far as Xen is concerned in terms of processing requests emanating from the guest. Similar to request production Xen is going to be the guy that is offering responses back to the guest operating system. **So Xen is the response producer**. So it's going to have a pointer where it can place new responses.

![[Pasted image 20250119180630.png]]

So once again this pointer is a shared pointer updated by Xen but can be read by the guest. These are two new responses that Xen has placed in the I/O Ring in response to request that it has already processed **but these responses have not yet been picked up by the producer.** 

The guest operating system has a pointer that says where it is in this I/O ring data structure in terms of picking up the responses coming from Xen. Xen has produced these two new responses as a result of processing some prior requests from the guest 
operating system but the guest operating system is yet to process this request. 

![[Pasted image 20250119180718.png]]

So this pointer is private to the guest operating system that is telling it where it is in processing responses coming back from Xen. The difference between the producer and consumer response pointers is the number of responses that are yet to be picked-up by the guest operating system. 

Just to recap the difference between the producer and consumer request pointers is the number of requests that are yet to be processed by Xen. And the difference between these two pointers is the number of responses that are yet to be picked-up by the guest. 

![[Pasted image 20250119180831.png]]

And these slots are the empty slots into which new requests can be placed by the request producers. **All of the pointers move in a counter-clockwise fashion,** filling empty slots or pulling off the next slot. And these are the empty slots into which Xen can place additional responses as it processes more requests from the guest operating system. That's the idea behind this I/O ring data structure. Very powerful extraction that allows the guest to place requests asynchronously, and for Xen to place responses asynchronously into the data structure. As I mentioned already the guests can identify the request for which a particular response is intended because the id that is associated. With the response is going to be exactly the same as the ID that was associated with the request in the first place. 

The other thing that we should remember is that **these are just descriptors of the requests and responses**. Any data that has to be passed from the guest to Xen will just be a pointer from these descriptors to a machine page that the guest owns so that Xen can pick up the data directly from that machine page without any copy. Similarly if the responses given by Xen is going to have data associated with it, that will again be a pointer from the descriptive data structure to the machine page that contains the data that Xen wants to pass up to the guest. In other words this asynchronous I/O rings is a powerful mechanism both for efficient communication between the guest and Xen, and for also avoiding copying overhead between the para-virtualized guest operating system and Xen which is the virtualization layer. 

We will look at two specific examples on how these I/O rings are used for guest and Xen communication.

## 8. Control and Data Transfer in Action 

The first example that we'll look at is how control and data transfer is affected in zen for a network virtualization. 

## Network Virtualization 

![[Pasted image 20250119181119.png]]

Each guest has two I/O rings–one for transmission and one for reception. If the guest wants to transmit packets it enqueues descriptors in the transmit I/O ring. And placing these descriptors in this IO ring is done via hypercalls that Xen provide for guest operating systems. 

![[Pasted image 20250119181216.png]]

The packets that need to be transmitted are not copies into Xen, but the buffers that contain these packets are in the guest operating system buffers. And what the guest does is it **embeds pointers to these buffers in the descriptors that have been enqueued for transmission by hypervisor**. So in other words there's no copying of the package themselves from the guest operating system buffers into Xen. Because the pointers to the guest operating system buffers have been embedded in these descriptors. 

![[Pasted image 20250119181310.png]]

And for the duration of the transmission the pages associated with these network packets are pinned so that Xen can complete the transmission of the packets. Now this is showing interaction of one guest operating system with Xen. Of course more than one VM may want to transmit. And what Xen does is it adopts a round-robin packet scheduler in order to transmit packets from different virtual machines. 

![[Pasted image 20250119181350.png]]

Receiving packets from the network and passing it to the appropriate domain works exactly similar to what I described for transmission except in the opposite direction. What Xen does when it receives a network packet intended for a particular guest it exchanges the received packet for one of the guest operating system page that has been already provided to Xen as the holding place for incoming packets. In other words in order to make things efficient, what a guest operating system will do is pre-allocate network buffers, which are pages owned by the guest operating system. So when a package comes in Xen can directly put the network packet into the buffer that is owned by the guest operating system and enqueue a descriptor for that particular guest operating system. 

So once again we can avoid copying in the direction of going from Xen to the guest operating system. So one of the cute tricks that xen also plays is that when it receives a packet into a machine page, then it can simply exchange that machine page with some other page that belongs to the guest operating system. And there again is another trick to avoid copying. Either the guest can pre-allocate a buffer in which case Xen can directly put that packet into the pre-allocated buffer. Or if Xen receives a packet into a machine page it can simply swap that machine page for a page that the guest already owns so those are two different techinques for avoiding copying altogether in the reverse direction as well.

## 9. Disk I/O Virtualization 

![[Pasted image 20250119181505.png]]

Disk I/O virtualization works quite similarly. Every VM has an I/O ring which is dedicated for disk I/O. This is an I/O ring associated with VM1. This is an I/O ring associated with VM2. 

Similar to network virtualization. Here also the communication between the guest operating system and Xen strives to avoid copying altogether. No copying into Xen because what we're doing is we are enqueuing descriptors for the disk I/O that we want to get done with pointers ro the guest operating system buffers where the data is already contained and the transfer has to go into the disk, or a placeholder for the data to come into from the disk. And the philosophy being asynchronous I/O. Enqueuing these descriptors into this I/O ring by the guest operating system happens asynchronously with respect to Xen enqueuing responses back for prior requests coming from this VM. Since Xen is in charge of the actual devices in particular in this case the disk subsystem, it may reorder requests from competing domains in order to make the I/O throughput efficient. 

But there may be situations where such a request reordering may be inappropriate from the semantics of the I/O operation. And therefore Xen also provides a reorder barrier for guest operating systems to enforce that "do this operations in exactly the order in which I've given them." Such a reorder barrier for instance may be necessary for higher level semantics such as write ahead logging and things like that. 

So that completes discussion of all the subsystems that need to be virtualized whether it is by a fully virtualized environment or a para virtualized environment. Next we will talk about usage and billing.

## 10. Measuring Time 

The whole tenant of utility computing is that resources are being shared by several potential clients. And we've gotta have a mechanism for billing them so it's extremely important that we have good ways of measuring the usage of every one of these resources: CPU, Memory, Storage, and Network. And virtualized environments have mechanisms for recording both the time and the space usage of the resources that it possesses so that it can accurately bill the users of the resources.

## 11. Xen and Guests

![[Pasted image 20250119181717.png]]

I want to conclude this course module with a couple of pictures. One showing Xen and guests supported on top of it. And this is from the original paper which shows what are all the resources available. And how they are virtualized and how guest operating systems may sit on top of it. With a similar picture for VMware. As you know Xen is a paravirtualized environment. VMware is a fully virtualized environment. And there's a picture showing how VMware allows different guests to sit on top of its virtualization layer. 

The main difference of virtualization technology from extensible operating system that we have seen in the earlier course module is in the case of virtualization technology whether we are talking about para-virtualization a la Xen, or a  full virtualization a la VMware. **The focus is on protection and flexibility**. Performance is important but we are focusing on protection and flexibility and making sure that we are sharing the resources not at the granularity of individual applications that may run on top of the hardware, but at the granularity of entire operating systems that is running on top of the virtualization layer.

## 12. CPU and Device Virtualization Conclusion 

In closing we covered a lot of ground on operating system structuring. We saw how the vision of extensible services which had its roots in the HYDRA operating system and the virtualization technology for hardware that started with IBM VM/370 have all culminated in the virtualization technology of today that has now become mainstream. Data centers around the world are powered by this technology. All the major IT giants of the day from chip makers to box makers to service providers are all in this game. Processor chips from both Intel and AMD have started incorporating virtualization support and hardware that makes it easier to implement the hypervisor, especially for overcoming architectural quirks that we discussed in the context of VM were like full virtualization. Even Xen which started out with paravirtualization, has a newer version that exploit such architectural features of the processor architectures to run unmodified operating systems on top of Xen a technique that they call hardware-assisted virtualization.