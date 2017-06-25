---
layout: single
title: Compiler vs Interpreter
date: 2017-06-25 20:14:02 +0200
categories: [cs, basics]
tags: [compilers, computer-science, overview]
excerpt: What is a compiler and an interpreter, also bit about their differences.
header:
    teaser: /assets/images/teasers/2017-06-25_compiler_vs_interpreter.jpg
---

This post is an *introduction* to my attempt of creating my own **programming language** 
along with its **compiler**. I'm trying to *keep it simple* and I'm skipping some in my
opinion unnecessary details to make this post also *understandable* for beginners.

My motivation for writing a compiler is that a long time ago during my 
*Computer Science* studies at university I wrote an **interpreter**
of some really *easy procedural language* (in **Haskell**). I remember I really enjoyed
that experience - I learnt a lot. Although writing a **compiler** is hard, because
there are plenty things one have to know - starting from **context-free grammars**, 
**semantics**,
**programming languages theory** to **optimization algorithms**
I really loved these topics. It's a great
opportunity to recall them and learn plenty new stuff!

So, let's start from what actually is a **compiler**.

## Compiler

In short words compiler is a **program** which can **translate** *source code* in one
programming language to a *source code* in an other language.

To describe some types of **compilers** we need to know a bit more
about types of *programming languages* in case of **abstraction** level -
they are called **high-level** and **low-level** programming languages.

### Low-level programming languages

You can think about **low-level** programming languages as languages, 
which instructions are
very close to **processor instructions**. They have almost **no abstractions**
and *hardware determines* a way in which program is written. That's why they
are **not portable** between different processor architectures.

It's really **hard to write** in them, you have to care about everything -
*memory management*, using proper *numerical codes* for instructions, 
*managing register*, etc. Also programmer needs to deeply understand
an **architecture of processor** on which he is writing code.
One of examples is **assembly language**.

For **low-level** languages there are only **compilers**, I'm not aware if
any **interpreter** exists.

### High-level programming languages

On the other side we have **high-level** programming languages. They
use **abstractions**, even a bit similar to **natural language**, so it's much easier to
write a code. They also *don't rely*
on **architecture** so much. Code written in them may *not execute* so *quickly*
as code in **low-level** languages - its execution speed depends on **optimizations** 
used by
a **compiler**. Also usually they use much **more memory** to run.

**High-level** programming languages can be both **compiled** and **interpreted**.

So, now as we know what is **high-level** and **low-level**
programming language, we can describe some types of **compilers**.

### Types of compilers

There are many types of **compilers**, starting from:

* **machine code** compiler - outputs *machine code* or some code in *low-level* programming 
language
* **source-to-source** compiler - it translates from *high-level* language
to another *high-level* language
* **just-in-time** compiler - some compilation *stages* are **deferred** to be done in 
a *run-time* (used in `Java`, `.NET`, `Smalltalk`), used for *high-level* languages
* **stage** compiler - it translates code to **assembly language** 
 or some abstract language of some (virtual
or abstract) machine (used in `Erlang`, `Java`, `Python` for translating to **bytecode**),
used for *high-level* languages

### Structure of a compiler

Usually **compilers** have two parts - **frontend** and **backend**. They can
be implemented *separately* - they are almost **independent** of each
other. Only one condition must be fulfilled - *output* of a **frontend**
part must be a *valid input* to a **backend** part.

#### Compiler frontend

Frontend part of a compiler is mostly focused on **analyzing** an output
then generating an **intermediate representation**
of an input program.
On the picture there are components used by fronted part of **compiler** with
their outputs:

![screenshot]({{ site.url }}/assets/images/2017-06-25/2017-06-25_compiler_frontend.svg){: .align-center}.

I'm not going to describe them deeply now, because each of these parts 
is quite *complicated*. I'm going to write about them in the future.

