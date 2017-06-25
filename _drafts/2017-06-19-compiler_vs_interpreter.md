---
layout: single-with-chart
title: Compiler vs Interpreter
date:   2017-06-19 12:40:02 +0100
categories: [cs, basics]
tags: [compiler, computer-science]
excerpt: What is a compiler and an interpreter, also bit on their differences.
---

This post is an introduction to my attempt of writing my own language along
with its compiler. My motivation is that during my CS studies at University
I wrote an interpreter
of some really easy procedural language in Haskell. I remember I really enjoyed
that experience - I learnt a lot. Writing a compiler is much harder, because
there are plenty things one have to know - starting from BNF, semantics,
programming language theory to optimization algorithms...
As I loved these topics, it's a great
opportunity to recall them and learn plenty new stuff!

So, let's start from what is a compiler.

## Compiler

In short words compiler is a program which can translate program in one
programming language to a program in an other language.

There are many types of compilers, starting from:

* compiler - outputs machine code
* source-to-source compiler - it translates from high-level language
to another high-level language (ghc)
* just-in-time compiler - some compilation stages are deferred to be done in a
run-time (used in Java, .NET, Smalltalk)
* stage compiler - it translates code to assembly language of some (virtual
or abstract) machine (used in Erlang, Java, Python for translating to bytecode,
Prolog)

### High-level vs Low-level programming languages

You can think about low-level languages as languages which instructions are
very close to processor instructions. They have almost no abstractions
and hardware determines a way in which program is written. That's way they
are not portable between different processor architectures.
Also it's really hard to write in them, you have to care about everything -
memory management, using proper numerical codes for instructions, managing register, etc.
One of examples is assembly language.

On the other side we have high-level programming languages. They
use abstractions, even a bit of natural language, so it's much easier to
write a code. They also don't rely
on architecture so much. Code written in them may not execute so quickly
as code in low-level language - it depends on optimizations used by
a compiler. Also usually they use much more memory to run.

### Structure of a compiler

Usually compilers have two parts - frontend and backend. They can
be implemented separately - they can be almost independent of each
other. Only one condition must be fulfilled - output of a frontend
part must be a valid input to a backend part.

#### Frontend

<img src="{{ site.baseurl }}/assets/images/2017-06-19_compiler_frontend.svg"/>

<img src="{{ site.baseurl }}/assets/images/2017-06-19_compiler_backend.svg"/>


Frontend part of a compiler is mostly focused on analyzing an output
then generating an intermediate representation
of an input program.

<div>
    <div class="mermaid" style="float:left">
        graph TD
            A((character<br/> stream)) --> B[Lexical<br/>Analyzer]
            B --> |token<br/>stream| C[Syntax<br/>Analyzer]
            C --> |syntax<br/>tree|D[Semantic<br/>Analyzer]
            D --> |syntax<br/>tree|E[Intermediate<br/>Code Generator]
            E --> F((intermediate<br/>representation))
            end
    </div>

    <div>
    Parts of a frontend:
    1. Lexical analyzer - does lexical analyze, so it divides input program
    to tokens
    2. Syntax analyzer -
    3. Semantic analyzer
    4. Intermediate code generator
    </div>
</div>



#### Backend

<div class="mermaid">
    graph TB
        E((syntax tree)) --> |intermediate representation|F[Machine Independent Code Optimizer]
        F --> |intermediate representation|G[Code Generator]
        G --> |target-machine code|H[Machine Dependent Code Optimizer]
        H --> I((target-machine code))
</div>


## Interpreter

Interpreter on the other hand usually executes program line(block) by line(block).
It means it executes instructions directly and also stores the actual state
of execution (like variables).

Examples of interpreted programming languages: Lisp, PHP, Perl, MATLAB, R

### Structure of an interpreter
Lexer, Parser, Semantic Analyzer, Interpreter
<div class="mermaid">
    graph TD
        A((Character stream)) --> B[Lexical Analyzer]
        B --> |token stream| C[Syntax Analyzer]
        C --> |syntax tree|D[Semantic Analyzer]
        D --> |syntax tree|E[Interpreter]
        E --> F((Program output))
</div>




## My compiler

I'm attempting to write source-to-source compiler in Python. My
language will be also based on Python, because I don't feel ready
to design my own language yet.

Next weeks I'm going to start from writing its frontend part, so
I've to design syntax of my language. Then I'll write lexical analyzer
along with syntax and semantic analyzer. Next part will be
intermediate code generator, which will be an introduction to backend
part of my compiler.


## Compiler vs Interpreter

* changes made to program - recompile to run with new changes
* interpret -> run, compile -> only prepare for running

