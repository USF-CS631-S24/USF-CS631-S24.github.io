---
layout: default
title: Project02
nav_order: 6
parent: Assignments
permalink: /assignments/project02
published: false
---

# RISC-V Assembly and NTLang Code Generation

## Due Wed Mar 6th by 11:59pm in your Project02 GitHub repo

## Note: there will be no interactive grading for Project02. We will review your code offline.

## Links

Tests: [https://github.com/USF-CS631-S24/tests](https://github.com/USF-CS631-S24/tests)

Autograder: [https://github.com/phpeterson-usf/autograder](https://github.com/phpeterson-usf/autograder)

## Background

In this project you will continue to write RISC-V Assembly code, now focusing on programs that work with string data in addition to integer data. You will also build a simple compiler for NTLang expressions.


## RISC-V Assembly Program Requirements

1. You will develop RISC-V assembly language implementations of the following problems, and print the results to ensure that both the C implementation and your RISC-V implementation compute the correct answer.
1. Your executables must be named as follows, and must be compiled with a `Makefile`
1. We will test your projects using autograder
1. Note that in all your programs below you must follow the RISC-V function calling conventions that we cover in class.
1. Your solutions must follow the logic and implement the same functions as given in the C code.
1. Remeber to remove the line `# YOUR CODE HERE` from the starter code.

**rstr:** Reverse a string

    $ ./rstr a
    C: a
    Asm: a

    $ ./rstr FooBar
    C: raBooF
    Asm: raBooF

    $ ./rstr "CS315 Computer Architecture"
    C: erutcetihcrA retupmoC 513SC
    Asm: erutcetihcrA retupmoC 513SC

**rstr_rec:** Reverse a string recursively

    $ ./rstr_rec a
    C: a
    Asm: a

    $ ./rstr_rec FooBar
    C: raBooF
    Asm: raBooF

    $ ./rstr_rec  "CS315 Computer Architecture"
    C: erutcetihcrA retupmoC 513SC
    Asm: erutcetihcrA retupmoC 513SC

**pack_bytes**

Given four 1-byte values (unsigned), you will combine them into a single 32-bit signed integer value.

    $ ./pack_bytes
    usage: pack_bytes <b3> <b2> <b1> <b0>

    $ ./pack_bytes 1 2 3 4
    C: 16909060
    Asm: 16909060

    $ ./pack_bytes 255 255 255 255
    C: -1
    Asm: -1

**unpack_bytes**

Given a signed 32 bit integer value, unpack each byte as an unsigned value:

    $ ./unpack_bytes 1000000
    C: 0 15 66 64
    Asm: 0 15 66 64

    $ ./unpack_bytes -2
    C: 255 255 255 254
    Asm: 255 255 255 254

**get_bitseq** (this will be developed in class)

Given a 32-bit unsigned integer, extract a sequence of bits and return as an unsigned integer. Bits are numbered from 0
 at the least-significant place to 31 at the most-significant place. Here is the usage:

`get_bitseq <value> <start_bit> <end_bit>`

    $ ./get_bitseq 94117 12 15
    C: 6
    Asm: 6

    $./get_bitseq 94117 4 7
    C: 10
    Asm: 10

**get_bitseq_signed**

Given a 32-bit unsigned integer, extract a sequence of bits and return as a signed integer. Bits are numbered from 0 at
 the least-significant place to 31 at the most-significant place. When computing the signed return value you can assume
 the end_bit is the most significant bit of the signed bit range. You need to sign-extend this value to become a 32-bit
 2's complement signed value. Here is the usage:

`get_bitseq_signed <value> <start_bit> <end_bit>`

    $ ./get_bitseq_signed 94117 12 15
    C: 6
    Asm: 6

    $./get_bitseq_signed 94117 4 7
    C: -6
    Asm: -6

## NTLang Code Generation Requirements



## Grading

1. Grading will have two components: automated and interactive.

2. (80%) Automated tests.

3. (20%) For interactive grading. We will schedule 1:1 meetings with the instructor or TAs using a Google Sheet published on Campuswire. You should be prepared to share your screen and answer questions about your design and implementation.

4. Code Quality - These will be deduction from your overall points that can be earned back. We are looking for:
  - Consistent spacing and indentation
  - Consistent naming and commenting
  - No commented-out ("dead") code
  - No redundant or overly complicated code
  - A clean repo, that is no build products, extra files, etc.