1. **Lexical Analyzer** - does *lexical analyze*, so it divides input program(*character
stream*) to **tokens** (they are just meaningful parts, 
for example each variable name is a token)
2. **Syntax Analyzer** - it *analyzes syntax*, usually it uses **context-free grammar**
definition of a language and checks if input fits to it.
3. **Semantic Analyzer** - it analyzes if meaning of input is correct also can 
checks types.
For example it can check if after every `if` there is a `boolean` expression.
4. **Intermediate Code Generator** - generates some **abstraction** of input code.

#### Compiler backend

**Backend** of a compiler is focused on **generating** source code in *target programming
language* along with some **optimizations**. Also it is responsible for making
code to fit to a desired **architecture**.

On the picture there are components used by a **backend** part of **compiler** with
their outputs:

![screenshot]({{ site.url }}/assets/images/2017-06-25/2017-06-25_compiler_backend.svg){: .align-center}.

It starts from **intermediate representation** of a code from a **frontend** part of
a **compiler**, then it does **optimizations** on this **abstract code**, **generates**
code in *target language* and again does some **optimizations**.

## Interpreter

**Interpreter** *executes* a program line/block by line/block.
It means it *executes* instructions almost **directly**. It has to store the actual state
of execution (like variables) in its memory (usually it uses some **virtual machine**).

Examples of interpreted programming languages: `Lisp`, `PHP`, `Perl`, `MATLAB`, `R`.

### Structure of an interpreter

**Interpreter** is quite *similar* to a **frontend** part of a **compiler**. Probably it's even
possible to create a **compiler** from an **interpreter**. The only difference is that
instead of generating intermediate representation of a code interpreter just **execute** it:

![screenshot]({{ site.url }}/assets/images/2017-06-25/2017-06-25_interpreter_structure.svg){: .align-center}.

We know a bit about **compilers** and **interpreters**. Let's jump into some *advantages*
and *disadvantages*
of using them!

## Compiler versus Interpreter

**Interpreter** could be almost a **frontend** part of a compiler. It means
they both do **lexical**, **syntax** and **semantic** *analyze*. But **interpreter** 
does *not* do any *optimizations*, so execution speed can be much **slower**. 
**Compiler** *prepares* code for running, **interpreter** *execute* it.

When we **make changes** to a program in *compiled* programming language,
we have to **recompile** it. It can take some time, because **compiler** needs to
go through all the code and perform **checks** and 
**optimizations** again. **Interpreter** on the other hand can pick up changes 
**immediately**, it will just *execute*
new code line by line. So it's **faster** to apply new changes in *interpreted* programming
languages than in *compiled* ones.

**Debugging** is easier in *interpreted* programming languages, because debugger doesn't have
to go through *intermediate representation* of a code. It also doesn't have to
go through all source code to show that there is for example some *syntax error*.

### Mixed approach 

As there are *important* pros and cons of using both **interpreter** (**faster** in development) 
and **compiler** (**optimizations** applied to code) programming languages designers came up
with mixed approach to use advantages of them both.

This approach is used in for example: `Java`, `C#`, `Python` or `Ruby`. 
In these languages source code is **compiled** to **bytecode** (or **object-code**), which
can be executed by an **interpreter**. In this approach **optimizations**
are applied during compilation, but we can also execute code *quicker* by using 
**interpreter**.

## My plans for a compiler

I'm attempting to write **source-to-source** compiler in `Python`. My
language will be also based on `Python`, because I don't feel ready
to design my own language from scratch (yet!).

In the next weeks I'm going to start from writing its **frontend** part, so
I will have to design **syntax** of my language. Then I'll write **lexical analyzer**
along with **syntax** and **semantic** analyzer. Next part will be
**intermediate code generator**, which will be an *introduction* to **backend**
part of my compiler.

## Summing up

I hope now you know a bit about what is a differences between **interpreter**
and **compiler** and what they are doing. Also what is a difference between
low-level and high-level languages. In next post I'm going to go through some necessary
**must-know** stuff about programming languages, which is necessary to make decisions when
designing a *completely new* programming language.

Having some thoughts? Leave a comment!
