## [[Xen and the Art of Virtualization.pdf]]

1) Imagine a Linux OS top of Xen (XenoLinux). A user launches an application "foo" on top of XenoLinux. Walk through the steps that happen before foo actually starts running. 
2) Given : 
   ```
   foo starts executing; it executes a blocking system call 
   fd = fopen("bar"); 
   ```
	1) Show all the interactions between XenoLinux and Xen for this call Clearly indicate when foo resumes execution. 
	2) Upon resumption, foo executes another blocking system call: `fwrite(fd, bufsize, buffer);` 
		1) Show all the interactions between XenoLinux and Xen for this call Clearly indicate when foo resumes execution. Don't worry about the exact syntax of the above calls. 
		2) Construct and analyze a similar example for network transmission and reception by foo. 
3) All processes in XenoLinux occupy the virtual address space 0 through VMMAX, where VMMAX is some system defined limit of virtual memory per process. XenoLinux itself is a protection domain on top of Xen and contains all the processes that run on top of it. Given this how does XenoLinux provide protection of processes from one another? [Hint: Work out the details of how the virtual address spaces of the processes are mapped by XenoLinux via Xen.]
4) Give three important motivations for OS virtualization.

## [[Memory Resource Management in VMware ESX Server.pdf]]

1) What is the difference between VMWare workstation and VMware ESX server? Discuss how OS calls from a user process running inside a VM are handled in the two cases. 
2) Discuss the difference in memory virtualization in VMware and Xen. 
3) What are the approaches to reclaiming memory in a virtualized over-subscribed server? Discuss the pros and cons of each approach. 
4) Describe how ballooning may be used with Xen. 
5) Explain how page sharing works in VMWare. 
6) Explain the allocation policy of VMWare.
