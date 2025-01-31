
## Outline 

1. Introduction 
2. Xen: Approach and Overview 
	1. The Virtual Machine Interface
		1. Memory management
		2. CPU
		3. Device I/O
	2. The Cost of Porting an OS to Xen 
	3. Control and Management
3. Detailed Design 
	1. Control Transfer - Hypercalls and Events
	2. Data Transfer - I/O Rings
	3. Subsystem Virtualization 
		1. CPU Scheduling
		2. Time and timers
		3. Virtual address translation 
		4. Physical Memory
		5. Network
		6. Disk
	4. Building a New Domain
4. Evaluation
	1. Relative Performance
	2. Operating System Benchmarks
		1. Network Performance
	3. Concurrent Virtual Machines
	4. Performance Isolationi 
	5. Scalability
5. Related Work
6. Discussion and Conclusion
	1. Future Work
	2. Conclusion 
7. References

## Vocabulary 
universal buffer cache 
last-chance page cache (LPC) 

