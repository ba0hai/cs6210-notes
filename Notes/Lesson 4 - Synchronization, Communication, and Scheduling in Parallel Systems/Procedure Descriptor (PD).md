Procedure descriptors are a data structure that is **created and primarily accessed by the kernel**. 
Given some entry-point procedure `foo`, which exists on some server, a **unique procedure descriptor** is instantiated for the procedure. 

Based on the answers to the following questions: 
- **Where** is the entry point in the server's domain for this particular procedure? 
- What is the **size** of the communication buffer required to be established by the kernel for communication between the client and the server? 
- **How many** simultaneous calls is the server is willing to accept for this procedure? 

it instantiates the following fields: 
 1. entry point address,
 2. the [[arguments stack]] (A-Stack) size, 
 3. and the allowed number of simultaneous calls for this specific procedure 
	 1. The purpose behind this, on a multi-processor, and there are multiple cores and processors available, then is it may be possible for the server $S$ to farm out multiple threads to execute **simultaneous calls** that are coming in from multiple clients distributed in the system.

