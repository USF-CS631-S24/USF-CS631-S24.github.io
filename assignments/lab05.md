---
layout: default
title: Lab05
nav_order: 7
parent: Assignments
permalink: /assignments/lab05
---

# RISC-V Emulation

## Code due Tue Mar 19th by 11:59pm in your Lab05 GitHub repo

## Links

Tests: [https://github.com/USF-CS631-S24/tests](https://github.com/USF-CS631-S24/tests)

Autograder: [https://github.com/phpeterson-usf/autograder](https://github.com/phpeterson-usf/autograder)

## Requirements

1. You will use the given code, lecture material, and RISC-V reference manual to develop a C program which can read RISC-V machine code and emulate what the instructions would do on a real processor.

1. For reference see the [RISC-V ISA Specification Volume 1](https://github.com/riscv/riscv-isa-manual/releases/download/Ratified-IMAFDQC/riscv-spec-20191213.pdf)
    1. Chapter 2 RV32I Base Integer Instruction  Set (pages 13-24)
    1. Chapter 5 RV64I Base Integer Instruction Set (pages 35-40)
    1. Chapter 7 "M" Standard Extension for Integer Multiplication and Division (pages 43-44)
    1. Chapter 24 RV32/64G Instruction Set Listings (pages 129-131)
    1. Chapter 25 RISC-V Assembly Programmers Handbook - Pseudo Instructions (pages 137-140)
    1. [General  RISC-V emulation guide](https://github.com/usfca-cs-tools/docs/blob/main/risc-v-emulation.md)

1. You do not need to emulate the whole RISC-V processor -- just enough to execute the given assembly programs: `quadratic_s`, `midpoint_s`, `max3_s`, `get_bitseq_s`, and `sumarr_idx_s`.

1. The given code provides a framework for testing that the emulator output matches the output of the C and assembly language versions of the programs

1. Your emulator will need logic, including data processing (`add`, `mv`, `sub`, etc.), and branching (`j`, `bCC`, `jal` (`call`), `jalr` (`ret`)), and memory (`lw`) to run the programs

1. Your emulator will need state, including the general purpose registers and the program counter (`pc`)

## Example Output
    $ ./lab05 quadratic 2 4 6 8
    C: 36
    Asm: 36
    Emu: 36

    $ ./lab05 midpoint 0 4
    C: 2
    Asm: 2
    Emu: 2

    $ ./lab05 max3 3 8 5
    C: 8
    Asm: 8
    Emu: 8

    $ ./lab05 get_bitseq 94117 12 15
    C: 6
    Asm: 6
    Emu: 6

    $ ./lab05 sumarr 1 2 3 4 5
    C: 15
    Asm: 15
    Asm: 15

## Code Quality

Note, like the other labs, we will not be checking your Lab05 solution for code quality, but we will be checking Project03 for code quality. You will want to make sure that you reduce redundancy in your code and also avoid using long binary literals for masks. Instead, you should implement `bits.h` and `bits.c` with the following functions:

```text
$ cat bits.h
#include <stdint.h>
#include <stdbool.h>

uint32_t get_bits(uint64_t num, uint32_t start, uint32_t count);
int64_t sign_extend(uint64_t num, uint32_t start);
bool get_bit(uint64_t num, uint32_t which);
```

## Rubric

1. Your lab will receive the score indicated by the autograder
1. To get the test cases, `git pull` in the tests repo
1. You will not be evaluated on code qaulity for Lab05, but you will be for Project03.
