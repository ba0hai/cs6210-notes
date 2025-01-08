
## 1. Intro to Virtualization Introduction 

The drive for extensibility in operating system services led to innovations in the internal structure of operating systems, as well as dynamic loading of operating system modules in modern operating systems. In this lesson, we will see how **the concept of virtualization** has taken the **vision of extensibility** to a whole new level, namely allowing **the simultaneous co-existence of entire operating systems** on top of the **same** hardware platform.
## 2. Quiz - Virtualization 

Let's start with a fun quiz to get you warmed up. You may have heard the term virtualization in different contexts. What I want you to do select all the context in which you've heard the term. There is no right answer for this and in fact there are several that might fit into this. Go ahead and pick the choices that you think jogs your memory when you hear the term virtualization.
- Memory systems
- Data centers
- JVM
- VirtualBox
- IBM VM 
- Google glass
- Cloud computing
- Dalvik
- VMWare Workstation
- "Inception"

---

I'm not including the answers here, it's a "fun" quiz

## 3. Platform Virtualization 

The words virtual and virtualization are popular these days. From virtual worlds and virtual reality to applications level virtual machines like Dalvik or Java's virtual machine. What we are concerned with here are virtual platforms. And by platform we mean an operating system running on top of some hardware. 

Let's say Alice has her own company Alice Inc. And she has some cool applications to run for productivity. And let's say that they're running on a Windows machine on some server that the company maintains. If cost were not an issue, then this will be the ideal situation for Alice Inc. 

The hope of virtualization is that we will be able to give a company perhaps not as large as Alice Inc., IE Bala Inc., a similar experience at a fraction of the cost. So instead of a real platform that Alice Inc. has Bala Inc. gets a virtual platform.As Bala Inc. is concerned they don't really care what goes on inside this black box. All they want is to make sure that the apps that they want to run can run on top of this virtual platform. 

In other words, as long as the platform affords the **same capabilities** and the **same abstractions** for running the applications that Bala Inc. wants, the company is satisfied. 

![[L03A_01.png]]

But operating system designers are very much interested in what goes on inside this black box. We want to know how we can give Bala Inc. the illusion of having his own platform without actually giving him one. And including all of the cost the implementation and maintenance associated with implementing such a virtual platform.

## 4. Utility Computing 

Now if we peak inside the black box however we find that it is not just Bala who is using the resources in the virtual platform but there is also Piero and there is also Kim and possibly others who are also running their own applications and operating systems on the same shared hardware resources. 

![[L03A_02.png]]

Why would we want to do this? **Sharing hardware resources** across **several different user communities** is going to result in **reducing the cost of ownership and maintenance of the shared resources**. 

The fundamental intuition that makes sharing hardware resources across the diverse set of users is the fact that **resource usage occurs in bursts**.  

Let's take one particular resource usage for instance memory. 

![[L03A_03.png]]

Now adding all of these dynamic needs of different user communities, we may see a cumulative usage pattern that might resemble the chart above. 

Now let's consider Bala's cost. If he were to buy his own server then he would have to buy the option that corresponds to slightly above his peak usage, with additional resources as a buffer. 

**The virtual machine actually has a total available memory that far exceeds the individual needs** of all parties involved, and all parties can **share the cost of the total resources among themselves**. 

On a big enough scale what this would mean is that **all of the aforementioned parties** would potentially have access to a lot more resources. Then they can individually afford to pay for at a fraction of the cost because both the cost of acquiring the resource as well as maintaining it and upgrading it and so on is borne collectively. And that in a nutshell is the ideal goal behind utility computing. This is how AWS, Microsoft, and so on provide access to resources on a shared basis for a wide clientele. 

Virtualization is **the logical extension** to the idea of **extensibility** or **specialization of services** studied in the SPIN and Exokernel papers. But here, it is applied at a much larger granularity–namely in the form of an entire operating system. In other words, **virtualization is extensibility applied at the granularity of an entire operating system** as opposed to **individual services** within an operating system.

## 5. Hypervisors 

How is it possible to have **multiple operating systems** running on the same shared hardware resources? How are the operating systems protected from one another and who decides who gets the resource and at what time? This is handled by an operating system of operating systems, also known as a virtual machine manager (VMM) or **hypervisor**. 

![[L03A_04.png]]

Where the hypervisor is referred to as a VMM, the operating systems that are running on top of the shared hardware resources are often referred to as **virtual machines.** 

