
## 1. RPC and Client Server Systems 
![[Pasted image 20250109111851.png]]

The client-server paradigm is used in structuring system services in a distributed system. If we're using a file server in a local area network every day we are using a client-server system when we are accessing a remote file server. A **remote procedure call** is the mechanism that is used in building this kind of a client server relationship in a distributed system. 

**What if the client and the server are on the same machine?** Would it also not be a good way to structure the relationship between client and servers using RPC. Even if the clients and the servers happen to be on the same machine. It seems logical to structure clients of a systems even on the same machine using this RPC paradigm. But the main concern is performance. And the relationship between performance and safety. 

Now for reasons of safety which we have talked a lot about when we talked about operating system structures you want to make sure that servers. And clients are in different address spaces or different protection domains as you've been calling them. Even if they are on the same machine uhthey will be running on different processors of an SMP but they're still on the same machine. You what you want to do is you want to give a separate protection domain for each one of these servers from the point of view of safety. But what that also means because we are providing safety there's going to be a hit on performance. Because of the fact that an RPC has to go across the outer spaces. A client on a particular outer space server on a different outer space. So that is going to be a performance penalty that you pay. 

Now as operating system designers what we would like to be able to do is to **make RPC calls across protection domains** as **efficient** as a **normal procedure call** that is happening inside a given process. 

If you could make the RPC across protection domains as **efficient** as a a normal procedure call it would actually Encourage system designers to use RPC as a vehicle for structuring services even within a same machine. Why is that a good idea? Well what that means is that you know we've talked about the fact that in structuring operating systems in microkernel you want. You want to be able to provide every service having its own protection domain. What that means is that to go across these protection domains you're making a a protected procedure call or a RPC call. Going from one from one protection domain to another protection domain. And that is going to be more expensive than. Simple procedure call. It won't encourage system designers to use these separate protection domains to provide the services independently. So in some sense again is the same question of wanting to have the cake and eat it too. So you want the protection and you also want the performance.

## 2. RPC vs Simple Procedure Call 

![[Pasted image 20250114182038.png]]

All of you know how a simple procedure call works. There is a caller, and there is a process in which **all the functions** are being compiled together and linked together to make an executable. 

When a caller makes a call to the callee, it makes a call passing the arguments on the stack. After which, the callee can execute the procedure, and then return to the caller. **All of these steps happen at compile time.** 

What happens during a **remote procedure call?**

The principles are similar–there is once more a **caller** and **callee**, wher ethe **caller** is making a call to **execute a procedure** before returning.

![[Pasted image 20250114182419.png]]

However, every call is actually a **trap** into the kernel. 

![[Pasted image 20250114182450.png]]

During the call trap, the kernel **validates the call**, and it copies the arguments of the call into **kernel buffers** from the client address space. 

The kernel then **locates the server procedure that needs to be executed** and **copies the arguments that it has buffered** in the kernel buffer into the address space of the server. Then, the kernel schedules the server to run the particular procedure. 

![[Pasted image 20250114183417.png]]

Here, the server procedure starts executing using the arguments of the call and performing the function requested by the client. When the server procedure is done with execution of the procedure, it needs to return the **results** of this procedure execution back to the client. The server will perform a **return trap** to return the results back to the client. Then, the kernel will copy the results from the address space of the server into the kernel buffers, before copying out the results from the kernel buffer into the client's address space. 

At this point, the results have been sent back to the client, meaning that the kernel can then reschedule the client to receive the results, before returning to execute its previous process. 

Note that, despite the simple appearance of the RPC, every step in the RPC involves a degree of complexity. Additionally, **all of these steps are happening at runtime**. The steps behind a procedure call take place at compile time by comparison. **Because these steps take place at run-time, the RPC steps incur a degree of runtime overhead**. 

There are two traps: a call trap, followed by a return trap. This means that there are also two context switches: one where the kernel switches from the client, to the server, to the procedure that runs the server. Additionally, once the server procedure is finished with execution, the procedure has to reschedule the client to run again. 

In total, there are: 
- two traps,
- two context switches, 
- and one procedure execution
that must be performed by the runtime system in order to execute a remote procedure call. 

 So what are the sources of overhead?
 - During this call trap, the kernel has to **validate** whether this client is allowed to make this procedure call or not the validation has to happen. 
 - And then it has to copy the arguments from the client's address space into kernel buffers. There are potentially **multiple copies** that will be created in order to do this exchange between the client and the server.
 - Then there is the **scheduling of the server** in order to run the server code,
 - As well as the **context switch overhead** that is incurred on every context switch,
 - and the act of **dispatching a thread on the processor** itself is also time which is explicit cost of scheduling. 

