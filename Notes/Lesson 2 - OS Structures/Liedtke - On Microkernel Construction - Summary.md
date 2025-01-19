## Outline 
1. Abstract
2. Rationale 
3. Some micro-kernel concepts
	1. Address Spaces
		1. Reasoning
		2. I/O
	2. Threads and IPC 
		1. Supervising IPC 
		2. Interrupts
	3. Unique Identifiers 
4. Flexibility 
	1. Memory Manager
	2. Pager
	3. Multimedia Resource Allocation 
	4. Device Driver
	5. Second Level Cache and TLB 
	6. Remote Communication 
	7. Unix Server 
	8. Conclusion 
5. Performance, Facts, and Rumors 
	1. Switching Overhead
		1. Kernel-User Switches
			1. Conclusion 
		2. Address Space Switches 
			1. Conclusion 
		3. Thread Switches and IPC 
	2. Memory Effects 
6. Non-Portability 
	1. Compatible Processors
		1. User-address-space implementation 
		2. IPC implementation 
	2. Incompatible Processors
7. Synthesis, Spin, DP-Mach, Panda, Cache and Exokernel 
	1. Synthesis
	2. Spin 
	3. Utah-Mach
	4. DP-Mach
	5. Panda
	6. Cache-Kernel
	7. Exokernel
8. Conclusions
	1. Availability
9. Appendix - Address Spaces
	1. An Abstract Model of Address Spaces 
	2. Implementing the Model 

The micro-kernel is an inherently non-portable operating system architecture. To ensure efficient performance, Liedtke argues that the micro-kernel should be implemented using processor-independent abstractions, but each implementation is tailored per-processor to exploit a wide range of hardware.
## Micro-Kernel Concepts 
 
 Liedtke's microkernel defines the minimum number of "primitives" that should be implemented. The defining criteria for what is considered a micro-kernel primitive is whether or not they fulfill the principle of independence and the principle of integrity. 

#### Principles of Independence and Integrity 
The principle of independence is defined as: 
- An arbitrary subsystem $S$ can be implemented without being corrupted or disturbed by other subsystems $S'$. 

The principle of integrity is defined as: 
- An arbitrary subsystem $S_1$ can establish a communication channel with arbitrary subsystem $S_2$ that will not be corrupted or interfered with by other subsystems $S'$. 

In other words, microkernel primitives should ensure that any subsystem implemented on top of the micro-kernel will be able to adhere to the principles of independence and integrity. Additionally, microkernel primitives themselves should embody these principles. 

## Flexibility 

## Performance, Facts, and Rumors 

## Non-Portability 

## Comparison with SPIN, Mach, Exokernel 

## Conclusion 

