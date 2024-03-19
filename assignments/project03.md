---
layout: default
title: Project03
nav_order: 8
parent: Assignments
permalink: /assignments/project03
---

# RISC-V Emulation - Analysis - Cache Simulation

## Code due Mon Mar 25th by 11:59pm in your Project03 GitHub repo

## Interactive Grading on Tue Mar 26th and Wed Mar 27th

## Links

Tests: [https://github.com/USF-CS631-S24/tests](https://github.com/USF-CS631-S24/tests)

Autograder: [https://github.com/phpeterson-usf/autograder](https://github.com/phpeterson-usf/autograder)

## Requirements 

1. You will write an emulator in C for a subset of the RISC-V Instruction Set Architecture (ISA). 

1. You do not have to emulate the entire instruction set; just enough to emulate the following programs:
    - `quadratic_s` (given)
    - `midpoint_s` (given)
    - `max3_s` (given)
    - `to_upper` (given)
    - `get_bitseq_s` (given)
    - `get_bitseq_signed_s` (given)
    - `swap_s` (given)
    - `sort_s` (given)
    - `fib_rec_s` (yours)

1. Your emulator will need the logic (decoding and emulating instructions) and state (`struct rv_state_st`) from lab05

1. Your emulator will support dynamic analysis of instruction execution. Here are the metrics you will collect:
    1. \# of instructions executed (`i_count`)
    1. \# of I-type and R-type instructions executed (`ir_count`)
    1. \# of LOAD instructions executed (`ld_count`)
    1. \# of STORE instructions executed (`st_count`)
    1. \# of jump instructions executed including `j`, `jal`, `jalr` (`j_count`)
    1. \# of conditional branches taken (`b_taken`)
    1. \# of conditional branches not taken (`b_not_taken`)

1. Your emulator will include an implementation of a processor cache simulator for the following cache types: 
    1. A direct mapped cache with a block size on 1 word (given)
    1. A direct mapped cache with a block size of 4 words
    1. A 4-way set associative cache with a block size of 1 word and LRU slot replacement
    1. A 4-way set associative cache with a block size of 4 words and LRU slot replacement

## Given
1. In lecture and lab, we will: 
    1. Illustrate how to decode machine code and execute the operations specified
    1. Illustrate a direct-mapped cache and describe the data structures and algorithms required for a set-associative cache
    1. We have written a [Guide to Cache Memory]({{ site.url }}/guides/cache-memory.html) to help you develop your cache implementation
1. In-class coding will be pushed to Github. You will provide the rest of the code yourself
1. We will provide autograder test cases for the emulation targets

## Grading Rubric
**Automated testing**

90 pts: Automated tests

**Interactive grading**

10 pts: code walkthrough including, but not limited to, your implementation of dynamic analysis and the instruction cache.

**Code Quality**

You need to have a clean repo, consistent naming and indentation, no dead code, no unnecessarily complex code. Any deductions can be earned back.

## Extra Credit

1. (5 points) Implement the RISC-V emulator in either Rust (https://www.rust-lang.org) or Zig (https://ziglang.org). You can only get credit for re-implementation in one language.
