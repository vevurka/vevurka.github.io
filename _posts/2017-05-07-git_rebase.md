---
layout: single
title: Don't forget about git rebase!
date:   2017-05-07 20:54:02 +0200
categories: [dsp17, git, quality]
excerpt: One of my git stories
---

I want to tell you about *one of mistakes* I made quite recently. Actually I
made *two mistakes* - the other one was about **deploying at Friday afternoon**.
Anyway everything is ok now. Let's dive in into my **git** story.

### My **git merge** without **git rebase** story

Our **git** workflow is quite common one - when we are finished with a task, we
create a **PR** (**pull request**) and add reviewers. After it's **approved**
by one of them, the author is supposed to **merge** it to `development` or `master` branch.

So I did my **PR** with a few **commits** created during few days. I didn't **squash** them
to one commit. Also I didn't **rebase** my branch to current `development` branch.

So, let's say I started my branch `feature/file3` from `development` when there were two commits:

![screenshot]({{ site.url }}/assets/images/2017-05-07/git_before.png)

I added **one commit** to my `feature/file3` branch:

![screenshot]({{ site.url }}/assets/images/2017-05-07/git_first_commit.png)

In the meantime there was another **commit** added to `development` branch created by
another author:

![screenshot]({{ site.url }}/assets/images/2017-05-07/git_another.png)

After that I added another **commit** to `feature/file3` branch and I created a **PR**:

![screenshot]({{ site.url }}/assets/images/2017-05-07/git_my_another.png)

It was **approved**, so I decided to **merge** it. And shamefully I did this without **rebasing**
to current `development` branch. Running:

    $ git checkout development
    $ git pull
    $ git checkout feature/file3
    $ git rebase -i development

could save me plenty of time, because my current changes would be on top
of **git history** after **merge**.

Instead of that I just clicked **merge** on *bitbucket* and it just did something like this:

![screenshot]({{ site.url }}/assets/images/2017-05-07/git_after_merge.png)

My **commits** (these with *file3* in title) were between other author **commits**.

We started to deploy it, but it happen that we wasn't successful for some reason. That's way
we wanted to **revert** my changes and go back to previous version of `development` branch.
But it wasn't easy, because it wasn't enough to go back to commit
`d6f1f201fe91cf21c21cba82c2173eab4ccf395a` (this with title *File 2 fixed typo*).
In situation from a picture,
it's enough to **cherry-pick** commit `c2a4ac986617d1546a18764692d7c9f0cd34aea1`
created by another author. In my real situation it was much worse, because
there were more **commits** in between. There was no point in **cherry-picking** all commits,
it was better to create a **patch**.

### Summing up

Don't be lazy - **rebase** your branch before **merging**. It will help you avoid
many issues when **revert** is needed. Also the **git history** will be much more readable -
there will be no commits between your changes from that **branch**. Also I think
it's a good idea to **squash** all commits into one before merge, because
when project becomes big it's more easy to study its history - there will be
one **commit** per *feature*.
