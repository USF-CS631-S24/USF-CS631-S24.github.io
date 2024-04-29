---
layout: default
title: Project05
nav_order: 12
parent: Assignments
permalink: /assignments/project05
published: true
---

# RISC-V Processor Implementation

## Circuits due Mon Apr 22nd by 11:59pm in your Project05 GitHub repo

## Interactive Grading on Tue Apr 23rd and Wed Apr 24th.

## Overview

In this project you will complete the RISC-V single cycle implementation so that it is capable of running all of the RISC-V assembly programs you wrote in Project02 and then were able to emulate in Project03. In addition, you are going to modify your Project02 version of NTLang so that it can generate code that can execute directly on your processor implementation.

## Deliverables

1. Submit all of the .dig files and .hex files required to run your processor implementation
1. Submit your modified NTLang for processor codegen
1. Submit a PDF for your Decoder spreadsheets

## Processor Requirements

1. You will implement a single-cycle microarchitecture for a subset of the RISC-V instruction set architecture in Digital
    1. You may use any of Digital's library of components
    1. We will introduce some new and useful tools and techniques for Digital in lecture
1. You need to pass unit tests and complete program tests provided below. The unit tests will help you incrementally build and test your processor. We will provide a starter instruction memory with the unit tests and complete programs.
1. In order to make your own programs for testing runnable on your processor you must do the following:
    1. You will add an assembly language main to setup input values for your function
    1. You will add the end marker (`unimp`) to indicate when the program should stop
    1. Ensure that you do not have `.global` directives in your programs
1. Your processor implementation will include the following major sub-circuits:
    1. The Program Counter (`PC`) will be one 64-bit register with enable (`EN`) and clear (`CLR`) inputs.
    1. Machine code programs will be stored in ROM components, just as we did in Project04. Like in Project04 your instruction memory will be able to select the program you want to execute.
    1. A Register File that can support reading 2 registers in a single clock cycle: `RD0`, `RD1`
    1. The Arithmetic Logic Unit (ALU) will perform data processing tasks such as `add`, `sub`, `mul`, and shifting (`sll`, `slr`, `sra`). You will need to support more than these operations.
    1. The Branch Control Unit (BCU) will perform conditional branch computation and PC updates. You will need to decide where to put this logic in the top-level circuit.
    1. Data Memory. We will use a Digital RAM component for data memory. This will be used for stack memory by our test programs. We will configure the RAM component to hold 64 bit word values. This will make it easy to support `ld` and `sd`. You will need to add logic for word- and byte-size memory operations.
    1. The Control Unit will decode machine code instructions and generate control signals.
    1. The Data Path will connect data between the various sub-circuits
    1. The Control Path will connect the Control unit to various sub-circuits and multiplexers
    1. Your top-level processor circuit should have outputs for registers A0, A1, A2, and A3.

## NTLang Codegen Requirements (Extra Credit - 10 points)

You can put your project02 code with NTLang in your project05 repo as a subdirectory. You need to modify the NTLang code generation so that the assembly language that is generated can execute on the processor. We will discuss in class how to form assembly code to run on the processor. You can have your project02 Makefile compile the expressions to .hex files then manually load the hex files into the instruction memory of your processor. 

You need to be able to compile and run the following ntlang expressions:

**23-cg-1**
```
(3 * 4) + 5
```

**24-cg-2**
```
(a0 * a1) + a2
```

**25-cg-3**
```
(((((~((-(2 * ((1023 + 1) / 4)) >- 2) << 8)) >> 10) ^ 0b011
10) & 0x1E) | ~(0b10000))
```

**26-cg-4**
```
(((((~((-(a0 * ((a1 + a2) / a3)) >- a4) << a5)) >> a6) ^ 0b01110) & 0x1E) | ~(0b10000))
```

In order to support initial values for `a0` through `a7` in you processor, we are going to add a new instuction that will load an IO argument (`ioA0`, `ioA1`, etc) into a destination register. We can use `lbu` for this purpose:

```text
lbu rd, (rs1)
```

In this case `rd` will be the destination register and `rs1` will be the number of the IO argument you are loading from, with possible values of 0, 1, ..., 7. Your IO arguments should be named `ioA0`, `ioA1`, etc to be compatible with the tests. Now, your preamble should look like this:

```text
main:
    li sp, 1024

    lbu a0, (x0)
    lbu a1, (x1)
    lbu a2, (x2)
    lbu a3, (x3)
    lbu a4, (x4)
    lbu a5, (x5)
    lbu a6, (x6)
    lbu a7, (x7)

    jal codegen_func_s

    unimp

codegen_func_s:
```

I have created separate tests for the codegen portion. They are in `tests/project05-cg`. To run these tests you need to have a top-level circuit named `project05-gc` and you can run the tests like this:

```text
$ grade test -p project05-cg
```

This assumes you've added the generated codegen tests from above into your instruction memory.

## Given

1. We will discuss the major sub-circuits in lecture and you will have hands-on time to develop and ask questions
1. We have compiled [Processor Guide Part 3](/guides/processor-part-3.html)
1. We will provide a starter instruction memory with the unit tests and complete program tests.
1. We will provide a Python script for creating a decode ROM hex file.

## Tests

See the tests repo for Project05. You will have to copy the inst-mem.dig file from the Projec05 test repo to your project05 repo.

## Rubric

For interactive grading you should be able to run the autograder tests and manually execute any required test.

- (30 points) Automated unit tests
- (50 points) Automated prog tests
- (20 points) Interactive grading
- (0 points) Neatness - Circuit quality
