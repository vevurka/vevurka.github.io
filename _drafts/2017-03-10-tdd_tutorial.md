---
layout: single
title: TDD quasi tutorial
date:   2017-04-15 12:40:02 +0100
categories: [dsp17, quality]
excerpt: How to start with TDD?
---

## How to start with TDD? (*quasi tutorial*)

For me it was hard to start using TDD. I had many questions, for example: how big steps should I do when adding
new cases? How to write a test? What should be in it? Aren't my tests too small? Aren't my tests too big?
On the other side it's so easy to write first test... It's strange for me now that I had any doubts.

Let's say we have to implement **FizzBuzz** which is a common problem asked on
interviews. Description is as follows:

> input
: `number n`
>
> output
: None, but function should print *Fizz* when `n` is divisable by `3` and *Buzz* when `n` is divisable by `5`.

This problem is really easy to solve, without special thought it looks like we will need a few `ifs` to resolve it.
That's way it's a good problem to practice **TDD**.

So my first test probably will look like this:
***** Test which will fail
***** Output





### TODO?

When working on a bug I usually try to write a test which reproduces that bug.