# How linux works?

## Overview

Linux system has 3 main levels. Hardware, Kernel and Processes.

- Kernel: core of the operating system. Resides in memory and tells CPU what to do. It manages hardware and acts as intermediary between running processes and hardware.

- Processes: running programs that kernel manages, resides in top level, also called as user space.

There are differences between kernel processes and user processes. Prior runs in kernel mode with unrestricted access to memory in kernel space, while latter runs in user mode with restrictions in user space. User processes has limited access to memory and CPU, in which during crash can be cleaned up by kernel. Thus, a crash in a user process will not effect other processes. User processes may also have privileges if given and can cause more 'damage'.

### Main Memory

Memory in its raw form, is just a big storage of 0s and 1s. This is where running kernel and user processes reside. CPU is an operator on memory, it performs instructions to read and write into memory.
State is just a collection of 0s and 1s.

### Kernel

Nearly everything the kernel does resolves around main memory. It allocates part of memory for each process and ensures that they keep to their share.

- Processes: Kernel is responsible for determining which processes has access to CPU.

- Memory: Keeps track of allocated memory, what might be shared between processes and what is free.

- Device Drivers: Kernel acts as interface between processes and hardware, its job is to operate hardware.

- System calls and support: Processes usually communicate using system calls with kernel.

### Process Management

Process management refers to start, pause, termination and resuming of processes. CPU is shared between processes where each takes its turn to use the allocated resource. This is also referred as `context switching`.

Each piece of time taken by process is referred as `time slice`. Kernel is responsible for managing context switch.

1. CPU interrupts current process based on internal timer, and switches into kernel mode where it hands control.

2. Kernel records current state of memory and CPU, which will be essentialfor resuming current process.

3. Kernel performs any IO operations.

4. Kernel is now ready to let another process run, analyzing current list of processes and chooses one.

5. Kernel prepares memory for the new process and then prepares CPU.

6. Kernel tells the CPU how long is the time slice for this process.

7. Kernel switches CPU into user mode and hands control to the process.

For multi core CPU systems, things get a little more complicated. In this, case Kernel does not need to relinquish CPU for process to run on another CPU.

However, it does so anyway to increase the usage of all CPUs(also using some tricks to have more CPU time for itself as well).

### Memory Management

Kernel must manage memory during the context switch, and it gets complicated due to:

- It must have its own private memory that no user process can access.

- Each process needs its own allocation of memory

- One user process may not access others private memory

- User processes may share memory

- Some memory in user process may be read only

System can use more memory than available -- utilizing disk space as auxiliary.

For help to kernel, modern CPUs come with a memory management unit MMU, that enables processes access virtual memory. When using a virtual memory, a process does not directly access memory in its physical location. Instead, kernel sets up so that process may think it has all memory to itself. When processes tries to access memory, MMU intercepts the requests and maps the address in virtual memory to its hardware location.

### Device Drivers and Management

Device is typically only accessed in kernel mode because improper access by user space process could crash the machine. Another problem is that different devices may have different interfaces, thus its the reason why device drivers are usually been part of the kernel, to strive for uniformed interface to user processes to ease software developers job.

### System calls and support

There are several other kinds of kernel features that are available for user processes. For instance, system calls user to perform specific tasks that user processes may not be able to perform.

Such as opening, reading writing files all involve system calls.

Two system calls, fork() and exec() are important to understand how processes start up:

- fork(): when fork is called, kernel creates nearly identical copy of the process

- exec(): when a process calls exec(program), kernel starts up the program replacing current process

Other than init() all processes in Linux system starts as a result of fork().

### Users

Kernel does not differentiate users by usernames but with their ids. Root user is still run in user space.
