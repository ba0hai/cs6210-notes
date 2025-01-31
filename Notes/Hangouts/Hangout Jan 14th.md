
The limitation of a VIPT cache is that you have a **limited number of bits** from the page offset to size the cache. 

![[Pasted image 20250114191012.png]]

Page coloring changes the virtual page number and handles this problem specifically. 

![[Pasted image 20250114191047.png]]

OS guarantees that some bits of VPN will remain unchanged, ensuring a **bigger index**, which once more ensures a **larger cache size**. 

Operating systems do require resources to execute a process, but it should **not** perform at the expense of th euser. 

DOS-like, monolithic, and microkernel structures. 

Monolithic--you cannot change anything about the operating system. 
Microkernel--minimal set of interfaces and services on top of it. This is the functionality expected of an operating system.

Services live above the microkernel. 

There is always tension between generality and specailization. The other extreme is the DOS-like structure. 

DOS-operating systems there is a separation between OS and application, but the separation is imaginary. 

![[Pasted image 20250114191509.png]]

Why do you need a process abstraction? If you have a system that's going to be **shared**, this is only needed if the processor is being multiplexed. On DOS, you're running a single app at a time. No reason for having the process abstraction in the first place. 

![[Pasted image 20250114191743.png]]

Compare with the **monolithic structure**. 

![[Pasted image 20250114191801.png]]

And compare with **microkernel** structure. 

Simple abstractions. All services use these abstractions to provide the functionality you would expect for the application..

No hard line between application and operating system services, because in a microkernel system, all of these OS services are an application. But OS services are implemented by processes that have higher privileges than a normal application process. 

Quiz - Which came first? Unix or MS DOS? 
- MS DOS was invented for the PC, not a general purpose operating system. Unix came out in 1974, but MS DOS came out 1982. At the time that the personal computer was invented, the intent for DOs was a single-user, single-application system. So MS DOS was going for simplicity. 


Quiz - What is extensibility of an OS? 
- Customizing system services on the fly without taking down the system? 
- Doing software updates to the OS as it happens today with windows/mac/iPhone 

It is yes: customizing system services. 

![[Pasted image 20250114192207.png]]

What are the customization opportunities in the steps above: 
1. None
2. Update page table for faulting proces
3. **Page replacement algorithm**
4. Resuming a process

What is the difference between protection domains and address spaces?
1. No difference
2. Protection domain is an architectural feature
3. Address space is an OS concept
4. **Protection domain is an OS abstraction while address space is a hardware mechanism for implementing it** 

Protection domains are a logical concept. Address space is provided by the hardware, it's an implementation mechanism for the concept of a protection domain. 

All of the above are the preliminary knowledge required for L02. 

Border crossing as used in lecture i: 
- Exactly the same as a process context switch
- Going from one hardware address space to another
- Going from one logical protection domain to another 

Border crossing is going from one hardware address space to another, **not** going from one logical protection domain to another.

OS is written in Modula 3 so each logical protection domain 

Can an entire OS be written in a type-safe high level language? 
- No

Subsystens on top of SPIN (eg: CPU scheduler)
- Fully adhere to modula-3 type safety
- May have to 

![[Pasted image 20250114193111.png]]
