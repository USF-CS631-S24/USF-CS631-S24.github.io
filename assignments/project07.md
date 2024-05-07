---
layout: default
title: Project07
nav_order: 15
parent: Assignments
permalink: /assignments/project07
published: true
---

# xv6 strace

## Due in your project07 GitHub repo by Thu May 16th at 11:59pm

## There is no interactive grading for project07

## Overview

In this project you will get experience using the xv6 systems calls for working with processes in Unix and with making modifcations to the kernel to support a command called `strace`.

The `strace` program is standard on many versions of Unix, including Linux that allows you to see all the system calls that a program invokes during execution. This can be helpful for debugging certain types of problems. It also helps you understand how system calls are used. You can learn more about it here:

[https://opensource.com/article/19/10/strace](https://opensource.com/article/19/10/strace)

We are going to modify xv6 to support a simple version of the `strace` command. You will make modifications to the kernel, add an `strace` system call, and write a new user level program called `strace` that will run a program with tracing on.

## Reading

You should read Chapters 1, 2, and 4 from the xv6 book:

[xv6 book Chapter 3](/files/xv6-book-riscv-rev3.pdf)

## Adding the strace() system call

To support strace you will need to add a new field to the `struct proc` in `proc.c`. You can call this field `int strace`. A value of 0 means tracing is off and a value of 1 means tracing is on. You will add a system call, `strace(int value)`, that will set the strace field appropriately. Only values of 0 and 1 should be accepted.

## Modifying the system calls

To support `strace` you will need to modify each of the system calls to print the tracing info if the `strace` value for the currently running process is 1. Note that in the kernel, you can get access to the currently running process struct using the `myproc()` function, you can combine this with field access, e.g., `myproc()->strace`. For buffers we will just print `addr` to make the autograder work. In some cases adding the trace printing is easy, in other cases you have to rework the code a bit in order to get access to the arguments before checking for errors, here is an example:

```c
uint64
sys_close(void)
{
  int fd;
  struct file *f;
  int rv;
  rv = argfd(0, &fd, &f);
  if (myproc()->trace)
    printf("[%d] sys_close(%d)\n", myproc()->pid, fd);
  if(rv < 0)
    return -1;
  myproc()->ofile[fd] = 0;
  fileclose(f);
  return 0;
} 
```

Compare this to the original code and notice the added rv variable and order of access, printing, and then error condition checking.

## The strace user program

The `strace` user program 

```text
$ strace echo a b c
[13] sys_write(1, addr, 1)
a[13] sys_write(1, addr, 1)
 [13] sys_write(1, addr, 1)
b[13] sys_write(1, addr, 1)
 [13] sys_write(1, addr, 1)
c[13] sys_write(1, addr, 1)

[13] sys_exit(0)
$
```

## sctest.c

```c

/* sctest.c - Test all system call for strace output testing */
#include "kernel/fcntl.h"
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

/*
int fork(void);
int exit(int) __attribute__((noreturn));
int wait(int*);
int pipe(int*);
int write(int, const void*, int);
int read(int, void*, int);
int close(int);
int kill(int);
int exec(const char*, char**);
int open(const char*, int);
int mknod(const char*, short, short);
int unlink(const char*);
int fstat(int fd, struct stat*);
int link(const char*, const char*);
int mkdir(const char*);
int chdir(const char*);
int dup(int);
int getpid(void);
char* sbrk(int);
int sleep(int);
int uptime(void);
*/

int
main(int argc, char *argv[])
{
    int id;
    char *nargv[4];
    char arg0[16] = "sctest";
    char arg1[16] = "a";
    nargv[0] = arg0;
    nargv[1] = arg1;
    nargv[2] = 0;
    int pfd[2];
    char buf[64];
    int i;
    int rv;
    int fd;
    struct stat stat_st;
    char *s;
    int r;

    /* For two tests, we will fork and then exec sctest with arguments.
     * If argv[1] = "a" then do the exit()/wait() test.
     * If argv[1] = "b" then do the kill()/wait() test.
     */
    if (argc > 1) {
        /* We did an exec on sctest to test exec. */
        if (argv[1][0] == 'a') {
            /* exec()/wait() test */
            printf("In child program: pid = %d\n", getpid());
            exit(0);
        } else if (argv[1][0] == 'b') {
            /* kill()/wait() test */
            printf("In child program: pid = %d\n", getpid());
            printf("In child waiting to be killed.\n");
            /* Loop forever until killed by parent. */
            while(1) {
                sleep(10);
            }
        }
    }

    /* uptime() test */
    rv = uptime();
    //printf("uptime() = %d\n", rv);
    
    /* fork()/exec()/wait() test */
    id = fork();

    if (id == 0) {
        nargv[1][0] = 'a';
        printf("In child process: pid = %d\n", getpid());
        exec("sctest", nargv);
    }

    /* Parent */
    id = wait(&r);

    /* fork()/exec()/kill()/wait() test */
    id = fork();

    if (id == 0) {
        nargv[1][0] = 'b';
        printf("In child process: pid = %d\n", getpid());
        exec("sctest", nargv);
    }

    for (i = 0; i < 3; i++) {
        sleep(10);
    }

    /* Kill child and wait. */
    kill(id);
    id = wait(0);
    printf("wait(0) = %d\n", id);

    /* Pipes test with dup() */
    pipe(pfd);

    write(pfd[1], "hello", 5);
    fd = dup(pfd[0]);
    read(fd, buf, 5);
    buf[5] = '\0';
    printf("pipe read buf = %s\n", buf);

    close(pfd[0]);
    close(pfd[1]);
    close(fd);

    /* Directories and Files test */
    mkdir("foo");

    fd = open("foo/test.txt", O_CREATE | O_RDWR);
    write(fd, "sctest\n", 7);
    close(fd);

    fd = open ("foo/test.txt", O_RDONLY);
    read(fd, buf, 7);
    buf[8] = '\0';
    printf(buf);
    close(fd);

    /* mknod() test */
    rv = mknod("foo/fakeconsole", 3, 27);
    printf("mknod() = %d\n", rv);

    /* unlink() test */
    rv = mknod("foo/fakeconsole2", 3, 27);
    printf("mknod() = %d\n", rv);
    
    unlink("foo/fakeconsole2");

    /* fstat() test */
    fd = open("foo", O_RDONLY);
    rv = fstat(fd, &stat_st);
    printf("type of \"foo\" is %d\n", stat_st.type);
    close(fd);

    /* chdir() test */
    chdir("foo");

    /* link() test */
    link("test.txt", "test2.txt");

    /* unlink() test */
    unlink("test.txt");

    /* chdir() test */
    chdir("..");

    /* sbrk() test */
    s = (char *) sbrk(4096);
    strcpy(s, "more memory");
    printf("data at <%x> = \"%s\"\n", s, s);
    
    exit(0);
}

```

## The sctest.c sample output

```text
$ strace sctest
[4] sys_exec(sctest)
[4] uptime()
[4] sys_write(1, addr, 1)
u[4] sys_write(1, addr, 1)
p[4] sys_write(1, addr, 1)
t[4] sys_write(1, addr, 1)
i[4] sys_write(1, addr, 1)
m[4] sys_write(1, addr, 1)
e[4] sys_write(1, addr, 1)
([4] sys_write(1, addr, 1)
)[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
=[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
1[4] sys_write(1, addr, 1)

[4] fork()
[4] wait(addr)
In child process: pid = 5
In child program: pid = 5
[4] fork()
[4] sleep(10)
In child process: pid = 6
In child program: pid = 6
In child waiting to be killed.
[4] sleep(10)
[4] sleep(10)
[4] kill(6)
[4] wait(addr)
[4] sys_write(1, addr, 1)
w[4] sys_write(1, addr, 1)
a[4] sys_write(1, addr, 1)
i[4] sys_write(1, addr, 1)
t[4] sys_write(1, addr, 1)
([4] sys_write(1, addr, 1)
0[4] sys_write(1, addr, 1)
)[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
=[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
6[4] sys_write(1, addr, 1)

[4] sys_pipe(addr)
[4] sys_write(4, addr, 5)
[4] sys_dup(3)
[4] sys_read(5, addr, 5)
[4] sys_write(1, addr, 1)
p[4] sys_write(1, addr, 1)
i[4] sys_write(1, addr, 1)
p[4] sys_write(1, addr, 1)
e[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
r[4] sys_write(1, addr, 1)
e[4] sys_write(1, addr, 1)
a[4] sys_write(1, addr, 1)
d[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
b[4] sys_write(1, addr, 1)
u[4] sys_write(1, addr, 1)
f[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
=[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
h[4] sys_write(1, addr, 1)
e[4] sys_write(1, addr, 1)
l[4] sys_write(1, addr, 1)
l[4] sys_write(1, addr, 1)
o[4] sys_write(1, addr, 1)

[4] sys_close(3)
[4] sys_close(4)
[4] sys_close(5)
[4] sys_mkdir(foo)
[4] sys_open(foo/test.txt, 514)
[4] sys_write(3, addr, 7)
[4] sys_close(3)
[4] sys_open(foo/test.txt, 0)
[4] sys_read(3, addr, 7)
[4] sys_write(1, addr, 1)
s[4] sys_write(1, addr, 1)
c[4] sys_write(1, addr, 1)
t[4] sys_write(1, addr, 1)
e[4] sys_write(1, addr, 1)
s[4] sys_write(1, addr, 1)
t[4] sys_write(1, addr, 1)

[4] sys_close(3)
[4] sys_mknod(foo/fakeconsole, 3, 27)
[4] sys_write(1, addr, 1)
m[4] sys_write(1, addr, 1)
k[4] sys_write(1, addr, 1)
n[4] sys_write(1, addr, 1)
o[4] sys_write(1, addr, 1)
d[4] sys_write(1, addr, 1)
([4] sys_write(1, addr, 1)
)[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
=[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
-[4] sys_write(1, addr, 1)
1[4] sys_write(1, addr, 1)

[4] sys_mknod(foo/fakeconsole2, 3, 27)
[4] sys_write(1, addr, 1)
m[4] sys_write(1, addr, 1)
k[4] sys_write(1, addr, 1)
n[4] sys_write(1, addr, 1)
o[4] sys_write(1, addr, 1)
d[4] sys_write(1, addr, 1)
([4] sys_write(1, addr, 1)
)[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
=[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
0[4] sys_write(1, addr, 1)

[4] sys_unlink(foo/fakeconsole2)
[4] sys_open(foo, 0)
[4] sys_fstat(3)
[4] sys_write(1, addr, 1)
t[4] sys_write(1, addr, 1)
y[4] sys_write(1, addr, 1)
p[4] sys_write(1, addr, 1)
e[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
o[4] sys_write(1, addr, 1)
f[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
"[4] sys_write(1, addr, 1)
f[4] sys_write(1, addr, 1)
o[4] sys_write(1, addr, 1)
o[4] sys_write(1, addr, 1)
"[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
i[4] sys_write(1, addr, 1)
s[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
1[4] sys_write(1, addr, 1)

[4] sys_close(3)
[4] sys_chdir(foo)
[4] sys_link(test.txt, test2.txt)
[4] sys_unlink(test.txt)
[4] sys_chdir(..)
[4] sbrk(4096)
[4] sys_write(1, addr, 1)
d[4] sys_write(1, addr, 1)
a[4] sys_write(1, addr, 1)
t[4] sys_write(1, addr, 1)
a[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
a[4] sys_write(1, addr, 1)
t[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
<[4] sys_write(1, addr, 1)
4[4] sys_write(1, addr, 1)
0[4] sys_write(1, addr, 1)
0[4] sys_write(1, addr, 1)
0[4] sys_write(1, addr, 1)
>[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
=[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
"[4] sys_write(1, addr, 1)
m[4] sys_write(1, addr, 1)
o[4] sys_write(1, addr, 1)
r[4] sys_write(1, addr, 1)
e[4] sys_write(1, addr, 1)
 [4] sys_write(1, addr, 1)
m[4] sys_write(1, addr, 1)
e[4] sys_write(1, addr, 1)
m[4] sys_write(1, addr, 1)
o[4] sys_write(1, addr, 1)
r[4] sys_write(1, addr, 1)
y[4] sys_write(1, addr, 1)
"[4] sys_write(1, addr, 1)

[4] exit(0)
```

