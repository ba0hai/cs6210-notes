## [[Extensibility, Safety and Performance in the SPIN Operating System.pdf]]

1) What features of Modula-3 are essential for implementing the extensibility features of SPIN? Why? 


3) Strand is an abstraction provided by SPIN for processor scheduling. Explain what this means. In particular, how will a library OS use the strand abstraction to achieve its desired functionality with respect to processor scheduling? (Hint: Think of the data structures to be associated by the the library OS with a strand.)


1) How can SPIN ensure that a given library OS does not become a CPU hog, shutting out other library OSes? (Hint: Think of how SPIN can control "macro" resource allocation at start up time for a library OS.)
2) Repeat Q3 for physical memory.

## [[Exokernel_ An Operating System Architecture for Application-Level Resource Management On Micro-Kernel Construction.pdf]]

1) A library OS implements a paged virtual memory on top of Exokernel. An application running in this OS encounters a page fault. Walk through the steps from the time the page fault occurs to the resumption of this application. Make any reasonable assumptions to complete this exercise (stating the assumptions). 
2) "The high level goals of SPIN and Exokernel are the same." Explain why this statement is true. 
3) Map the mechanisms in Exokernel to corresponding mechanisms in SPIN. 
4) Explain how the mechanisms in Exokernel help to meet the high level goals. 
5) Repeat Q4 for SPIN.

## [[On Micro-Kernel Construction.pdf]]

1) Sketch how you will implement a device driver (say for a simple device like a keyboard; if you are really adventurous do for a disk) using the primitives provided by Liedtke's microkernel. (Hint: Assume address space A0 magically exists that exposes the physical memory. Assume the device registers are memory mapped at some well known physical addresses in high memory.) 
2) What is the difference between a "thread" as defined in Liedtke's paper and a Strand in SPIN? 
	1) A **thread** $\tau$ is an activity executing inside an address space. It's characterized by: 
		1) registers,
		2) an instruction pointer, 
		3) a stack pointer, 
		4) state information, 
		5) address space ${\sigma}^{\tau}$ in which $\tau$ currently executes. 
	2) By contrast, a **strand** has no minimal or requisite kernel state other than a name. Strands are simply an interface that reflects processor context, and have the following methods in the interface: `block`, `unblock`, `checkpoint`, and `resume`. 
3) Liedtke argues that a microkernel should fully exploit whatever the architecture gives to get a performance-conscious implementation of the microkernel. With this argument as the backdrop, explain how **context switching overhead of a microkernel** may be mitigated in modern architectures. Specifically, discuss the difference in approaches to solving this problem for the PowerPC and Intel Pentium architectures. Clearly explain the architectural difference between the two that warrant the difference in the microkernel implementation approaches. 
4) Explain the notions of "independence" and "integrity" in the L3 microkernel. 
5) All the subsystems in L3 live in user space. Every such subsystem has its own address space. The address space mechanism "grant" allows an address space to give a physical memory page frame to a peer subsystem. Can this compromise the "integrity" of a subsystem? Explain why or why not. 
6) "Trust in the L3 microkernel is transitive." Explain what this sentence means.