## 3. Quiz - Kernel Copies 

In  an RPC, there is a client call, which is followed by the server procedure execution and then the returning the results to the client. **How many times does the kernel copy stuff from the user address spaces into the kernel and vice versa?** In other words, throughout the entire interaction between client and server during an RPC–going from the client call to server execution and returning results back to the client the whole package in order to execute an RPC–**how many times does the kernel copy information from user address spaces into the kernel buffers and vice versa?** 

- Once
- Twice
- Thrice
- Four times

---

The correct answer is **four** times. 
1: The kernel has to copy **from the client address space into the kernel buffer**. 
2: The kernel has to copy **from the kernel buffer into the server**. 
3: When the procedure is completed, the kernel has to copy results from the server address using the kernel. 
4: **The kernel has to copy from the kernel buffer into the client.** 

## 4. Copying Overhead 

The copying overhead is incurred **every time** a call is returned between client and server. So **how can we avoid repeated copying** during RPC? 

And if you go back to this analogy of a procedure call the nice thing about this is that this **the arguments are set up in the stack.** The kernel **is not involved** in the data movement.

![[Pasted image 20250115175236.png]]

Let's analyze how many times copying happens in the RPC system. Recall that in a RPC system the kernel has no idea of the syntax and semantics of the arguments that are passed between the client and the server. But yet the kernel has to be the intermediary in arranging the rendezvous between the client and the server. And therefore what happens in the RPC system is that when a client makes a call there's an entity that is called the client stub. And what the client stub is going to do is the client's thinking that it's making a normal procedure call but it is a remote procedure call. And the client stub knows that. And what it does is it takes the arguments that is in the client call which is living on the stack of the client and makes an RPC packet out of it. This RPC packet is essentially serializing the data structures that are being passed as arguments by the client into a sequence of bytes. It's sort of like herding cats into an enclosed space. So that's what is happening by the client stack taking the arguments that are on the stack of the client and creating a packet of contiguous bytes which is the RPC message. Because that is the only way the client can actually communicate this information to the kernel. So this is the first copy that's happening from the client stack into creating the RPC message is the first copy that's happening. Even before the kernel is involved in this client server interchange. The next thing that happens the client traps into the kernel and the kernel says well you know there is a message which is the RPC message that has to be communicated to the server. And that's sitting in the user address space. I better copy it into my kernel buffer so that's a second copy that's happening. From the address piece of the client is the RPC message is copied into the kernel buffer. So that's the second copy. Next the kernel schedules the server in the server domain because the server has to execute this procedure. So once that server has been scheduled the kernel copies the buffer. It it it has all the arguments packaged in into the server domain. So that it the third copy that's happening. So this so we went from the client stack to the RPC message first copy. From the RPC message to the kernel buffer second copy. And now the kernel buffer is passed out to the service domain that's a third copy. But unfortunately even though we've reached the address space of the server the server procedure cannot access this because from the point of view of the procedure call semantics the client of the server think that they are just doing procedure call. So the server procedure is expecting all of the arguments in the original form on the stack of the server and that's where the server stub comes in. So what the server stub is just like the client stub the server stub is a piece of code that is part of the RPC infrastructure that understands the syntax and semantics of the client server communication for this particular RPC call. And therefore it can take this information that has now been passed into the server's address space by the kernel and structure it into the set of actual parameters that the procedure the server procedure is expecting. So this from the server domain wherever the kernel put it into the stack of the server for the server procedure to execute that procedure that's the fourth copy. So you can see that just going from the client to the server there are four copies involved. These two copies are at the user level. And these two copies are what the kernel is doing in order to protect itself and the address spaces from one another by buffering the address space contents into a kernel buffer and passing that to the server domain before the server domain can start using it in the form of actual parameters on the stack. So at this point the server can start executing the server procedure can start executing do its job. And when it is done it has to do exactly the same thing in order to pass the results back to the client. So it is going to go through four copies except that we're going to reverse it. We're going to start from the server stack and go all the way down to getting the information to the client stack in order for that exchange to happen. So in other words with the client server RPC call on the same machine with the kernel involvement in this process there's going to be four copies each way. Going from the client to the server there's four copies. Going from the server back to the client there's going to be four copies. Two copies are happening in the user space and two copies are happening in the kernel space. Are orchestrated by the kernel and two copies orchestrated on the user level. Now as you can see this is a huge huge overhead compared to a simple procedure call that I showed you early on.

