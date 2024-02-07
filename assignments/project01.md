---
layout: default
title: Project01
nav_order: 3
parent: Assignments
permalink: /assignments/project01
published: false
---

# NTlang - Number Tool Language

## Due Mon Feb 12th by 11:59pm in your Project01 GitHub repo

## Links

Tests: [https://github.com/USF-CS631-S24/tests](https://github.com/USF-CS631-S24/tests)

Autograder: [https://github.com/phpeterson-usf/autograder](https://github.com/phpeterson-usf/autograder)

## Background

The goal of this project is to implement a command line program that can interpret simple NTlang expressions for working with different number bases and bit widths. This tool will also be handy later on when working on other projects to do various types of number conversions and bit twiddling.

For interactive grading you will also demonstrate that you can work from the command line, run a local Ubuntu RISC-V environment, work remotely on the BeagleV machines, use ssh with ssh keys and no password, use console-based editor, and access GitHub from the command line using ssh keys.


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


3. Valid inputs for the number base are -b 2, -b 10 (default if -b not given), and -b 16

4. Valid inputs for the width are -w 4, -w 8, -w 16, and -w 32 (default if -w not given)

5. Print unsigned ints with -u (this only has meaning for -b 10)

6. You will write the scanner and parser yourself, without using C strtok() or scanf() or compiler generators such as lex, yacc, bison, antlr, etc.

7. You will write the base conversion tools yourself without using C printf("%d", i), printf("%x", i), or strtol() That is, you must write conversion functions that convert binary forms into strings, then use you can only use printf("%s", str) to output the converted string.

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
>>   logical shift right
>-   arithmetic shift right (note that C does not have this operator)
<<   logical shift left
~    bitwise not
&    bitwise and
|    bitwise or
^    bitwise xor
```

## Interpreter

1. Your interpreter will walk the AST depth-first, evaluating the expressions defined by the nodes, and printing the results.

2. You should store the intermediate results in a C uint32_t (unsigned int), to make conversion to binary or hexadecimal easy.

3. The width parameter controls how many bits wide the output should be.

##Grading

1. Grading will have two components: automated and interactive.

2. For interactive grading, we will schedule 1:1 meetings with the instructor or TAs using a Google Sheet published on Campuswire. You should be prepared to share your screen and answer questions about your design and implementation. 
Interactive grading counts for 40% of the project grade, as follows:
10% Question 1 (examples given in lecture)
10% Question 2
10% Question 3
10% Code quality
The code quality rubric is:
+2 for consistent spacing and indentation
+2 for consistent naming and commenting
+2 for no commented-out ("dead") code
+2 for no redundant or overly complicated code
+2 for a clean repo, that is no build products, extra files, etc.
Automated grading counts for 60% of your project grade
We will use the autograder (grade) tool to grade correctness of your project. You can run it yourself on your local repository to see how your code scores. Instructions are in the README.
Your project must be generated by a Makefile, and the executable must be called project02