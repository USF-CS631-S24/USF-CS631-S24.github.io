---
layout: default
title: Project08
nav_order: 16
parent: Assignments
permalink: /assignments/project08
---

# Project08 Page Tables and Virtual Memory

## Source code due Mon May 16th by 11:59pm in your Project08 GitHub repo

## There will be no interactive grading for Project08

## You can use GitHub Copilot to help develop your solutions

[https://github.com/features/copilot](https://github.com/features/copilot)

[https://docs.github.com/en/copilot/quickstart](https://docs.github.com/en/copilot/quickstart)

## Overview

In this project you will get some experience with how the kernel manages the virtual address space of user proceses. By default every process is given its own virtual address space and user processes cannot access the memory (virtual address space) of anothe process. However, sometimes it is useful to allow two or more processes to share a portion of their virtual address space so they can communicte data among themselves. In modern Unix this can be achieved with the `mmap()` system call. We are going implement a much simpler version called `smem()`.

## Deliverables

- A copy of xv6 with your modified kernel to support the virtual memory requirements described below.
- Your implemenation should pass all of the Project08 Autograder tests.
- Your source code should conform to xv6 formatting conventions.
- Your Project04 repo should not have any extraneous files or build artifacts
- Make sure to test your repo by cloning from GitHub to a different location and run the Autograder. This will catch problems like missing files.

## Reading

[xv6 book Chapter 3](/files/xv6-book-riscv-rev3.pdf)

# Contents
1. [Kernel Free Pages](#kernel-free-pages)
2. [Page Directories](#page-directories)
3. [Process Shared Memory](#process-shared-memory)

# Kernel Free Pages

In order to help with the testing of later modifications we are going to add a new system call to xv6 called `int kpages(void)`. This system call will return the number of free kernel pages. The kernel allocator in `kernel/kalloc.c` keeps track of free kernel pages on a free list. Your job is to walk the freelist to count the number of pages available. The `user/kpages.c` program will show the number of free pages when executed. The `user/kpagestests.c` is provided to test your implementation of the `kpages()` system call. Note that a page in xv6 on riscv64 is 4KB (4096 bytes). The default memory size given to Qemu is 128 MB (megabytes). So, 128 * (2^20) = 134217728 bytes. (128 * (2^20))/(2^12) = 32768 total available pages. After startup you should see the available free pages close to this number. For me, I see 32554. This means the kernel and initial processes are using 32768 - 32554 = 214 pages, or (214 * (2^12))/(2^10) = 865 KB (kilobytes), which is not much for a kernel!

Note that this system call is slightly different than the `kmemfree()` system call which returned the number free bytes, not the number of free pages. You should put your implementation of `kpages()` into `kernel/kalloc.c` at the end of the file.

# Page Directories

To get experience traversing the page table structure you are going to add two more system calls: `int udirs(void)` and `int kdirs(void)`. The `udirs()` system call will return number of allocated page directories for the calling user process and the `kdirs()` system call will return the number of allocated page directories for the kernel page table. The test programs `user/udirs.c` and `user/kdirs.c` will show these values and will be used to test your implementations.

You should put your implementations of `udirs()` and `kdirs()` in `kernel/vm.c` at the end of the file.

Hint: look at `kernel/vm.c:freewalk()`.

# Process Shared Memory

As we know, when you `fork()` a child process, the child's memory is a copy of the parent's memory. However, the parent and child do not share any memory. It can be useful to allow two or more processs to share a region of memory. This could be useful to allow a child process to run in parallel on a seprate process to update shared data that will be used by the parent. We are going to implement a simple mechanism that allows a parent process to create a shared memory region that will be inherited by one or more child processes.

You will implement a new system call, `int smem(char *addr, int n)`, which stands for "shared memory". The `addr` parameter is an unmapped user virtual address and `n` is the size of the memory region in bytes. You can assume that both `addr` and `n` are both multiples of 4096 (PGSIZE), but your `smem()` implementation should check to make sure this is the case. For example,
```
int r;

r = smem(0x40000000, 4*4096);
```
will allocate and map 4 pages starting at address `0x40000000` (1GB). It will also update field in the `proc struct` to record the starting virtual address, the size, and the owner pid of this shared memory region. When this process forks a child, the shared memory region should be inherited by the child. Consider the following code snippet:

```
int r;
int id;
char *p;

p = (char *) 0x40000000;

r = smem(p, 4*4096);

p[0] = 'a';

id = fork();
if(id == 0){
  p[0] = 'b';
  exit(0);
}

wait(0);
if(p[0] == 'b'){
    printf("success\n");
}
```

In this example, the child can write to the inherited shared memory region and update it's contents, which can be seen by the parent.

#### Implementation 

To get the `smem()` system call to work properly you will have to consider several issues:

- In `proc.c:smem()` you need to check if both `addr` and `n` are multiplies of 4096. Return -1 if not.
- In `proc.c:smem()` you need to use `kalloc()` to allocate n / 4096 pages.
- In `proc.c:smem()` you need to use `mappages()` to map the allocated kernel pages starting at user virtual address `addr`.
- Hint: look at `proc.c:growproc()` which calls `vm.c:uvmalloc()`. Understanding this code is key to understanding how to use `kalloc()` and `mappages()` in your `smem()` implementation.
- In `proc.c:smem()` you need to update the calling (parent) proc struct with `addr`, `n`, and also record the owning pid of the shared memory region (the owner is the caller of `smem()`).
- In `proc.c:fork()` you will need to properly inherit the shared memory region in the child. In this case you need to create mappings in the child's page table that point to the kernel pages originally allocated for the shared memory region in the parent. Hint: look at `vm.c:uvmcopy()` for guidance on how to do this (note: `uvmcopy()` won't work for this directly, why?).
- In `proc.c:freeproc()` you need to handle two cases if the proc has a shared memory region. If the proc is the owner of the region, then you need to unmap the shared memory region and deallocate the kernel pages. If the proc is not the owner, then you need to just unmap the shared memory region. You will need to understand how to use `vm:uvmunmap()`.