## 5. Making RPC Cheap 

In order to make RPC cheap, we need to **optimize for the common case**, which is **the actual calls being made from the client to the server.** Make sure that during the actual calls, the locality of the arguments and cache contents are accessible to both client and server. 

Setting up the relationship (binding) between client and server needs to be done **once** per server-client setup, so it is considered a one-time cost, not a recurring cost. 

### How does binding work? 
![[Pasted image 20250115181538.png]]

1. Given an entry point procedure called `foo` that exists on the server, the server registers the entry point procedure in a **name server**.
	2. The server **notifies the kernel** of the newly added entry point procedure. 
	3. Now, the server **waits for bind requests** to come from the **kernel**. 
2. The client calls the entry point procedure `foo` of the server, generating a trap in the kernel. 
3. (Validation) The kernel **must check** with the server if this client can make calls to the server's entry point. 
	1. Kernel makes an up-call to the server, which includes details about the client and its desired entry point procedure on the server.
	2. The server **grants permission via the kernel** to the client to call the entry point procedure. 
4. The kernel sets up a **[[Procedure Descriptor (PD)]]** for each procedure provided by servers, with the:
	1. entry point address,
	2. the [[arguments stack]] (A-Stack) size, 
	3. and the allowed number of simultaneous calls for this specific procedure 
		1. The purpose behind this, on a multi-processor, and there are multiple cores and processors available, then is it may be possible for the server $S$ to farm out multiple threads to execute **simultaneous calls** that are coming in from multiple clients distributed in the system.
5. The kernel **allocates a buffer** shared between the client and the server with the size of the A-Stack (specified by the server). 
6. The client and server can exchange data using this buffer **without** any intervention from the kernel.
7. The kernel provides authentication to the client in the form of a Binding Object (BO). The client can use this BO whenever it needs to make a call to this specific server. 

## 6. Making RPC Cheap (Binding) 

![[Pasted image 20250115184451.png]]

Binding begins with the **instantiation of a Procedure Descriptor**, and concludes with the instantiation of an **A-stack** and **Binding Object**, which is detailed in the final 3 steps below. 

5. The kernel **allocates a buffer** shared between the client and the server with the size of the [[arguments stack]] (specified by the server). 
6. The client and server can exchange data using this buffer **without** any intervention from the kernel.
7. The kernel provides authentication to the client in the form of a [[Binding Object]] (BO). The client can use this BO whenever it needs to make a call to this specific server. 

This is the **kernel mediation** that takes place at **entry point setup**, which occurs on the **first call of the client**. One important takeaway of this process is that **the kernel knows that the BO and PD are related.** That is, **the kernel is able to determine the corresponding PD because it is information embedded in the BO.** 

## 7. Making RPC Cheap (Actual Calls)

![[Pasted image 20250115190330.png]]
#### Making the Actual Call: 
1. Passing arguments between the client and the server through the A-Stack can only be by value, not by reference, since the client and server don't have access to each other's address space. 
2. The stub procedure copies the arguments from the client memory space into the A-Stack. 
3. The client traps into the kernel, and the client stub presents the BO to the kernel (trap). 
	1. Blocks on the kernel's response. Kernel checks whether the PD associated with the BO corresponds to the particular RPC being made by the client. 
4. At this point, the client will be blocked waiting for the call to be executed. The kernel can use the client thread for executing the procedure on the server's domain. 
	1. An optimization that the kernel can do is **borrow the client thread** and **[[doctor the client thread]] to run on the server address space.**
5. The kernel validates the BO and allocates an [[execution stack]] (E-stack) for the server to use it for its own execution. 
6. The server stub **copies the arguments** from the A-Stack to the E-Stack. 
7. After finishing execution, the server stub will copy the results from the E-Stack to the A-Stack. 
8. The server then **traps into the kernel** (called a return trap), which will allow the kernel to return control back to the client. There is **no need for the kernel to authenticate the server's return trap**. It will then [[doctor the client thread]] again to execute in the **client address space**, since it knows the **return address** to start the thread from, as well as the **client's address space**. 
9. Everything will be done reversely to return to the client. That is, the client stub will copy over the results of the procedure from the A-stack into the client's private stack, and the client thread resumes normal execution.
## 8. Making RPC Cheap (Actual Calls) (cont)

### Topic

![[Pasted image 20250119183024.png]]

At this point, once the kernel has doctored this client thread to start executing the server procedure, it can transfer control to the server. So it transfers the control to the server and so now now we're starting to execute the server procedure in the server's address space. And in the server's address space, because A-stack has been mapped in this is also available to the server domain. 

