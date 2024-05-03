---
layout: default
title: Project07
nav_order: 17
parent: Assignments
permalink: /assignments/project07
published: false
---

# Adding strace to xv6

## Due in your project07 GitHub repo by Thu May 9th at 11:59pm

## There is no interactive grading for project07

## Overview

The `strace` program is standard on many versions of Unix, including Linux that allows you to see all the system calls that a program invokes during execution. This can be helpful for debugging certain types of problems. It also helps you understand how system calls are used. You can learn more about it here:

[https://opensource.com/article/19/10/strace](https://opensource.com/article/19/10/strace)

We are going to modify xv6 to support a simple version of the `strace` command. You will make modifications to the kernel, add an `strace` system call, and write a new user level program called `strace` that will run a program with tracing on.



## Reading

You should read Chapters 1, 2, and 4 from the xv6 book.

[https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf)


## Part 1 - Adding the strace() system call



Adding a new user program to xv6 involves several steps. Here's a step-by-step guide on how to add a new user program, called hello.c, to xv6:

1. **Create the Program**

   First, you need to create your C program. In this case, we'll create a simple "Hello, World!" program. Save it as `hello.c` in the `xv6-riscv/user` directory.

```c
#include "kernel/types.h"
#include "user/user.h"

int main(void) {
    printf("Hello, World!\n");
    exit(0);
}
```

2. **Update the Makefile**

   Next, you need to add your program to the Makefile so that it gets compiled and linked when you build xv6. Open the `Makefile` in the `xv6-riscv` directory and find the `UPROGS` variable. Add your program to the list:

```makefile
UPROGS=\
    _cat\
    _echo\
    ...
    _hello\
    ...
```

3. **Build and Test the Program**

   Finally, you can test your program by running xv6 and then running your program. Start xv6:

```bash
$ make qemu
```

   Then, in the xv6 shell, run your program:

```bash
$ hello
```

   You should see "Hello, World!" printed to the console.

## Adding a new system call to xv6

Adding a new system call to the xv6 kernel involves several steps. Here's how you can add a system call named `kmemfree()` that returns the amount of free kernel memory available.

1. **Define the System Call**

   First, you need to define the system call. Open the `kernel/syscall.h` file and add a new system call number at the end of the list. For example:

```c
#define SYS_kmemfree 22
```

2. **Implement the System Call**

   Next, you need to implement the system call. Create a new function in `kernel/sysproc.c`:

```c
uint64
sys_kmemfree(void)
{
  return (uint64) kmemfree();
}
```

   Here, `kmemfree()` is a function that returns the amount of free kernel memory. You would need to implement this function in `kernel/kalloc.c`:

```c
uint64
kmemfree(void)
{
  struct run *r;
  uint64 free = 0;
  for(r = kmem.freelist; r; r=r->next)
    free += PGSIZE;
  return free;
}
```
3. **Add Prototype to kernel/defs.h**

```c
// kalloc.c
void*           kalloc(void);
void            kfree(void *);
void            kinit(void);
uint64          kmemfree(void);
```

4. **Add the System Call to the System Call Table**

   Then, you need to add the system call to the system call table. Open the `kernel/syscall.c` and add an `extern` for `sys_kmemfree` after the existing extern definitions.

```c
extern uint64 sys_kmemfree(void);
```
   
    Then add a new entry in the `syscalls[]` array:

```c
[SYS_kmemfree] sys_kmemfree,
```

5. **Add kmemfree() to user/usys.pl**

You need to add our new system call to `user/usys.pl` at the end:

```
entry("kmemfree");
```

6. **Add kmemfree() to user/user.h**

You need to add the prototype for `kmemfree()` to user/user.h:

```
int kmemfree(void);
```

7. **Create a User Program**

    Now, you can create a new user program that calls this system call. Create a new file `user/kmemfree.c`:

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  printf("Free kernel memory: %d\n", kmemfree());
  exit(0);
}
```

8. **Add the User Program to the Makefile**

    Finally, you need to add the user program to the Makefile. Open the `Makefile` and add `freekmem` to the `UPROGS` list:

```makefile
UPROGS=\
    _cat\
    ...
    _kmemfree\
    ...
```

    After these steps, you should be able to compile and run xv6 with the new system call and user program. You can test it by running `kmemfree` in the xv6 shell.

## Running the Autograder tests

Inside the starter xv6 code I've provided a program call runxv6.py. In order for the autograder to run this you will need to install the Python pexpect module:

```text
pip3 install pexpect
```

You will also have to install the autograder requirements. Go to your autograder repo and type:

```text
pip3 install -r requirements
```

