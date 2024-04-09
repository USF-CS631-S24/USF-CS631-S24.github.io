---
layout: default
title: Lab07
nav_order: 11
parent: Assignments
permalink: /assignments/lab07
---

# Incremental Processor Design 

## Circuits due Tue Apr 16 by 11:59pm in your Lab07 GitHub Repo

## Requirements

1. For this lab, follow incremental development approach as described in lecture and the guides. Commit three top-level circuits: `lab07-part1.dig`, `lab07-part2.dig` and `lab07-part3.dig`. 
1. Submit all your `.dig` files, `.s` and `.hex` files, and a PDF of your instruction decoder spreadsheet. 


## Part 1 - A Partial Processor

For this part you should implement all the components and the top-level partial processor circuit described in the [Processor Guide Part 1](/guides/processor-part-1.html) with the following exceptions: the Extender and Data Memory.

In summary you need to implement:

1. A Program Counter using  a 64-bit Register with CLR. You may use your own register, or the Digital Register component
1. A Register File that has 32 registers. Two registers can be read, and one can be written, in a single clock cycle.
1. An ALU that supports `addi`, `sub`, `mul`, `sll`, and `srl`.
1. Your top-level processor should have a variation of the dashboard view using splitters, tunnels, and probes as shown in the guide. You do not have the replicate this view identically, but it should show the same information. You are free to come up with new and better ways to display the same information.
1. Your top-level circuit should be able to increment through a program in instruction memory showing each instruction word for the specified program, although this is not tested by the autograder in this lab.
1. By manipulating the inputs to the Register File, ALU, `CLK`, and `CLR`, your circuit should be able to execute four small programs:
    1. `addi t0, t0, 1` resulting in `T0` = 1
    1. `li t1, 2 `resulting in `T1` = 2
    1. `addi t0, t0, -1` resulting in `T0` = -1 (0xFFFFFFFF)
    1. this multi-instruction program

        `li t0, 1` resulting in `T0` = 1

        `li t1, 1` resulting in `T1` = 1

        `sub t0, t0, t1` resulting in `T0` = 0
1. Your ALU should be able to subtract A - B and calculate the correct result
1. For the autograder to test the four cases above:
    1. Your main circuit must be named `lab07-part1.dig`, have inputs named `CLK`, `CLR`, `RR0`, `RR1`, `WR`, `WE`, `ALUSrcB`, `ALUOp`, and `Imm`, and outputs named `T0` and `T1`
    1. Your `ALU` must be named `alu.dig`, have inputs named `A`, `B`, and `ALUOp`, and an output named `R`

## Part 2 - Our First Processor

1. Part one required manual manipulation. In this part you build a processor that can automatically execute code.

1. The top-level circuit should contain its own decoder circuit to identify instructions and propagate the appropriate control signals. 

1. For this part you will combine your work from part1 with a new implementation of the control lines as described in the [Processor Guide Part 2](/guides/processor-part-2.html)

1. Build decoders (for instructions, registers, and immediates) and top-level processor circuit which can execute this program (also given in the Guide)

1. Use the spreadsheet approach to develop a decoder table that associated inputs (opcode, funct3, funct7, and funct6) with decoder outputs (`RFW`, `ALUOp`, etc.).

1. You must have inputs for `CLK`, `EN`, `CLR`, and `PROG`, and outputs for `A0`, `A1`, `A2` and `DONE`


    ```
    first_s:
        li a0, 1
        li a1, 2
        add a2, a0, a1
        unimp
    ```
1. Your circuit must be named `lab07-part2.dig`.
    
## Part 3 - JAL and JALR

1. The top-level circuit should contain its own decoder circuit to identify instructions and propagate the appropriate control signals. 

1. For this part you will combine your work from part2 with a new implementation of the control lines as described in the [Processor Guide Part 2](/guides/processor-part-2.html)

1. Build the control and top-level processor circuit which can execute this program (also given in the Guide)
    ```
    main:    
        li a0, 1
        li a1, 2
        jal first_s
        unimp
    first_s:
        add a0, a0, a1
        ret
    ```

1. Your circuit must be named `lab07-part3.dig`
    
## Given

1. You may use any of the circuits shown in [Processor Guide Part 1](/guides/processor-part-1.html) or [Processor Guide Part 2](/guides/processor-part-2.html)
1. You may use any of Digital's built-in components, or your own if you prefer.

## Rubric

100 points from autograder tests

