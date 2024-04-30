---
layout: default
title: Lab08
nav_order: 14
parent: Assignments
permalink: /assignments/lab08
published: false
---

# Getting Started with xv6

## Due in your lab08 GitHub repo by Fri May 3 at 11:59pm

## There is no interactive grading for lab08


## Overview

For our study of operating system kernel design we will explore and modify the xv6 kernel and user programs from MIT. The xv6 kernel is written mostly in C with some RISC-V Assembly. Because of this, we will be able to understand some of the low-level parts of the kernel. In this lab you will get your local development environment setup to compile, run, and debug xv6. We will be using Qemu to as our RISC-V emulator to run xv6. You main objective is to add a new user program to the xv6 user file system.

## xv6 Resources

The main xv6 site:

[https://pdos.csail.mit.edu/6.828/2023/xv6.html](https://pdos.csail.mit.edu/6.828/2023/xv6.html)

The xv6 book:

[https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf)

MIT Instructions for installation:

[https://pdos.csail.mit.edu/6.828/2023/tools.html](https://pdos.csail.mit.edu/6.828/2023/tools.html)

The GitHub repo for the RISC-V version of xv6:

[https://github.com/mit-pdos/xv6-riscv](https://github.com/mit-pdos/xv6-riscv)

## Installation Instructions

To compile and run xv6 you will need to install Qemu, RISC-V build tools, and gdb

### BeagleV Machines

We have qemu, the RISC-V build tools, and gdb already installed on the BeagleV machines. You can compile, run, and debug xv6 on the BeagleV machines.

### macOS

If you have 


### Ubuntu 


1. For this project, you will start with a given, basic implementation of a pipelined processor. This is a 5-stage pipeline processor that includes pipeline registers between each stage. To execute code on this processor, code must contain `nop` (no operation) instructions (`addi x0, x0, 0`) between real instructions so that the register file is updated properly. We will learn how this processor works in class this week.

1. You will evolve the given implementation to include data path additions and a Hazard Unit, which provide support for register forwarding for data dependencies, stalling for memory reads, and flushing the pipeline to handle branches. The ultimate goal is to remove the need for `nop` instructions to properly execute code and to allow the pipeline to run at the fastest rate possible.

1. You will commit the entire implementation of your pipelined processor, including the given circuits and ROMs, to your assignment repo. Your top-level processor must be named project06.dig.

## Requirements

1. For this project you will evolve the given processor design to include data path additions and a Hazard Unit. The Hazard Unit gives the processor the ability to automatically handle the hazard conditions described in the following test cases.

**04-add-fwd.s**

```
main:
    li a1, 1
    li a2, 2
    add a0, a1, a2   # a0 should be 3
    unimp            # marker instruction
```
This program illustrates a Data Hazard, since `a0` depends on the Writeback stage of the two immediate-form `li` (`addi`) instructions. Your Hazard Unit will enable Register Forwarding so that the values of `a1` and `a2` are available when the `add` instruction begins the Execute stage.

This test requires the implementation of forwarding from the MEM and WB stages to the EX stage if a instructions in these stages are going to write to RD0 or RD1. Here is is an outline of what is needed:

- New datapath lines that connect ALUR_3 and the MR_4 MUX output to two new MUXex in the EX stage.
- There will be two EX MUXes. One for RD0 and one for RD1.
  - That is the RD0 from the DR/EX registers will go to the MUX and the output of the MUX will go to all the inputs where RD0 was original connected. Same for RD1.
- The RD0 MUX will have three inputs: RD0, ALUR_3, and MR_4 and the selector is called FRD0.
- The RD1 MUX will have three inputs: RD1, ALUR_3, and MR_4 and the seledtor is called FRD1.
- The logic in the Hazard Unit for for FRD0, looks like this:
  ```
  if ((RR0_2 == WR_3) && (RFW_3)) {
      FRD0 = 1;
  } else if ((RR0_2 == WR_4) && (RFW_4)) {
      FRD0 = 2;
  } else {
      FRD0 = 0;
  }
  ```

**05-ld-stl.s**

```
main:
    li a0, 0
    li a1, 1
    sd a1, (a0)
    ld a2, (a0)
    addi a0, a2, 1   # a0 should be 2
    unimp            # marker instruction
```
This program illustrates the need to Flush the EX/MEM registers and Stall the pipeline, since the addi instruction depends on the value loaded by the `ld` instruction. Your Hazard Unit will enable a Stall in the appropriate Pipeline Registers.

Here is an outline of the implementation:

- We need to be able to flush the EX/MEM register so that we don't propogate the instruction in the EX stage. Essentially we need to insert a NOP. To do this, we can just set the CLR input to the EX/MEM registers to 1.
- We need to stall all the instructions in EX, DR, and IF:
  - We just need to disable (set enable to 0) on the DR/EX registers, IF/DR registers, and the PC.
- Here is the logic we need in the Hazard Unit:
  ```
  if ((RFW_3 == 1) && (MLD_3 == 1) && ((RR0_2 == WR_3) || (RR1_2 == WR_3))) {
      PC_EN = 0;
      IF_DR_EN = 0;
      DR_EX_EN = 0;
      EX_MEM_CLR = 1;
  } else {
      PC_EN = EN_ORG;
      IF_DR_EN = 1;
      DR_EX_EN = 1;
      EX_MEM_CLR = CLR_ORG;
  }
  ```

**06-jal-fls.s**

```
main:
    li a0, 2
    jal foo
    unimp           # marker instruction
foo:
    addi a0, a0, 4  # a0 should be 6
    ret
```
This program illustrates a Control Hazard. Since the `jal` instruction means that subsequent instructions (the marker) should not be executed, we need to update the PC in the EX stage and flush the pipeline upto EX.

Here is an outline of the implementation:

- The second input to the PCBr MUX should come from the ALU Result in the EX stage (not from the MR_4 MUX.
- The PCBr MUX selector should come from PCbr_2, not PCbr_4.
- The Hazard Unit need the following logic to Flush IF/DR and DR/EX:
  ```
  if (PCbr_2 == 1) {
     IF_EX_CLR = 1;
     EX_DR_CLR = 1;
  } else {
     IF_EX_CLR = CLR_ORG;
     EX_DR_CLR = CLR_ORG;
  }
  ```

## Given

1. You may use any of the circuits shown in lecture, including the pipeline registers and disassembly circuit.

1. The project06 starter circuits includes a simplified pipelined processor including:
    1. Support for RAM including lw and sw
    1. An implementation of the ASCII-based disassembler, including the decoder which outputs the inum to match the disassembler's ROM.
    1. Instruction memory, including assembly source and hex files, for:
        1. Four programs which execute using nop instructions to manually avoid hazards
        1. The three programs which require your Hazard Unit to execute correctly

1. You may use any of Digital's built-in components, or your own if you prefer.

## Rubric
- 10 pts: passes `00-add-3nop`, `01-add-2nop`, `02-jal`, `03-lw` test cases
- 70 pts: passes the `04-add-fwd` test case
- 10 pts: passes the `05-ld-stl` test case 
- 10 pts: passes the `06-jal-fls` test case


## Extra Credit

- (5 points) Get all the Project05 tests to run on your Project06 pipelined processor.
