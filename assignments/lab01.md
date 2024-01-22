---
layout: default
title: Lab01
nav_order: 1
parent: Assignments
permalink: /assignments/lab01
---

# Hello World in the RISC-V Development Environment

## Deliverables due Wed Aug 30th by 11:59pm in your Project01 GitHub repo

## Links

Tests: [https://github.com/USF-CS315-F23/tests](https://github.com/USF-CS315-F23/tests)

Autograder: [https://github.com/phpeterson-usf/autograder](https://github.com/phpeterson-usf/autograder)

## Overview

This lab will get you up to speed in the development environment. You will learn how to do the following:

- Use git and access GitHub in the RISC-V VM
- Use a console-based editor to write C code
- Create a Makefile
- Install and run the autograder

## Program Requirements

You will write a program that takes a single argument, <str>, and prints "Hello, <str>". For example:

```text
$ ./lab01 World
Hello, World!
$ ./lab01 CS315
Hello, CS315!
```

For all labs and projects this semester the name of the main executable will be the name of the lab or project. For example, this is Lab01, so the name of the main executable will be ```lab01```.

We will use the Unix ```make``` program to compile and link our programs. By default the make command looks for build rules in a file called ```Makefile```. Here is a Makefile you can use for Lab01:

```
$ cat Makefile
PROG = lab01

OBJS = lab01.o

%.o: %.c
	gcc -c -g -o $@ $<

$(PROG): $(OBJS)
	gcc -g -o $@ $^

clean:
	rm -rf $(PROG) $(OBJS)
```

While this looks a little complicated, this form will be useful when we start to use Assembly Language and build larger programs. The ```PROG``` variable determines the name of the final executable. ```OBJS``` are individual modules, which could be written in C or in Assembly Language. In this example, we only have one, ```lab01.o```. The rule ```%.o : %.c``` Tells make how we want to compile C programs. We use gcc and tell it to include debugging information with ```-g```. The ```$(PROG) : $(OBJS)``` rule tells make how to build the main executable. In this case it just uses one file, ```lab01.o``` from the ```OBJS``` variable. The ```clean``` rule allows use to type ```make clean``` on the command line to remove any generated files.

## Autograder

To run the Autograder tests for Lab01, make sure you have cloned the tests repo for the class and you have configured the Autograder to point to the location of you tests repo in your home directory. Once you have the autograder installed and configured, you should be able to run the Lab01 tests like this:

```text
$ grade test -p lab01
. 01(50/50) 02(50/50) 100/100
```

## Code Submission

You will submit your code in your Lab01 GitHub repo. You will be provided a link to create your repo. Please only submit your Makefile, lab01.c, and optionally a README.md file. You should not include any binary files such as executables or object files. In general, you should not include any files that will be generated as a result of building your program with make.

## Rubric

100% Lab01 autograder tests.