![[Pasted image 20250119183139.png]]
And the first thing that's going to happen in the server domain is: our server stub is going to get into action and take the arguments that are sitting in the A-stack and copy them into the stack that the server procedure's going to use. 

Recall that the kernel provides a special stack for the purpose–an E- stack, or execution stack–and that is a stack into which the client the server stub is going to copy the A-stack arguments into that E-stack and then at that point the procedure `foo` is ready to start executing.

At this point procedure foo is like any normal procedure. `foo` finds the information it wants on the stack it does its job. Once it is done with executing this procedure it has to pass back the results to the client. 

![[Pasted image 20250119183355.png]]

What is going to happen is that the server stub is going to take the results of this procedure execution and copy them into the A-stack. And of course all of this action is happening in the server domain, without any mediation by the kernel. 

Once the server stub has copied the results into the A-stack, it can trap into the kernel and this is the vehicle by which the kernel can transfer control back to the client so it it does a return trap. 

![[Pasted image 20250119183436.png]]

Now when this return trap happens there is no need for the kernel to validate this trap as opposed to the call trap because the up call was made by the kernel in the very first place. Therefore, the kernel is expecting this return trap to happen and so the kernel doesn't have to do any special validation for this. 

And at this point what the kernel is going to do is it is basically going to re-doctor the thread to start executing the client address space. So basically it knows the return address where it has to go back in order to start executing the client code, and it knows the client's address space so it's going to re-doctor the thread to start executing in the client address space. 

Finally, when the client thread is rescheduled to execute, at that point the client stub gets back into action and copies the results that are sitting in the A-stack into the stack of the client and once it has done that the client thread can continue on with its normal execution. 

So that's what is going on. And the important point that you notice is that **the copying through the kernel that used to happen is now completely eliminated** because your arguments are passed through the A-stack into the server. And similarly, the result is passed through the A-stack into the client. So let's analyze what we've accomplished in terms of reducing the cost of the RPC in the actual calls that are being made between the client and the server.

## 9. Making RPC Cheap (Actual Calls) (cont)
![[Pasted image 20250115190428.png]]

#### Results of using an A-Stack: 
1. Using an A-Stack reduces the number of copies from four to two. 
2. The two copies happen in the client or server userspace above the kernel. 

The copying that happens through the kernel is now **completely eliminated**. 
#### Even with this trick, we still have the overhead associated with the context switch itself: 

1. The client trap. 
2. Switching the protection domain. 
3. The server trap. 
4. Loss of locality (implicit). 


Recall that we had four copies in doing the client call just transferring the arguments from the client to the server's domain. That was the original cost. 
The four copies were:
1. first creating an RPC packet,
2. copying that RPC packet into the kernel buffer,
3. Copying the kernel buffer out into the server domain,
4. And in the server domain the server stub copied this information that had been passed up to it by the kernel and putting it on the server stack to start executing the server code. 
So this was the original costs that we incurred in terms of copying. 

Now life is much simpler. All that is happening is on the client side, the client's stub is copying the parameters into the A-stack. Copying the parameters is very different from what was happening over here. Here the client stub was doing a lot more work. It actually had to **serialize the data structures** that are being passed as as actually arguments **into a sequence of bytes in this RPC message**. Whereas here it is simply copying it because **the client and the server know exactly what the semantics and syntax of the arguments that are being passed back and forth** and therefore there is **no need to serialize the data structure**. It just has to create a copy of the parameters into the A-stack. 

This A-stack is of course shared between the client and the server. What the server stub is going to do is basically going to do is **take the arguments that are now sitting in the A-stack and copy it into the E- stack**. Remember the execution stack provided by the kernel for executing the server procedure? That is a the special server stack that we're going to use. 

So the arguments are copied by the server stub into the E-stack and once it is done the server procedure is now ready to be executed in the server domain. 

So what we accomplished is that the entire client server interaction requires only two copies:
1. One for copying the arguments from the client stack into the A-stack which is usually called the **marshalling** of the arguments. 
2. And the second copy is taking the A-stack arguments that are sitting in the A-stack and copying it into the server's stack that is the **unmarshalling**. 

