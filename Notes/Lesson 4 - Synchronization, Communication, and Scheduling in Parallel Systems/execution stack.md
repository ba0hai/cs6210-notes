This is the stack that the **server procedure will use** to execute its own procedure calls. The A-stack, by contrast, serves the purpose of sharing arguments, but the E-stack is a data structure used by the server in order to execute its own procedures. 

The server stub **copies arguments from the A-stack** into the execution stack. Only after copying arguments into the E-stack is the server stub ready to execute a procedure. 

After procedure execution, the server stub **will copy the results from the E-stack into the A-stack**. 