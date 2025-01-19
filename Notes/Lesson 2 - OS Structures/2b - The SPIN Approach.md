Note: [[Extensibility, Safety and Performance in the SPIN Operating System.pdf]] This section refers directly to the Bershad paper on Extensibility, Safety and Performance in the SPIN Operating System.

## 1. The SPIN Approach Introduction 

## 2. What are we Shooting for in OS Structure? 

We want the operating system structure to be thin and resemble a microkernel. 
- Only mechanisms should be in the kernel, and 
- no policies should be ingrained into the kernel itself. 

It should also resemble the DOS-like structure: 
- Allow fine-grain access to system resources.
- Minimize border-crossing as much as possible when accessing resources. 

Should also be flexible. 
- Resource management can be easily morphed to suit the needs of the application, **without** sacrificing performance and protection. 

In short: performance, protection, and flexibility. 
## 3. Approaches to Extensibility 

The Hydra operating system in 1981 is one example of interest in extensibility starting in 1981.

It provided kernel mechanisms for resource allocations. 

It had capability-based resource access. 
- Capability has a special connotation in operating systems. It refers to an entity that can be passed from one to the other. Cannot be forged, can be verified--all of the things that you want in order to make sure that system integrity is not compromised. Capability captures these notions. 
- Capability was a heavyweight mechanism in terms of implementing it efficiently in an operating system. 

**Resource managers** were built as **course-grained objects** to reduce border-crossing. 
- Border-crossing in the Hydra system meant that you have to pass capability from one object to another, and validate capability for entering a particular, etc. 
- For that reason, Hydra used coarse-grained objects to implement resource managers. That way, it can reduce border-crossing overhead. 
- This limits opportunities for customization and extensibility. 
	- The coarser you make these objects, the less opportunities available for customizing the services. This is another strike against monolithic kernerls.

In principle, Hydra provided the minimum required mechanisms in the kernel, and having the resource managers implement policies, because the fundamental mechanism for accessing the resources was through this capability, which was a heavyweight abstract notion to implement efficiently, in practice, Hydra did not fully achieve its goal of extensibility. 

The most well-known extensible operating system of the early 90s was the Mach operating system from CMU. 
- Mach was micro-kernel based, 
- provided limited mechanisms in the microkernel, 
- and implemented all the services expected from an operrating system as server processes that ran as normal user-level processes above the kernel. 

With this microkernel based approach, Mach achieved its goal of extensibility and portability. 

However, performance now took a backseat. 
- Mach was focused on making the operating system portable across different architectures in addition to extensibility. 

This led to micro-kernel based design becoming associated with poor performance. However L3 kernel-based design improves upon these problems introduced by Mach. 

The key idea in SPIN is to co-locate a minimal kernel with its extension in the hardware address space, and avoid border crossing between the kernel and the extensions of the kernel that are containing the specific services that the applications need. 
This co-location also means that we avoid the border crossing, which is said to be one of the biggest potentials for losing out on performance. 

If we are going to co-locate the kernel and extensions in the same hardware address space, isn't that comproomising on protection? (See: disadvantages on DOS-like structures).  

The approach that SPIN took is to **rely on the characteristics of a strongly-typed programming language (Modular 3)** so that the compiler can **enforce the modularity** that we need in order to give guarantees for protection. 
- By using a strongly-typed language, the kernel is able to provide **well-defined interfaces** (think of declaring function types in a header file, separated from the implementations of these functions in a software project). 
	- In this case, an operating system is a complex piece of software. Why not use a strongly-typed language to build an operating system? That is the main foundation for the SPIN approach. 

When you use a strongly typed language, you cannot cheat data structure types. 
- Data abstractions provided by the programming langauge, such as an object, serve as **containers** for logical protection domains. 
	- We are no longer relying on hardware address spaces to provide the protection between services and the kernel. 
	- The kernel only provides the interfaces, while these logic protection domains actually implement the functionality that is enshrined in those interface functions. 
	- As a result, there can be several implementations for these interface functions. Applications can dynamically bind to different implementations of the same interface functions, which leads to different instantiations of specific system components, providing the flexibility that we want in constructing an operating system. 

**Because we are co-locating the kernel and extension in the same hardware address space, we are making extensions as cheap as a procedure call.** 

We are writing on the characteristics of a strongly-typed programming language which enforces strong typing, allowing the OS designer to implement logical protection domains instead of relying on hardware address spaces. 

## 4. Logical Protection Domains 