So these are the two copies involved. One on the client side and one on the server side **and both these copies are happening above the kernel**. It is in the user space, or the space of the client, that the client stub is making this copy of the arguments into the A-stack. And similarly it is in the space of the server domain that the unmarshaling is happening. **We have basically taking the original four copies and gotten rid of the two copies that were being done inside the kernel**. One into the kernel and one out of the kernel. These two copies which is done by the kernel we got rid of them. And instead we have only two copies, which is a more efficient way of creating the information that needs to be passed back and forth between the client and the server using this A-stack. Needless to say the same thing is going to happen in the reverse direction for returning the results. **So it is the server stack that is going to have the result** and **the server stub is going to put it in the A-stack** and the client stub is going to take it from the A-stack and give it to the client so that the client can state resuming its execution. So there's two copies involved in going from the client to the server and two copies involved in going back to the client from the server.

## 10. Making RPC Cheap Summary 

So to summarize what goes on in the new way we are doing the RPC between the client and the server. During the actual call copies through the kernel is completely eliminated because all of the argument-result passing between the client and the serving is happening through this A-stack which is mapped into the outer space of the client and the server. 

And so the actual overheads that are incurred in making this RPC call is:
- this client trap and validation by the kernel that this call can be allowed to go through,
- and switching the domains, that trick of doctoring the client thread to start executing in the server procedure. **That is really switching the protection domain from the client address space into the server address space** so that you can start executing the procedure that's visible only in this address space. So that is the switching domain in the second overhead. 
- And finally the return trap at the end of th server procedure. That's the third explicit cost. 

So three explicit costs associated with the actual call. To further summarize: the first explicit is the client trap and and validating this BO. And the second explicit cost is switching the protection domain from the client to the server so that you can start executing the server procedure. And the third explicit cost is when we have this return trap to go back to the client address space. So those are the explicit costs. 

But we know that there are **implicit overheads** that are associated with switching protection domains. The implicit overhead is **the loss of locality due to the domain switching**. When we go from the client address space to the server address space we we are touching some part of the address space are going to be in physical memory and therefore in the caches of the processor, but there's a lot of stuff that may not be in the caches of the processor. So there is going to be a loss of locality due to the domain switch that that may happen, specifically the caches and the processor may not have all the stuff that the server needs in order to do its execution. **Multiprocessing can facilitate 

## 11. RPC on SMP 

This is where multiprocessor comes in. If you're implementing this RPC package on a shared memory multiprocessor then we can exploit multiple processors that are are available in the SMP. 

What we can do is preload the server domains in a particular processor and don't let anything else run on this processor (IE: This particular server is loaded on CPU 2. We're not going to let any other thing disturb what's going on in this CPU), the caches associated with this CPU will be warm with the stuff that this particular domain needs. In other words the server's address space is **pre-loaded in a particular processor**. If you have multiple processors then you can exploit the fact that you have multiple processors in the SMP. 

If a client comes along and wants to make an RPC call, then what we want to do is **use the server that has been preloaded in a particular CPU** as a **recipient** of this particular RPC call, such that when this client makes that call, that call is going to be directed to the server that has been preloaded in a particular CPU where the caches will be warm, allowing us to reduce mitigate the impact on loss of locality incurred when you go from one protection domain to another protection domain. 

This is assuming that kernel intervention has been eliminated in making the actual call and return between the client and the server by providing an argument stack in shared memory that is shared in the address space of the client. And the address space of the server. And this way the client can pass the actual arguments of the call to the A-stack and the server can retrieve it from the A-stack without kernel intervention. And when the server is ready to return the results back to the client once again it can do the same thing–put it in the A-stack so that it is available for the client. So without any kernel intervention you can actually do the call and return and of course the mediation happens **only in the fact that the kernel has to validate the call**. Every time the client makes a call it has to validate that call. B**ut the loss of locality** you can avoid by making sure that **the server domain is pre-loaded in one of the CPUs**. 

And the other thing that the kernel can do is **look at the traffic of a particular server**. If a server is serving lots of different clients than in a multiprocessor, then it can potentially, based on monitoring the site, that we may want to have multiple CPUs dedicated to the servers such that several different domains of the same server can be preloaded in several CPUs to cater to the needs of several simultaneous requests that may be coming in for a particular service.

## 12. RPC on SMP Summary 

So in summary what we have done is we have taken a mechanism that is typically used in distributed systems namely RPC and we ask the question suppose we want to use RPC as a structuring mechanism in a multiprocessor. How to make that efficient so that the designers of services will in fact use RPC as a vehicle. For building these services. And the reason why you want to promote that is because when you put every service in its own protection domain you are building safety into the system. And that is very important for the integrity of an operating system. As an operating system designer, we worry about the integrity of services and we can provide the integrity by putting every service in its own protection domain. And we're making RPC cheap enough that you would use as a structuring mechanism we are promoting  a software engineering practice of building services in separate protection domains.