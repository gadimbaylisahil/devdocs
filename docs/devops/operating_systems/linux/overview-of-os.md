# Overview of OS

## Permissions

Files:

r - read
w - write
x - execute

Directories:

r - can list files
w - can create files/dirs
x - can cd into and access files

Sticky bit is a way to mark the file so that only the owner, the directory's owner or the root user can overwrite, delete or rename a file. Other users can't delete, rename or overwrite the folder even when having full permissions on read/write.

```sh
# To set sticky bit
chmod +x /path/to/folder

chmod o+x /path/to/folder
```

File permissions are 12 bits

'xxx' 'xxx' '***' '***'

## /proc

Every process on Linux has PID

If you access /proc/pid, you will find a lot of useful information about the process.

### /proc/PID/stack

Kernel's current stack for the process

### /proc/PID/environ

All of the process' environment variables during the runtime

### /proc/PID/cmdline

Command line arguments passed in when running the process

`man proc` for more information

## System Calls

Programs use system calls to communicate with the Kernel.

Every system call can be identified as number.

You can see system calls a program using with `strace command`

## File Descriptors

Unix use integers to track open files which is called file descriptors.

`lsof` shows you a process' open files

`lsof -p PID`

It can refer to files on disk, pipes, sockets, terminals, devises etc, as almost everything in UNIX is a file.

Almost every process has 3 file descriptors:

STDIN 0
STDOUT 1
STDERR 2

## Pipes

You can name pipes. And a different process can read through that pipe

```
f = open('./custom-pipe')
f.write("something")

f = open('./custom-pipe')
f.readline() # read the pipe above
```

## Unix Domain Sockets

UDS lets two programs on the same computer communicate by writing to the socket file.

2 kinds of domain sockets:

- stream: like TCP. Lets you send continuous stream of bytes
- datagram: like UDP. Send discrete chunks of data

Docker uses domain sockets.

## Memory Allocation

Memory allocators are responsible for:

- Keeping track of what memory is used/free
- Ask the OS for more memory

Malloc is just a function. Alternatives: jemalloc, tcmalloc.

Malloc tries to fill in unused space when you ask for more memory.

## Virtual Memory

Computers have physical memory, however, when a program references an address like `0x5c69a2a2` it's a `virtual address`.

Linux keeps a mapping from virtual memory pages to physical memory.

When a program tries to access a memory using a virtual address, MMU(Memory Management Unit) tries to find it's location in physical part.

## Shared libraries

Use `ldd programname` to get list of shared libraries for the given program.

## Copy On Write

On linux we start a new process using fork(), or clone().

However, using so, a new identical process will be created with the same memory, same heap and same stack. This would be wasteful.

Thus exec() is called right after fork() which means they don't use parent process' memory at all.

Linux handles when the new process tries to write the memory and allocates as necessary.

## Page Faults

Every linux process has `page table` - which is a table for mapping of virtual memory to physical memory.

Some pages are marked as read only or not resident in memory. When a program tries to access a page as such, it results in `page fault`. Linux kernel handles the page fault runs. However, it is slow.

These can happen if memory is moved to swap(disk) and later program tries to access it.