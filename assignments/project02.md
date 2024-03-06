---
layout: default
title: Project02
nav_order: 6
parent: Assignments
permalink: /assignments/project02
published: true
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

A compiler translates a high-level language to assembly code or machine code directly. You can think of NTLang as a mini high-level language. In this part of the project you are going to add a compiler mode to NTLang so that it will generate RISC-V assembly code that can be assembled and linked to become an executable, which you can run directly on a RISC-V processor (and eventually on the RISC-V that you will build later this semester).

### NTLang Parameters

The current NTLang is not very interesting to compile as is, because NTLang expressions have no parameters (variables). A real compiler would be able to evaluate expressions at compile time. That is, something like `x = (3 + 4) * 5` would reduce to `x = 35` at compile time. So, first we are going to add simple paramters to NTLang that will work like this:

```text
$ ./ntlang -e "(a0 + a1) * a2" -a0 3 -a1 4 -a2 5
35
```

So your first goal will be to add this paramter support to your implementation of ntlang. Note you only have to support 8 parameters: a0, a1, a2, a3, a4, a5, a6, a7. You will notice that these correspond to the RISC-V registers used as function call arguments.

To add this support, you will need to add a new token, modify the scanner, modify the parser, and then the evaluator to reference the paramters. You can add an args array to the nt_config struct. You will also need to add command line parsing support for up to 8 parameters.

Here is the updated micro syntax:

```text
/*

# Scanner EBNF

whitespace  ::=  (' ' | '\t') (' ' | '\t')*

tokenlist  ::= (token)*
token      ::= intlit | hexlit | binlit | symbol
symbol     ::= '+' | '-' | '*' | '/' | '>>' | '>-' | '<<' | '~' | '&' | '|' | '^'
intlit     ::= digit (digit)*
regname    ::= 'a0' | 'a1' | ... | 'a7'
hexlit     ::= '0x' | hexdigit (hexdigit)*
binlit     ::= '0b' ['0', '1'] (['0', '1'])*
hexdigit   ::= 'a' | ... | 'f' | 'A' | ... | 'F' | digit
digit      ::= '0' | ... | '9'

# Ignore
whitespace ::= ' ' | '\t' (' ' | '\t')*

*/
```

Here is the updated parser grammar:

```text
/*

# Parser

program    ::= expression EOT

expression ::= operand (operator operand)*

operand    ::= intlit
             | hexlit
             | binlit
             | regname
             | '-' operand
             | '~' operand
             | '(' expression ')'
*/
```

### Code Generation

To support a compiler mode for NTLang, you will add a new option called `-c <name>` that tells NTLang to compile instead of interpreting an expression:

```text
$ ./ntlang -e "(a0 + a1) * a2" -c foo > foo.s
$ gcc -o foo foo.s
$ ./foo 3 4 5
35 (0x23)
$ ./foo 1 2 3
9 (0x9)
```

Note that in the compiled version you don't need to support `-b` or `-w`.

Compilation (generated of the .s file)  will invole two parts:
- Output a preamble in assembly that parses the arguments `codegen_main.s`
- Output a stack machine code from the parse tree

Here is a C version of `codegen_main.s`:

```
#include <stdio.h>
#include <stdlib.h>

int codegen_func_s(int a0, int a1, int a2, int a3,
                  int a4, int a5, int a6, int a7);

int main(int argc, char *argv[]) {
    int a[8];
    int i;
    int r;

    // Initialize all 8 args to 0 to be passed to codegen_func
    for (i = 0; i < 8; i++) {
        a[i] = 0;
    }

    // Populate args with up to 8 args from command line
    for (i = 1; i < argc && i < 9; i++) {
        a[i - 1] = atoi(argv[i]);
    }

    r = codegen_func_s(a[0], a[1], a[2], a[3], a[4], a[5], a[6], a[7]);

    printf("%d (0x%X)\n", r, r);

    return 0;
}
```

You need to write the equivalent RISC-V version of this and put it into `codegen_main.s`.

When generating the output, you can use this C function to write out `codegen_main.s`:

```text
void compile_output_main(char *name) {
    int fd;
    char c;
    int rv;

    fd = open(name, O_RDONLY);
    while (true) {
        rv = read(fd, &c, 1);
        if (rv <= 0) {
            break;
        }
        printf("%c", c);
    }
}
```

Next you will need to create a new module, I would call it `compile.c` based on `eval.c` that will generate stack machine code:

[https://en.wikipedia.org/wiki/Stack_machine](https://en.wikipedia.org/wiki/Stack_machine)

For example, here is the function output for:

```text
$ ./ntlang -e "(a0 + a1) * a2" -c foo > foo.s
```

```text
codegen_func_s:
foo:
    addi sp, sp, -4
    sw a0, (sp)
    addi sp, sp, -4
    sw a1, (sp)
    lw t1, (sp)
    addi sp, sp, 4
    lw t0, (sp)
    add t0, t0, t1
    sw t0, (sp)
    addi sp, sp, -4
    sw a2, (sp)
    lw t1, (sp)
    addi sp, sp, 4
    lw t0, (sp)
    mul t0, t0, t1
    sw t0, (sp)
    lw a0, (sp)
    addi sp, sp, 4
    ret
```

The way the generated stack works is as follows:

- Constants are put onto the stack:

    ```text
    addi sp, sp, -4
    li t0, 4
    sw t0, (sp)
    ```

- Parameters are put onto the stack:

    ```text
    addi sp, sp, -4
    sw a0, (sp)
    ```

- Unary operator removes value from the stack, applies operator, then pushes back onto the stack:

    ```text
    lw t0, (sp)
    // Apply operator
    sw t0, (sp)
    ```

- Binary operators removes two values from the stack, applies operator, then pushes back onto the stack:

    ```text
    lw t1, (sp)
    addi sp, sp, 4
    lw t0, (sp)
    add t0, t0, t1
    sw t0, (sp)
    ```

## Grading

1. (100%) Automated tests.

2. Code Quality - These will be deduction from your overall points that can be earned back. We are looking for:
  - Consistent spacing and indentation
  - Consistent naming and commenting
  - No commented-out ("dead") code
  - No redundant or overly complicated code
  - A clean repo, that is no build products, extra files, etc.


## Extra Credit Problems
1. (1 point) Add constant folding support to NTLang compiler mode:

    [https://en.wikipedia.org/wiki/Constant_folding](https://en.wikipedia.org/wiki/Constant_folding). 
    
    Create test cases that demonstrate your solution works.

2. (3 points) Extend NTLang compiler mode to support variables and statements. Specifically add support for assignment and a print statement. For example consider the following new NTLang programs:
 
```text
$ cat p1.nt
x = 1
y = 2
z = x + y
print(z, 10, 32)

$ cat p2.nt
print((0b1 << 8) | 0xF), 16, 8)

$ cat p3.nt
x = -1
print(x, 2, 8)
print(x, 2, 16)

$ cat p4.nt
x = -8
print(x, -10, 32)
print(x, 10, 32)
```

The print statement accepts the following parameters:

```text
print(expr, base-expr, width-expr)
```

If `base-expr = -10`, then print the decimal integer as a signed value.

With this extension, you need to modify ntlang to accept a pathname as the name of the ntlang program to exexecute:

```text
$ ntlang p1.nt
```

3. (3 points) Implement ntlang with compiler support in either Rust (https://www.rust-lang.org) or Zig (https://ziglang.org). These are modern systems programming languages. You can only get credit for re-implementation in one language.