Modula 3 is a **strongly-typed language** with built-in safety and encapsulation mechanisms.
- It does automatic management of memory--so there are no memory leaks.
- Data abstractions supported in Modula 3: 
	- Objects
		- Has well-defined entry points. 
			- Only the entry points are known outside the object--not the implementation of the code for that entry point, or the data structure contained within that object. 
	- Threads
		- Execute in the context of the object.
	- Exception
		- IE: raises exceptions when there is a memory access violation. 
	- Generic interface
		- Exposes the externally-visible methods inside an object. 

What you can do from outside the object is what the entry point methods inside the object allow and nothing more. We are gating the safety property of a monolithic kernel without having to put system code in a hardware address space. 

In other words, **the logical protection domains give us two of our primary goals in operating systems design: protection and performance.** What about flexibility?

Flexibility is provided through the **generic interface mechanism**, which allows us to have multiple instances of the same service. A given application may be able to exploit the different instances of available services which cater to the same generic interface.

And objects that implement specific services can be at the desired granularity of the system designer, where they can be fine-grained, or a collection. 
- For example, individual hardware resources can be a fine-grained object, such as a page frame and its relevant functions 
- Alternatively, the same service can be implemented by interfaces which provide a certain functionality. In this example, a page allocation module can be an object 
- We are also free to make a collection of interfaces into an object--an entire virtual memory subsystem can be an object that is hierarchically composed of a page allocation module, where you can define hardware resources as objects inside of the module. 

Whether at the coarse-level of a collection of interfaces, or an individual interface that is a component of a collection, or specific hardware resources, are accessible via *capabilities*. 
- While capabilities traditionally refer to heavyweight implementations with poor performance, because we are dealing with a strongly-typed language, capabilities to objects can simply be supported as pointers.
- In other words, programming language-supported pointers can serve as capabilities to the objects. 
So now, with this idea, access to the resources--that is, entry point functions within an object that is representing a specific resource--is provided via capabilities that are simply language-supported pointers. 

Because they are language-supported pointers, these capabilities are much cheaper compared to real capabilities, as was used in the Hydra operating system. 

## 5. Pointers 

## Quiz: The SPIN Approach: Pointers 

Differentiate between pointers in C and pointers in Modula-3. Select from the following choices: 
- No difference 
- C pointers are more restricted
- Modula-3 pointers are type-specific 

Pointers in Modula-3 cannot be forged--there is no way to subvert the protection mechanism built into the language.
For example, if I have a data structure defined in Modula-3, and I have a pointer to that data structure, the only way we can use that pointer is as a pointer to that type of data structure. It is impossible to cast a data structure in the appearance of a different data structure. This is what allows users to implement logical protection demains using objects and capabilities to objects as pointers supported by the programming language. 

## 6. Spin Mechanisms for Protection Domains

There are three mechanisms in SPIN to create and use protection domains. 

### Create 
Allows the creation of protection domains. This allows initiating an object file with the contents and exporting the names that are contained as entry point methods inside the object to be visible outside of it. That is what this create call provides to a service creator. 
- FOr example, if creating a memory amangement service, we can write the functions inside of our memory management serfices and export the names using the create mechanism.

### Resolve 
If one protection domain wants to use the names that is there in a nother protection domain, the way it can accomplish that is using the resolve primitive. 
- This is very similar to linking two separately compiled files together that a compiler does routinely. You may be very familiar with the compilation process where you separately compile files, and then go through a link phase of the compiler, where the linker resovles the names being used by one object file with the names defined in another object file. That is the same thing that Resolve does--it resolves the names being used in source, the logical protection domain, into the target logical protection domain. 
- The source LPD an the target LPD are thus dynamically linked. Once linked / bound together, accessing methods located int he target protection domain is as efficient as a procedure call, and thus performed at memory speeds. 

### Combine
To reduce a proliferation of small logical protection domains, we may want to combine them into an aggregate logic protection domain. 
- Once the names in a source and target protection domain have been resolved, they can be combined into an aggregate logic protection domain. The aggregate logic protection domain will have entry points that are the union of entry points that were exported as names from the source, target, or any such domains combined into the aggregate domain. 
- Combine is useful as a software engineering management tool to combat the proliferation of many small domains. 

The roadmap for creating services is: 
1. Write code as a Modula 3 program with well-defined entry points.
2. Using SPIN mechanism of `create`, instantiate a service and export the names available in that service. 
	1. If another logical protection domain wants to use exported names, it can do so using the SPIN mechanism `resolve` that causes the dynamic binding of the source and target logical protection domains. 
3. Finally, perform aggregation of logical protection domains with `combine` , which creates a union of al entry points contained in component logical protection domains. 

