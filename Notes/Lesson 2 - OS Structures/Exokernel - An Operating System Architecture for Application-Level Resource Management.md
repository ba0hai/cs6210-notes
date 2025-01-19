1. Introduction
2. Motivation for Exokernels
	1. The Cost of Fixed High-Level Abstractions 
	2. Exokernels - An End-to-End Argument
	3. Library operating Systems 
3. Exokernel Design
	1. Design Principles
		1. Securely Expose Hardware
		2. Expose Allocation
		3. Expose Names
		4. Expose Revocation
		5. Policy
	2. Secure Bindings
			1. Multiplexing Physical Memory
			2. Multiplexing the Network
		1. Downloading Code
	3. Visible Resource Revocation
		1. Revocation and Physical Naming
	4. The Abort Protocol
4. Status and Experimental Methodology
5. Aegis - An Exokernel
	1. Aegis overview
		1. Processor Time Slices
		2. Processor Environments
			1. Exception context
			2. Interrupt context
			3. Protected Entry context 
			4. Addressing context
	2. Base Costs
	3. Exceptions
	4. Address Translations
	5. Protected Control Transfers
	6. Dynamic Packet Filter (DPF) 
6. ExOS - A Library Operating System
	1. IPC Abstractions
		1. pipe
		2. shm
		3. lrpc
	2. Application level Virtual Memory
	3. Application-Specific Safe Handlers
7. Extensibility with ExOS
	1. Extensible RPC
	2. Extensible Page-table Structures
	3. Extensible Schedulers
8. Related Work
9. Conclusion
