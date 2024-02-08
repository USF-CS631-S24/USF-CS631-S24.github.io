---
layout: default
title: Project01
nav_order: 3
parent: Assignments
permalink: /assignments/project01
published: true
---

# NTlang - Number Tool Language

## Due Mon Feb 12th by 11:59pm in your Project01 GitHub repo

## Interactive Grading on Tue Feb 13th. No Lectures. You will signup on a provided Google signup sheet.


## Links

Tests: [https://github.com/USF-CS631-S24/tests](https://github.com/USF-CS631-S24/tests)

Autograder: [https://github.com/phpeterson-usf/autograder](https://github.com/phpeterson-usf/autograder)

## Background

The goal of this project is to implement a command line program that can interpret simple NTlang expressions for working with different number bases and bit widths. This tool will also be handy later on when working on other projects to do various types of number conversions and bit twiddling.

For interactive grading you will also demonstrate that you can work from the command line, run a local Ubuntu RISC-V environment, work remotely on the BeagleV machines, use ssh with ssh keys and no passwords, access GitHub from the command line using ssh keys, use a console-based editor, and use gdb for debugging.


## Given

1. You will be given a working reference implementation of the scanner and parser we have developed in the labs. 

2. You will expand this implementation to match the EBNF given below, and provide your own bitwise operations and base conversions.

## Requirements

1. You will use the techniques we've learned for scanning, parsing, and number manipulation to build an interpreter for a little language with the EBNF grammars given below.


2. Your interpreter will take an expression, a number base, and a width on the command line, and print its output to stdout like this:

```text
$ ./project01 -e "1 + 1"
2
$ ./project01 -e "10" -b 16
0x0000000A
$ ./project01 -e "10 + 1"
11
$ ./project01 -e "10 + 1" -b 16
0x0000000B

$ ./project01 -e "10" -b 16 -w 16
0x000A
$ ./project01 -e "10" -b 2
0b00000000000000000000000000001010
$ ./project01 -e "10" -b 2 -w 4
0b1010
$ ./project01 -e "0x0A" -b 10
10
$ ./project01 -e "0x0A" -b 2 -w 8
0b00001010

$ ./project01 -e "((1 + 1) * 1)" -b 16 -w 16
0x0002
$ ./project01 -e "((1 + 1) * 1)" -b 2 -w 8
0b00000010

$ ./project01 -e "(1 << 16)" -b 10 -w 32
65536
$ ./project01 -e "(1 << 16)" -b 10 -w 16
0
$ ./project01 -e "(1 << 16)" -b 16 -w 32
0x00010000

$ ./project01 -b 10 -w 8 -e "(2 * (0b1111 & 0b1010))"
20

$ ./project01 -b 10 -w 8 -e "0b00001000"
8
$ ./project01 -b 10 -w 4 -e "0b00001000"
-8
$ ./project01 -b 10 -u -w 4 -e "0b00001000"
8

$ ./project01 -b 10 -w 8 -e "-128 >- 2"
-32
$ ./project01 -e "(((((~((-(2 * ((1023 + 1) / 4)) >- 2) << 8)) >> 10) ^ 0b01110) & 0x1E) | ~(0b10000))"
-1

$ ./project02 -e "0xffffffff"
-1
$ ./project02 -e "0xffffffff1"
overflows uint32_t: ffffffff1
$ ./project02 -e "0x000000000ffffffff"
-1
```


3. Valid inputs for the number base are `-b 2`, `-b 10` (default if `-b` not given), and `-b 16`

4. Valid inputs for the width are `-w 4`, `-w 8`, `-w 16`, and `-w 32` (default if `-w` not given)

5. Print unsigned ints with `-u` (this only has meaning for `-b 10`)

6. You will write the scanner and parser yourself, without using C `strtok()` or `scanf()` or compiler generators such as lex, yacc, bison, antlr, etc.

7. You will write the base conversion tools yourself without using C `printf("%d", i)`, `printf("%x", i)`, or `strtol()` That is, you must write conversion functions that convert binary forms into strings, then use you can only use `printf("%s", str)` to output the converted string.

## Scanner

1. Your scanner will read the expression from the command line and create a data structure of tokens. 

2. The EBNF grammar (micro syntax) for the scanner is:

```text
whitespace  ::=  (' ' | '\t') (' ' | '\t')*

tokenlist  ::= (token)*
token      ::= intlit | hexlit | binlit | symbol
symbol     ::= '+' | '-' | '*' | '/' | '>>' | '>-' | '<<' | '~' | '&' | '|' | '^'
intlit     ::= digit (digit)*
hexlit     ::= '0x' | hexdigit (hexdigit)*
binlit     ::= '0b' ['0', '1'] (['0', '1'])*
hexdigit   ::= 'a' | ... | 'f' | 'A' | ... | 'F' | digit
digit      ::= '0' | ... | '9'

# Ignore
whitespace ::= ' ' | '\t' (' ' | '\t')*
```

## Parser

1. Your parser will generate a tree (AST) from the scanned tokens that represent valid NTLang programs.

2. The parser adds the following elements to the EBNF grammar:

```text
# Parser

program    ::= expression EOT

expression ::= operand (operator operand)*

operand    ::= intlit
             | hexlit
             | binlit
             | '-' operand
             | '~' operand
             | '(' expression ')'

operator   ::= '+' | '-' | '*' | '/' | '>>' | '<<' | '&' | '|' | '^' | '>-' | '~'
```
The bitwise operators do the same thing as their counterparts in C:

```text
Binary operators:

>>   logical shift right
>-   arithmetic shift right (note that C does not have this operator)
<<   logical shift left
&    bitwise and
|    bitwise or
^    bitwise xor

Unary operators:

~    bitwise not
```

## Interpreter

1. Your interpreter will walk the AST depth-first, evaluating the expressions defined by the nodes, and printing the results.

2. You should store the intermediate results in a C `uint32_t (unsigned int)`, to make conversion to binary or hexadecimal easy.

3. When evaluating, you will need to typecast `uint32_t` some values into a signed value for some operators, such as `-` and `>-`.

4. The width parameter controls how many bits wide the output should be. This only applies to `-b 2` and `-b 16`.

## Grading

1. Grading will have two components: automated and interactive.

2. (80%) Automated tests.

3. (20%) For interactive grading, we will schedule 1:1 meetings with the instructor or TAs using a Google Sheet published on Campuswire. You should be prepared to share your screen and answer questions about your design and implementation. You will need to demonstrate the following:
  - Run the Ubuntu RISC-V image locally on your machine
  - Acccess the remote CS Beagle-V machines
  - Use ssh keys to login to both environments without a password
  - Access GitHub in both environments using ssh keys
  - Use a console-based editor from the shell (i.e., micro)
  - Use gdb for debugging
  - Clone your project01 repo
  - Run the autograder test
  - Explan your code 

4. Code Quality - These will be deduction from your overall points that can be earned back. We are looking for:
  - Consistent spacing and indentation
  - Consistent naming and commenting
  - No commented-out ("dead") code
  - No redundant or overly complicated code
  - A clean repo, that is no build products, extra files, etc.

## Extra Credit

**Notes**

- You can only earn extra credit if you complete all of the project01 requirements.
- You can turn in extra credit by **Fri Feb 16 at 11:59pm**.
- In your repo provide instructions on how we can build and run you extra credit versions and provide tests that show it works.
- The extra credit problem are long and not worth much in terms of points. They are designed to give you some additional to explore and learn beyond the main goals of the project.

**Extra Credit Problems**

1. (3 points) Extend NTLang to support variables and statements. Specifically add support for assignment and a print statement. For example consider the following new NTLang programs:

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

2. (1 point) Extend extra credit item (1) to enter a REPL if no pathname is given:

```
$ ntlang
nt> 1 + 3
4
nt> x = 7
nt> print(x, 16, 32)
0x00000007
```

3. (3 points) Implement ntlang in either Rust (https://www.rust-lang.org) or Zig (https://ziglang.org). These are modern systems programming languages. You can only get credit for re-implementation in one language.
