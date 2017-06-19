---
layout: single
title:  Some thoughts about TDD
date:   2017-03-08 16:43:01 +0100
categories: [dsp17, quality, python]
tags: [tdd, testing, quality, software-development]
excerpt: TDD or not TDD
---

A few months ago one of my colleagues did an awesome training in one of software development methodologies: **TDD**.
I had heard about TDD before - mostly bad opinions, and I was quite sure that I know enough to just reject
this methodology. This training changed things a lot. It's much better to listen about TDD from someone who actually
use it and who doesn't try to convince anybody that it's the best programming methodology. For some people it will
not work, because we have different ways of thinking, etc. So, as I wasn't forced to use TDD during training I
decided to do my best on practical part. My previous experiences with people using TDD
was that they were like some blindfolded people, who always try to convince other that this is the only way.

## A bit about TDD
**TDD (test-driven development)** is a software development methodology in which we repeat short cycle:

* **write an unit test to method or function** - test will fail (code for this method/function may not exist yet).
* **write or fix a method** - all tests should pass
* **refactor** - all tests should pass

What is not obvious - the most important part is refactoring. It's when all magic happens and still we
can easily check previous parts of a feature by running our tests.


**TDD** is not about testing - it's about a design. It allows to design code to be testable, self-documented, scalable
and easier to maintain. In this approach developer is forced to think about relations with other objects, interfaces at
earlier stage. Also it's a way of doing software in an incremental way, so when we think on this on higher level - it
will work as in agile methodologies like **Scrum** but on a lower level.

## About tests in TDD (and in general)

### How should unit tests look like?
Tests should be **as small as possible** - they are simpler to debug if something is wrong, they can also be a part of
development documentation. Ideally the should have a few lines of code ended with one assertion.

They should have a **meaningful names**: `test_get_cat_throws_exception_for_name_longer_than_60()` is better than
`test_get_cat_error()`. Even if you want to change a validation for cat name you know what you should fix and what
is a current value.

It's good to name expected values with **expected** prefix. Then you can write an assertion
like this: `assertEquals(expected_value, value)` and it's quite obvious what's going on. On the other hand it's
not important to avoid constant values: `assertEquals(60, value)` is also a good example of assertion.


All the tests should be **independent** - result of one test can't depend on previous tests, so better to not have any
side-effects in them. Usually testing frameworks allow to create some methods which are run before and after each test
case for example:

* *python*: `unittest.TestCase` we can implement `setUp()` and `tearDown()` methods
* *GO*: `Ginkgo` framework we can use `BeforeEach` and `AfterEach` methods
* *Erlang*: `eunit` framework has `Setup` and `Cleanup` fixtures

In writing tests it's good to follow some rules from functional and object-oriented paradigms:

* **SRP (Single Responsibility Principle)** from SOLID rules - it means you should test only one thing in your test.
* **no side effects** as it is in functional paradigm - it's good to mock all external services and states. Mocks are
great for simulating behaviour of other objects which aren't in scope of test. There are many mocking libraries,
for example: in *python* - `unittest.mock.MagicMock`, in *erlang* - `meck`, in *GO* - `GoMock`.


### Tips on writing tests
* It's good to finish a day with one not passing test - next day you know where to start.
* When for some reason several tests aren't passing (and it's not a matter of an error in testing environment) it's good
to comment all not-passing tests except one and focus on fixing only that one.

### Subjective benefits of using TDD

* I code much faster (even if I don't count test code as a code) - I'm designing my code when I write tests, because
I've to think how to divide my new feature into testable small functions or methods. On every part of functionality
I've to know what is an input and output.
* I'm more sure that my code is doing what it is supposed to do, because my small tests are helping me with
self-checking.
* I think about corner cases earlier, because I've to think about possible input and outputs at very early stage.
* I think more about design - I don't rush into actual coding, because sometimes it's harder to start from tests.
* My code is testable, because I actually write tests.

#### Some other notes and tips
* TDD is good for frame of mind - you can see good results (passing tests) faster.
* TDD doesn't replace functional or integration tests - we can end up like this:


<img src="http://x3.wykop.pl/cdn/c3201142/comment_jP2f4B7YsFrU7ixhLBJieJ7QTGNPFjTR.gif" width="400" height="400"
 style="display: block; margin-left: auto; margin-right: auto;"
 />


* Intention is very important in TDD, sometimes you have to actually fix tests and then fix a code.
It's important to don't approach writing tests to match written code - if we have an intention to not do that,
then it's ok.