> **Note:** While VM can refer to virtual memory throughout our lessons, here, VM refers to virtual machine. And throughout this lesson, VMs can also be known as gust operating systems. 
 
There are **two types** of hypervisors. 

### Native Hypervisor 

The first type is what is called a native hypervisor or bare metal meaning that **the hypervisor is running on top of the bare hardware**. And that's why it's called a bare-metal hypervisor or a native hypervisor. And all of the operating systems that I'm showing you inside of the black box are running on top of this hypervisor. They're called the **guest operating systems** because they're **guests of the hypervisor** running on the shared resource. 

### Hosted Hypervisor

![[L03A_05.png]]

The second type of hypervisor what is called the hosted hypervisor, which will run on top of a host operating system allow the user to emulate the **functionality** of other operating systems. The hosted hypervisor is not running on top of bare metal, but it is running as an **application process** on top of a host operating system. And the guest operating systems are themselves clients of this hosted hypervisor. 

Some examples of hosted hypervisors include **VMware Workstation** and **Virtual Box**. Both of these terminologies you may have heard of. If you don't have access to a computer that's running Linux operating system in this cours, you're likely to be doing your course projects on a virtual box or a VMWare workstation that's available to run on Windows platform. 

For the purpose of this lesson today however we will be focusing on the bare metal hypervisors. These **bare metal hypervisors** interfere **minimally** with the **normal operation of these guest operating systems** on the shared resources, and function similarly to the extensible operating systems that we studied earlier like the SPIN and Exokernel. Therefore, the bare metal hypervisors offer the **best performance** for the guest operating system on the shared resource.

## 6. Connecting the Dots 

The concept of virtualization was studied many decades ago. It started with IBM VM 370 in the 60s and the early 70s. And the intent behind IBM VM370 was to give the illusion to every user on the computer as though the computer is theirs. This laid the foundation for binary support for legacy applications that may run on older versions of IBM platforms. And then of course we had the microkernels that we have discussed in the earlier course module which surfaced in the 80s and early 90s. That in turn gave way to extensibility of operating systems in the 90s. 

The Stanford project SimOS in the late 90s **laid the basis for the modern resurgence of virtualization technology** at the operating system level and in fact was the basis for VMware.

Even the specific ideas we're going to explore in this course module through Xen and VMware are papers that date back to the early 2000s. They were proposed from the point of view of supporting **application mobility**, **server consolidation**, **co-locating hosting facilities**, and **distributed web services**. These are all sort of the candidate applications for which Xen and VMware were positioning this virtualization technology. 

Today virtualization has taken off like anything. Why the big resurgence today? Well companies want a share of everybody's pie. One of the things that has become very obvious is **the margin for device making companies in terms of profits is very small**. And everybody wants to get into **providing services for end users**. This is pioneered by IBM and others are following suit as well. 

The attraction with virtualization technology is that **companies can now provide resources with complete performance isolation** and **bill each individual user separately**. Companies like Microsoft, Amazon, HP, etc: everybody's in this game of **wanting to provide computing facilities through their data centers to a wide diversity of user communities**. So that it's a win-win situation for both the **users that do not want to maintain and upgrade their own computing infrastructure** , while companies like IBM have a way of **providing these resources on a rental basis** on a utility basis to the **user community**. 

You can see the dots connecting up from the extensibility studies of the 90s like the SPIN and Exokernel to virtualization today that is providing computational resources similarly to companies that provide electricity and water. In other words **virtualization technology has made computing very much like the other utilities** that we use, leading to a huge resurgence in the user virtualization technology in all the data centers across the world.

## 7. Full Virtualization 

One idea for this virtualization framework is what is called full virtualization and in full virtualization the idea is to **leave the operating system pretty much untouched**. So you can run the unchanged binary of the operating system on top of the hypervisor. 

![[L03A_06.png]]

This is called full virtualization because the operating system is completely untouched. Nothing has been changed. Not even a single line of code is modified in these operating systems in order to run on the hypervisor simultaneously. However, some workarounds need to take place. **Operating systems running on top of the hypervisor are run as user-level processes.** They're not running at the same level of privilege as a Linux operating system that is running on bare metal. 

But if the operating system code is **unchanged**, it doesn't know that **it does not have the privilege for doing certain things that it would do normally on bare metal hardware**. 

In other words, **when the operating system executes some privileged instructions**, meaning they have to be in a privileged mode or kernel mode to run on bare metal in order to execute those instructions. Those instructions will create a **trap** that goes into the hypervisor and the hypervisor will then **emulate the intended functionality** of the operating system. 