This is how we provide protection and performance while allowing for flexibility--by using a strongly-typed language that performs compile-time checking and run-time enforcement of the logical protection domains. 

## 7. Customized OS with Spin 
![[L02B_07.png]]

The upshot of the logical protection domain is the ability to extend SPIN to include operating system services, and make that all part of the same hardware address space. This means: no border crossing between the services or the mechanisms provided by SPIN. 

For example, here is one example where all these system services are all implemented as protection domains using `create`, `resolve` and `combine`. 
We've created all these services as logical extensions of SPIN. Here is another extension living on top of the same hardware concurrently with the first extension. 

As you see, each of these mounds represent a completely different operating system. Each of these mounds may have their own subsystems for the same functionality. For instance, P2 may use memory manager 2, while P1 uses memory manager 1. Both of them implement the same functionality, but very differently (hopefully) to cater to the needs of the applications that need those services. 

But they may also have common subsystems. The network protocol stack may be used by both extensions which live on top of the same hardware framework. 
## 8. Example Extensions 

![[L02B_08.png]]

Here is a concrete example of an extension (app, unix OS, ext interface, SPIN). It is a fairly standard implementation, let's say, of a UNIX operating system, but it is implemented as an extension on top of SPIN. 

Here is a more fun example--a client-server application that is implemented directly on top of SPIN as an extension. In other words--there is no operating system. Here, a display client uses an extension interface to implement the functionality for displaying video that is going to be seen by a video server. Both of these are extensions on top of basic spin and provide the basic functionality eneded for a video server application. 

The bounding box is showing SPIN and the extensions thereof. In the first case, it is the entire operating system; in the second case, it is just the video server and the display client, each running on top of SPIN directly. 

## 9. Border Crossings 
### Quiz: The SPIN Approach: Border Crossings 
![[L02B_09.png]]
Given 3 different structures below: 
1. A monolithic structure, 
2. a micro-kernel base structure, 
3. and the SPIN structure with extensions,

Which structures will result in the smallest number of border crossings? 
- Monolithic
- Microkernel
- SPIN
- Either SPIN or monolithic 
### Answer 
Either SPIN or monolithic will result in the least number of border-crossings. 

In the micro-kernel based structure, we are assuming that each service is available as a server process in their own hardware address space. 
- Therefore, any system service that an application needs may need to go through multiple border-crossings. This means going across different address spaces, and the incurring an inherent loss of locality.

In the case of monolithic structures, there are only two border crossings--one border-crossing to go from the application to the monolithic kernel, and another border-crossing to return to the application. 

In SPIN, since we are extending it with services, they are all contained in the same hardware address space. Though we may  be going through several protection domains and satisfying the system call emanating from an application, those protection domains are logical protection domains, and do not invovle border crossing, whicn involves change of locality and thus loss of performance. 

## 10. Spin Mechanisms for Events 
![[L02B_10.png]]

An operating system has to feel external events. For example--external interrupts that may come in when a process is executing, or the process itself may raise some exceptions such as a page fault, or make system calls. 

These are all events that must be fielded by the operating system, and SPIN supports such external events using an **event-based communication model**. 

Services can register what are called **event-handlers** (labeled `h1, h2, h3`) with the SPIN event dispatcher, and SPIN supports several types of mapping:
- a one-to-one mapping between an event (`e`) and a handler, 
- or a one-to-many mapping between events to handlers, or 
- many-to-one mappings between many events and the same handler. 

One concrete example to illustrate the power of this mapping can be a typical protocol stack. 
There are several different interfaces available on your machine--a network protocol packet may arrive through one of several interfaces.

Say that you have an Ethernet packet, and an ATM packet. A network packet can arrive on either Ethernet or ATM. This packet might be an IP event, which will register with an IP handler in order for the event to be seen. 
- This is one example of many-to-one mapping between two events and a single handler. 

The processing of the packet by this IP handler results in an IP packet arrival event, where several clients on the IP layer of the protocol stack (ICMP layer, UDP, TCP) can be sitting in this network layer. 
- Here, this is an example of one-to-many mapping between one event to several handlers that need to be triggered. The SPIN dispatcher allows any number of handlers as registerd to handl ea s pecific event type. All handlers associated with that event type will be scheduled byt he SPIN dispatcher. 

The order in which event handlers get scheduled is not something the designer can count on, because SPIN has the freedom in the ordering of scheduling event handlers. But all the handlers that are associated with an event will get triggered when an event is raised. 

If the packet is an ICMP packet, then it raises an event that the ICMP packet has arrived, which may be handled by a single client, such as the ping program. 

