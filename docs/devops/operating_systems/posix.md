# POSIX

[**Reference**](https://www.baeldung.com/linux/posix)

## What is it?

Family of standards by [IEEE](https://www.ieee.org/), enabling compatibility of software between different operating systems.

## History

Porting software between operating systems used to be a lot of work until POSIX was born. POSIX is a standardization of a UNIX like operating systems, but not limited to UNIX.

## Standards

### C-API

POSIX adds more functions on top of ANSI C standars, around:

- File operations
- Processes, threads, shared memory and scheduling parameters
- Networking
- Memory management
- Regular Expressions

### General concepts

General rules over how to write a software.

### File Formats

Rules for formatting strings that we use in files, STDOUT, STDIN, STDERR.  

[Escape Sequence Characters](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap05.html#tagtcjh_2)  

[Conversion Specification](https://www.gnu.org/software/libc/manual/html_node/Table-of-Input-Conversions.html)  

### Environment Variables

[Reserved Environment Variables in standard utilities](https://www.linuxtopia.org/online_books/introduction_to_linux/linux_Reserved_variables.html)

### Locale

[C Locale is POSIX locale][https://en.cppreference.com/w/c/locale]

### Other

- Character Sets
- Regular Expressions
- Directory Structure
- Utilities such as CLIs

## Operating systems

Most operating except Windows, are partly POSIX compliant. There are several OSes that are fully compliant including new versions of MacOS.

https://developer.huaweicloud.com/ict/en/site-euleros/euleros

https://www.opengroup.org/openbrand/register/brand3555.htm

## Bash

Earlier versions of Bash was POSIX compliant but now it became partially compliant.