And this is what is called **the trap and emulate strategy**. Essentially, **each operating system thinks it is running on bare metal**. And therefore it does exactly what it would have done on a bare-metal processor meaning that **it'll try to execute certain privileged instructions** thinking it has the right privilege. But it does not have the right privilege because it's run as a user-level process on top of the hypervisor. And therefore when they try to do something that requires. A high level of privilege, it will result in **a trap into the hypervisor** and **the hypervisor will then emulate the intended functionality of the particular operating system**. 

There are some thorny issues with this trap and emulate strategy of full virtualization and that is, in some architectures, **some privileged instructions may fail silently**. What that means is you would think that the instruction actually succeeded but it did not. And you may never know about it. In order to get around this problem in fully virtualized systems, **the hypervisor will resort to a binary translation strategy** meaning that the hypervisor is aware of instructions that fail silently in the architecture, and search for those instructions in each of these individual binaries of the unmodified guest operating systems. Through the **binary translation strategy**, the hypervisor will ensure that those instructions are caught upon failure and properly handled to produce the appropriate effect. 

This was a problem in early instances of Intel architecture. Both Intel and AMD have since started adding virtualization support to the hardware so that such problems don't exist any more. But in the early going when virtualization technology was experimented with in the late 90's and the early 2000s, this was a problem that virtualization technology had to overcome in order to make sure that operating systems can be run as unchanged binaries on a fully virtualized hypervisor. Full virtualization is the technology that is employed in the VMware system.

## 8. Para-Virtualization 

Another approach to virtualization is to modify the source code of the guest operating system. If we can do that not only can we **avoid problematic instructions** as discussed in full virtualization, but we can also include optimizations, such as:
- Allowing the guest operating system to see **real hardware resources** underneath the hypervisor,
- Providing access to real hardware resources,
- Being able to employ tricks such as **page coloring**, 
- and exploiting the characteristics of the underlaying hardware. 

It is important to note however that so far as the applications are concerned, **nothing is changed about the operating system** because the interfaces that the applications see is **exactly** the interfaces provided by the operating system.

If there is an application that is running on Windows it sees the same API as it would if this Windows operating system was running on native hardware, and if the application is running on top of Linux it sees exactly the same API as it would if this Linux operating system was running on native hardware. 

In this sense, **there's no change to the applications** themselves. But **the operating system has to be modified** in order to account for the fact that it is not running on bare metal but as a guest of the hypervisor. 

![[L03A_07.png]]

And this is why this technology is often referred to as. Para virtualization meaning it is not fully virtualized but a part of it is modified to account for being a guest of the hypervisor. 

The Xen product family uses this para-virtualization approach. Now this brings up an interesting question. We mentioned that, in order to do this para-virtualization, we have to modify the operating system–but how big is this modification?

## 9. Quiz -  Modification of Guest OS Code 

**What percentage of the guest operating system code may need modification with para virtualization?**
- 10% of the code base has to be changed 
- 50% of the code base has to be changed 
- 30% of the code base of the native operating system has to be changed
- Less than 2%?

---

The right answer is that the percentage of the guest operating system code that would need to be modified with the para virtualization technology is minuscule less than 2%. 

And this is shown by **proof of construction**. Multiple operating systems were implemented on top of Xen as a para-virtualized hypervisor. 


## 10. Para Virtualization (cont) 

![[L03A_08.png]]

This table is showing you the lines of code that the designers of Xen hypervisor had to change in the native operating systems. The two native operating systems that they implemented on top of Xen hypervisor are Linux and Windows XP. 

And you see that in the case of Linux, for instance, the total amount of the original code base that had to be changed is just about 1.36%. And in the case of XP it is miniscule. Even though in para-virtualization we have to modify the operating system to run on top of the hypervisor, the **amount of code change that has to be done in the operating system** in order to make it run on top of the hypervisor can be **bound to a very small percentage** of the total code base of the original operating system. Which is good news.

## 11. Big Picture 

So what is the big picture with virtualization? In either of the two approaches that I mentioned whether it is full virtualization or para-virtualization we have to **virtualize the hardware resources** and **make them available safely to the operating systems** that are running on top of the hypervisor. Hardware resources refers specifically to the **memory hierarchy**, **the CPU and the devices** that are there in the hardware platform. 

But how do we virtualize them and make them available in a transparent manner for use by the operating systems that live above the hypervisor? And how do we affect data and control transfer between the guest operating systems and the hypervisor? 