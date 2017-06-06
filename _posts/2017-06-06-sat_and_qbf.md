---
layout: single
title: Two important decision problems
date:   2017-06-06 21:09:02 +0200
categories: [dsp17, cs]
excerpt: Overview on SAT and QBF
---

Last time I wrote some stuff on 
[P versus NP]({{ site.baseurl }}{% link _posts/2017-05-28-np_vs_p_overview.md %}).
This time I want to describe a bit some important **decision problems** - $$SAT$$
and $$QBF$$. 

## Important decision problems

Short reminder on what is **decision problem**:

**Decision problems** are these which have an *output*: **YES** or **NO**.
{: .notice--primary}

### $$SAT$$

**Boolean satisfiability problem** ($$SAT$$) is about checking if given 
**propositional logical
formula** is **satisfiable**.
So for example when having formula like this: $$(x \lor y \lor z ) \land (x \lor \lnot y \lor \lnot z)$$ we want to know
if we can assign to all variables values from $$(TRUE, FALSE)$$ to make this formula to **evaluate** to $$TRUE$$. 
In this case to
satisfy this formula all variables can be set to $$TRUE$$.

It's proven that $$SAT$$ is $$NP$$-complete (so every problem
which is in $$NP$$ is at least *less hard* than $$SAT$$) and it's not my point to prove
it here. If you want to find the prove - look for **Cook–Levin theorem**.

Of course there are some variants of $$SAT$$ problems with some *additional constraints*,
which are
in $$P$$ - for example $$2-SAT$$ (when the number of *literals* in *clause* is limited to
$$2$$).

Because $$SAT$$ is $$NP$$-complete you can **reduce** any problem from $$NP$$ *set* 
to $$SAT$$. This **reduction** can be used to get a $$SAT$$ **representation** of your 
problem, which makes it easier to describe this problem 
using *programming language* or *formal logic*. That's why there are
plenty of $$SAT$$-solvers.

### $$QBF$$

**Quantified Boolean formula** problem ($$QBF$$) is a *generalization* of $$SAT$$ - 
we just add
**quantifiers** (*for all* - $$ \forall $$ or *exists* - $$ \exists $$) to 
given **formulas**, for example:
$$ \forall x\  \exists y\  \exists z\  ((x  \lor z) \land y) $$. 
Then we want to know if there exists an **evaluation** of variables which **satisfies** 
the **formula**.

$$QBF$$ is a $$PSPACE$$-complete problem, so it seems to be **even harder** 
than $$SAT$$.
This problem is so important, because it's all about **reasoning**. If we were able to 
have an algorithm which solves $$QBF$$ in *reasonable time*, 
then it would be possible
to **prove theorems automatically** (it is somehow possible, but in *limited* way, for
example using **CoQ**).
There exist some $$QBF$$-solvers, which do some heuristics on solution.
They are used in AI, but also in automatic theorem provers.

### So why $$P$$ vs $$NP$$ matters?

There are some consequences that we cannot solve $$SAT$$ and $$QBF$$ in **reasonable**
time. For example **Artificial Intelligence** still is based on **heuristic algorithms**, 
so they can have not
*deterministic* output.

The other thing is that when doing software development it's nice to know
problems which are in $$NP$$, $$PSPACE$$ or even higher **complexity classes**. 
If you know it, you will search of some **good heuristics** instead of 
writing **non-optimal solution**.



[P vs NP - further reading](https://www.win.tue.nl/~gwoegi/P-versus-NP.htm)
