---
layout: default
title: RISC-V Assembly
nav_order: 6
parent: Guides
permalink: /guides/riscv-asm
---

# RISC-V References

[RISC-V Green Card - Reference](/files/RISCVGreenCardv8.pdf)

[RISC-V Spec](/files/riscv-spec.pdf)

# RISC-V Assembly Language Overview

RISC-V is an open standard instruction set architecture (ISA) based on established reduced instruction set computer (RISC) principles. It is a simple, clean, and efficient assembly language. Assembly language is a human readable (mostly) form of the computer machine code (binary values). Ultimately, a computer only know how to execute machine code. We use an assembler (as) to translate assembly code into machine code, which is similar to how a C compiler generates machine code from C source code.

## Instructions and Registers

When thinking about assembly or machine code we need to think about core elements of a computer and computer processor. These elements include registers, memory, and instructions. A processor can only operate directly on values in registers. Memory holds both code (machine code) and data (globals, locals on the stack, and the heap). A processor executes instruction by loading an instruction from memory, then decoding it, then executing it. If program needs to operate on data, we need to use instruction to load data from memory into registers. Once the data is loaded into register, we compute new register values and often write the new values back to memory.

RISC-V assembly language has a set of instructions that operate on a set of 32 registers. Each register is 64 bits wide in the 64-bit variant of the ISA. The registers are named `x0` through `x31`. However, the ABI names are more commonly used:

- `zero` (x0): Holds the constant value 0.
- `ra` (x1): Return address.
- `sp` (x2): Stack pointer.
- `gp` (x3): Global pointer.
- `tp` (x4): Thread pointer.
- `t0-t6` (x5-x10): Temporary/alternate link registers.
- `a0-a7` (x10-x17): Function arguments/return values.
- `s0-s11` (x18-x31): Saved registers.
- `t3-t6` (x28-x31): More temporary registers.

Instructions typically take the form `opcode dst, src1, src2`, where `opcode` is the operation to perform, and `dst`, `src1`, and `src2` are the destination and source registers.

For example, to add the values in `a1` and `a2` and store the result in `a3`, you would write:

```text
add a0, a1, a2
```

## Labels and .global Directives

Labels in RISC-V assembly provide a way to name a specific location in your code. This is useful for branching or jumping to different parts of your program. For example:

```text
start:  # This is a label
    add a0, a0, a2
    add a0, a1, a3
```

Instructions are executed one after the other unless we direct the processor to go to an instruction at a different location in memory (see below).

The `.global` directive makes a label visible to the linker so it can be used from other files. This is typically used to define the entry point of your program:

```text
.global foo
foo:
    # Your code here
```

## Types of Instructions

RISC-V instructions can be broadly categorized into three types: data processing, control, and memory.

### Data Processing Instructions

These instructions are used to perform operations on data. Examples include `add` (addition), `addi` (add immediate), `sub` (subtraction), `mul` (multiplication), and `div` (division). For example:

```text
add a3, a1, a2  # a3 = a1 + a2
addi a3, a1, 10  # a3 = a1 + 10
sub a3, a1, a2  # a3 = a1 - a2
mul a3, a1, a2  # a3 = a1 * a2
div a3, a1, a2  # a3 = a1 / a2
```

We can load constant values into a register like this:

```text
li t0, 9
addi t0, zero, 9
```

These both do the same thing.

### Control Instructions

Control instructions are used to alter the flow of execution. Examples include `j` (jump), `beq` (branch if equal), `bne` (branch if not equal), `blt` (branch if less than), and `bge` (branch if greater than or equal). For example:

```assembly
j label  # Jump to label
beq a1, a2, label  # If a1 == a2, jump to label
bne a1, a2, label  # If a1 != a2, jump to label
blt a1, a2, label  # If a1 < a2, jump to label
bge a1, a2, label  # If a1 >= a2, jump to label
```

### Memory Instructions

Memory in RISC-V is byte-addressable, meaning each byte has a unique address. Larger values occupy multiple bytes. For example, a 64-bit word occupies 8 bytes.

Memory instructions are used to load and store data from and to memory. Examples include `lw` (load word), `sw` (store word), `ld` (load doubleword), and `sd` (store doubleword). For example:

```text
lw a1, (a2)  # Load word from memory at address in a2 into a1
sw a1, (a2)  # Store word in a1 into memory at address in a2
ld a1, (a2)  # Load doubleword from memory at address in a2 into a1
sd a1, (a2)  # Store doubleword in a1 into memory at address in a2
```

You can add an offset to the memory instructions that can be added to the base address:

```text
lw a1, 4(a2)  # Load word from memory at address a2 + 4
```

## Writing Functions

In RISC-V, functions are defined using labels. Here's a simple function that adds two numbers:

```text
add_numbers:
    add a0, a0, a2  # a0 = a0 + a`
    ret
```

This function takes its arguments in `a0` and `a1`, and returns the result in `a0`.

## RISC-V Function Call Conventions

When calling a function, arguments are passed in registers `a0-a7`. The return address is stored in `ra`, and the stack pointer in `sp` must be preserved across function calls.

Registers `a0-a7`, `t0-t6`, and `ra` are caller-saved, meaning if a function modifies these registers, it must save the old values on the stack and restore them before returning.

Registers `s0-s11` and `sp` are callee-saved, meaning if a function modifies these registers, it must save the old values on the stack and restore them before returning.

Here's an example of a function that calls another function, following these conventions:

```text
.global foo
foo:
    addi a0, zero, 2  # First argument
    addi a1, zero, 3  # Second argument
    call add_numbers  # Call function
    # Result is now in a0
    ret

add_numbers:
    add a0, a0, a1  # a0 = a0 + a1
    ret
```

## Translating C to Assembly

Here's how you might translate some common C constructs to RISC-V assembly:

### If/Else

```c
if (a == b) {
    c = a;
} else {
    c = b;
}
```

```text
beq a0, a1, equal
    add t0, zero, a1
    j end
equal:
    add t0, zero, a0
end:
```

### For Loop

```c
for (i = 0; i < 10; i++) {
    a = a + i;
}
```

```text
addi t0, zero, 0      # t0 = 0
addi t1, zero, 10     # t1 = 10
loop:
    blt t0, t1, end   # If i >= 10, jump to end
    add a0, a0, t0    # a0 = a0 + t0
    addi t0, t0, 1    # t0 = t0 + 1
    j loop            # Jump to start of loop
end:
```

## Array Access

Here's how you might implement array indexing in RISC-V assembly:

```c
x = arr[i];
```

```text
# a0 - int arr[]
# a1 - int i
li t0, 4
mul t0, a1, t0  # t0 = i * 4
add t0, a0, t0  # t0 = a0 + (i * 4)
lw t1, (t0)     # x = arr[i]
```

Here is way to do the multiplication with a shift instead of multiply:

```text
# a0 - int arr[]
# a1 - int i
slli t0, t0, 2  # t0 = t0 * 4
add t0, a0, t0  # t0 = a0 + (i * 4)
lw t1, (t0)     # x = arr[i]
```