Event handlers may also be specified with guards for finer grained handler execution. 
- For example, this handler may specify that only when IP packets arrive it can be executed. This is a guard that the IP packet handler can specify, so that the handler will only be triggered when the packet received by different interfaces is an IP packet. 

## 11. Default Core Services in SPIN 

We now know what SPIN provides for building an operating system. One can build each of the necessary operating system services--memory management, CPU scheduling, threads, file system, network protocol stack, etc--as extensions to SPIN. Here, memory management and CPU scheduling are core services that any operating system should provide. 

But an extensible operating system should NOT dictate how these services should be implemented. SPIN provides interface procedures for implementing these services in the operating system. 
### Physical Memory 

Native operating systems, such as Linux or Windows, manages the physical memory that is available from the hardware. **SPIN wants to allow extensions to manage physical memory allocated to them in whatever fashion they choose to**. The **macro-allocation** of a bunch of physical memory to an extension is not within scope of this discussion, but assume that the allocation of a bunch of physical memory happens when an extension starts up. 

The management of the pre-allocated physical memory by the extension is discussed below. 

Memory management functionality is provided by interface functions, or header files, provided by SPIN, such as: 
- Physical address
	- allocate, deallocate, reclaim a page frame
- Virtual address
	- allocate, deallocate (dynamic memory allocation)
- Translation
	- create/destroy address space, add/remove mapping between virtual pages and physical frames
- Event handlers are also provided for the following external events
	- page fault, access fault, bad address

These are interface functions that are defined as core services of memory management in the SPIn operating system. SPIN does not define how these services are implemented, but rather provides the functions as an entry point in a header file. It is the responsibility of the designer for an extension to write the code for these header functions and provide a logical protection domain that corresponds to physical address management, virtual address management, translation management, and the handler functions for dealing with these different types of events. 

Once the LPD is dynamically instantiated, it becomes an extension for  SPIN. After that, there is no border crossing between a particular service and SPIn itself. All of these functions are invoked automatically when the hardware events occur. 

## 12. Default Core Services in SPIN (cont) 

SPIN also arbitrates another precious resource, that is, the core service of CPU Scheduling. 

### CPU Scheduling 

SPIN only decides at a macro level, the amount of time given to a particular extension, which is done through the global scheduler. The **SPIN global scheduler** interacts with the application threads package, or the extension living on top of SPIN. Recall that an extension can be an entire operating system, or an application. 

For example, say we are running Linux and Vista as two extensions on top of SPIN. Each may be given a particular time-slice of `x` milliseconds. However, *how* each extensions uses the time given to it for scheduling user-level processes running inside the operating system is up to those extensions completely.

#### STRAND 
To support the concept of threads in the operating system and management of time, SPIN provides an abstraction called STRAND. 

The actual operating systems that extend SPIN will have threads map to strands. A strand is the unit of scheduling that SPIN's global scheduler uses, but the semantics of strand is entirely decided by the extension. 
- For example, for an implementation of pthreads, the semantics of the strand will simply be the semantics of pthreads' scheduler. 

There are also event handlers that facilitate the scheduling which needs to happen inside of a given extension. For CPU scheduling, SPIN provides event handlers for `block`, `unblock`, `checkpoint`, and `resume`. 
- Once again: the extension's event handlers must give the semantic meaning of what takes place when these event handlers are called, because SPIN provides only the interface functions of these event handlers. What needs to happen is up to the extension. 
	- For example, a disk interrupt handler may result in an `unblock` event being raised for a particular strand that was waiting for the disk IO completion. 
	- Similarly, if an application were to make a system call that was a blocking system call, then the service that provides this facility to the application will raise this `block` event, which will result in the extension taking the appropriate action of saving the state of the currently running process and putting it in the appropriate queues to wait for the system call completion. 

In short, SPIN provides the primitives that may be needed by an extension that needs to provide CPU scheduling services. However, it only provides the interface definitions. The semantics behind how scheduling is affected is up to the extensions.

All that SPIN concretely defines is the time an extension receives on the CPU through this global schedule that SPIN has for allocating time to different extensions concurrently living on top of SPIN. 
## 13. The SPIN Approach Conclusion

There are, however, some deep implications that arise as a result of the SPIN approach. Some services, particularly core services (which are trusted services, since they provide access to hardware mechanisms) may need to step outside of the language-enforced protection model to control hardware resources. 

**In other words, applications that run on top of an extension have to trust the extension.** 

Extensions to core services affect *only* the applications that use that extensions--that is, it does not affect other applications that do *not* rely on this particular extension. 