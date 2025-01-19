The **A-stack** is a buffer that is allocated **by the kernel** in shared memory, which maps **the address spaces of both the client and the server**. The size of the A-stack is established by the [[Procedure Descriptor (PD)]]. 

Arguments must be passed by **value**, not by **reference**. This is because if the A-stack has pointers to other parts of the unshared client address space, the server will be unable to access that.

It's important to understand that **the kernel is completely unaware** of the relationship between client and server.  Therefore, in order for the server and client to communicate, the server must signal to the kernel that it requires a buffer of a certain size. This shared memory is allocated and then **mapped into the address space of both the client and the server.** 

The purpose of the A-stack is to **facilitate communication between the server and the client**, but **without** any mediation from the kernel. By implementing an A-stack, there is **no longer any need** for the kernel to facilitate communication between the server and the client, which **reduces copying overhead**. 