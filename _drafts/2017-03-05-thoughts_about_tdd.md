---
layout: single
title:  Some thoughts about TDD
date:   2017-03-05 12:20:01 +0100
categories: [dsp17, quality]
excerpt: TDD or not TDD
---

A few months ago one of my colleagues did an awesome training in TDD. I had heard about TDD before - mostly bad opinions, and I was quite sure that I know
enough to just reject this methodology. This training changed things a lot. It's much better to listen about TDD from someone who actually use it.

He said that he doesn't want to convince anybody that it is the best methodology and that for some people it can just will not work, but for him it does.
Actually this statement convinced me to at least try and do my best during the practical part of training.

### A bit about TDD
TDD(test-driven development) is a software development process in which we repeat short cycle:

* write an unit test to method or function - test will fail
* write or fix a method - test should pass
* refactor - test should pass

What is not obvious - the most important part is refactoring. It's when all the magic happens and still we can be sure that we
didn't brake anything.

### Subjective benefits of using TDD
* I code much faster (even if I don't count test code as a code)
* I'm more sure that my code is doing what it is supposed to do
* I think about corner cases earlier
* I think more about design - I don't rush into coding

### Some other notes and tips
* TDD doesn't replace functional tests - we can end up like this:

![](http://x3.wykop.pl/cdn/c3201142/comment_jP2f4B7YsFrU7ixhLBJieJ7QTGNPFjTR.gif)

* Sometimes with a legacy code it's hard to fix all tests - it's good to comment all the tests except one and work only on one not working case.
* It's good to finish a day with one not passing test - next day we know from where to start.
* Intention is very important in TDD, sometimes you have to actually fix tests and then fix a code.
