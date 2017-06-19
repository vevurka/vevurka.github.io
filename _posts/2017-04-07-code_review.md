---
layout: single
title: Code review
date:   2017-04-07 20:10:02 +0200
categories: [dsp17, quality]
excerpt: How to start with code review?
tags: [code-review, quality, software-development, how-to, overview]
---

## What is **code review**?

**Code review** is an *examination of code* - usually it should be done by
software developer and his team mate after finishing coding task. It means
that after you finish some *feature*, *bug fix* or *refactoring*, you should **reread**
your code and check if you are satisfied with its **quality**. After that **some other developer**
from your team should read your code and give you his **feedback** and **remarks**.

## Benefits of **code review**

For me **code review** is very important part of creating software. There are plenty
reasons to have **code review** practice in your team:

* helps with **catching bugs** or other issues (like security or performance)
* helps with improving **code quality**
* makes programmer to **care more** about code
* helps with **sharing knowledge**
* great **learning opportunity**

The only reason not to do **code review** is that it's **time consuming**. *But when
looking at bigger picture is it so?* Actually, when someone double checks your work, there
is a chance, he will catch bugs at earlier stage, so it *might* (from my experience, definitely **will**)
**save time** spent on **bug fixing** later.

## How to learn **code reviewing**?

There are plenty of sites on which you can read about *best practices*, but usually
they cover only *cultural* part of **code review** - for example: don't
do code review for more than hour, divide code reviewing, etc. Resources on
tricky code reviews are harder to find.

Almost in all resources you can find
an information that you should have your own **code review checklist**, but suggestions
are quite overwhelming. Actually it's quite hard to design a generic **checklist**,
because some conventions depends on used *technologies*, *paradigms*, even *environments*, etc.
So probably it's better to have one, based on your experience.

As with everything - to learn how to do **code review** you should do more **code review**,
so don't waste your time too much on reading resources, just start doing it.

## How to start with **code review**?

I think for a start it's better to follow your **common sense** rather than going with a **checklists**
found in the Internet, because there is too much stuff in them. It's good to read some, though, to get some clues
on what can go wrong. To start you should know what issue is solved by this code, so read about it
in your **issue tracker** (hopefully you have some) or in **commit message** (hopefully developer wrote it - if not,
you have your first remark). At first just
**try to read** a code and try to imagine how it works. After you understand
that code, **reread** it and think about:

* quality (unit tests, naming patterns, conventions, formatting, etc)
* performance
* security
* design patterns used in this solution
* architecture of this solution
* corner cases

Of course, there is a possibility that in code you are reviewing, there is some stuff you don't know.
But that's **great news** for you! You can use this as **learning opportunity** - read a bit about it in the
*Internet* or ask your colleague about it.

You should be prepared for making mistakes, because it's not easy to catch bugs by reading a code.
After some practice you will get better in imagining, how code works. **Self-review** of your code
might help with gaining experience faster.

After you gain more experience it's good time to create your **own checklist** - it should be
based on your experience. I've my own in my head,
but maybe it's time to put it somewhere, because it started to be quite complicated!

### What to do when you don't understand **code** you are reviewing?

Usually people feel quite bad when they don't understand things, probably you are no exception. That's way
it's so hard to admit it. But when you don't understand something, find a **courage** and **ask** your
colleague how it works - this is the best option for you. Maybe this code should be **simplified**,
because it's too *complicated*? Or there is a *bug* or *flaw*
in this code and it just doesn't work as expected? Or it's not documented by **doc strings** or **unit tests**
enough?
There are plenty reasons why sometimes it's hard to understand - it is not necessarily your lack of knowledge.
That's way - don't be scared of asking!

There might be a discussion after asking a question and both sides can gain more experience from that. So use
this as **learning opportunity** and don't treat code and code review personally.

### How to inform about **flaws** in code you reviewed?

Remember, we are all human - we make mistakes, whenever we like it or not. So, try to give your remarks in
such a way as you would like to receive feedback. I usually use phrases like this:

> I think it's better to ... .
> For me it would be more readable if ... .
> Please add ... .
> Isn't it better to do this: ... like this: ...?

Sometimes I also ask **why** questions, when I'm not sure if something is a flaw or I lack of understanding.
Bear in mind that you don't judge a person, but a code - don't treat it personally.

## Final thoughts

**Code review** is very important and there are only benefits from it. It might be quite hard to start,
but it's possible to gain more practice when **self-reviewing**. **Asking questions** is a natural part
of code reviewing, it's also a way of giving feedback. Next week I'll try to describe some failures from which
I've learned on what I should pay attention more.

Let me know your thoughts in comments!


