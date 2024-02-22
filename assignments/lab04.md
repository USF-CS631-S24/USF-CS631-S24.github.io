---
layout: default
title: Lab04
nav_order: 5
parent: Assignments
permalink: /assignments/lab04
---

# RISC-V Assembly Programming - Functions

## Code due Tue Feb 27st by 11:59pm in your Lab04 GitHub repo

## Links

Tests: [https://github.com/USF-CS631-S24/tests](https://github.com/USF-CS631-S24/tests)

Autograder: [https://github.com/phpeterson-usf/autograder](https://github.com/phpeterson-usf/autograder)


## Requirements

1. You will develop RISC-V assembly language implementations of the following problems. You will be given the C implementations, you need to write the RISC-V implementations. 
1. Your executable must be compiled with a Makefile
1. Before you add files to your repo, do a `$ make clean` so you don't add/commit build products like executables or .o files
1. We will test the labs using autograder

**max3**

Given three signed integer parameters, find the largest by comparing the first two, and then comparing the larger of those with the third. That is, max3_s must be implemented using two calls to `max2_s`.

    $ ./max3 2 4 6
    C: 6
    Asm: 6

**swap**

Given an array of integers, swap element at index i with the element at index j:

    ./swap <i> <j> <a0> <a1> <a2> ...

    $ ./swap 0 1 5 4 3 2 1
    C: 4 5 3 2 1
    Asm: 4 5 3 2 1
    ./swap 4 3 11 22 33 55 66 77
    C: 11 22 33 66 55 77
    Asm: 11 22 33 66 55 77

**sort**

Given the address of an array of unsigned integers, and the length of the array, sort the input array in increasing order (largest to smallest) in place using the given C implementation of insertion sort. Your `sort_s` implementation must call `swap_s`.

    $ ./sort 10 30 20
    C: 10 20 30
    Asm: 10 20 30

**fibrec:** Recursive Fibonacci number generator. Examples:

    $ ./fib_rec 10
    C: 55
    Asm: 55

    $ ./fib_rec 20
    C: 6765
    Asm: 6765

## Given

The Lab04 GitHub repo will contain starter code including a Makefile, the C versions of the functions above, and empty `.s` files for you to fill in.

## Rubric

1. Your lab will receive the score indicated by the autograder
1. To get the test cases, `git pull` in the tests repo
