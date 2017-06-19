---
layout: single
title: Mistakes in code review
date:   2017-04-17 23:02:02 +0200
categories: [dsp17, quality]
excerpt: They happen all the time...
tags: [code-review, quality, software-development, mistakes, how-to]
header:
    teaser: /assets/images/cat_computer.jpg
---

It's obvious that inexperienced software developer will make more mistakes, but even the best engineers
make them. Mistakes in code reviewing happen all the time! When we **push** our change
(for example by creating **pull request**) and *reviewer* requests changes, it usually
means that we'd failed in **self-reviewing**. Or when our code
had been **approved** by *reviewer* and after *merging* it, it caused some **bug**, it means that both
*reviewer* and we'd failed in **reviewing**. But really, hard to blame anyone, it just should be
a lesson for both. In this post I want to write about some lessons I've learnt.

## Mistakes in code review

From my experience it seems that mistakes in code review usually come not from mistakes from
reading changed code, but from **not reading code that wasn't changed**. By this I mean that it's important to check
**the context** of reviewed code and other relevant places. For example
the author **forgot** to change the name of a *method* in all necessary places - **commit** looks good, but
the code doesn't work.

Also viewing the **bigger picture** is important - maybe achieving the same purpose
could be done in an easier way? Sometimes it's good to take a time and think how
you could **design** the solution by yourself. Of course, your design can be all wrong, but
in that case you will **learn more**. And of course, you don't have the same amount of time
as an author to think about all details.

When reviewing it's also hard to come up with code which
should be written but it wasn't, for example **not all exceptions** where *caught* or someone
**forgot to commit** a new file with whole functionality (yep, this happens!).

### Performance when querying database
One of standard mistakes is **getting records** from **database** or other resource, using **loop** instead
of having one query to get all resources, for example:

{% highlight python %}
    users = []
    for user_id in user_ids_list:
        users.append(User.objects.get(id=user_id))
{% endhighlight %}

Here for all `ids` in `user_ids_list` we perform a query like this:
{% highlight sql %}
    SELECT * FROM USERS WHERE ID = ?;
{% endhighlight %}
So we have to access database **many times** (length of `user_ids_list` actually) to get all users we want.

It could be fixed like this:
{% highlight python %}
    users = User.objects.filter(id__in=user_ids_list)
{% endhighlight %}
*(Actually in **Python** it will return a QuerySet instead of a list, but here it's not relevant).*

This is an example of using `filter` method in **Django ORM**, but I'm sure that every decent **ORM**
contains a **similar method**. This will run a query like this:

{% highlight sql %}
    SELECT * FROM USERS WHERE ID IN ?;
{% endhighlight %}
So it will access database **one time** and get all records we wanted.

Learn your **ORM** - it should contain methods or functions to perform more complicated queries instead
of relying on *loops*. You should know how to perform **filtering**, **grouping** and getting **distinct**
records from database with corresponding functions. If you don't use any **ORM** be
sure to use words like `IN`, `GROUP BY`, `DISTINCT` in your queries.
{: .notice--danger}

### Reviewing refactored code when rename was performed

Reviewing code which was *refactored* is sometimes quite hard.
Usually **refactor** option in **IDEs** perform well when **changing name** of *method* or *class*.
When using **static typed** or **compiled** language, it's usually quite obvious when
*renaming* wasn't performed well, but still when code reviewing you
should pay attention. There can be some situations where
*method* or *class* name is put into *non-source file* (like some `xml` or `config`), for example
in performing *dependency injections* in
**Java's Spring** framework. So it's good practice to check if after change,
there is no old name in code - just use `grep` or a **search** option in your **IDE**.

Code reviewing when name was changed becomes harder when two *classes*, *methods* or *variables*
**switched names**. Then `grep` will not be so helpful and you have carefully go through **diff**.

When using frameworks not all code has to be in source files.
{: .notice--danger}

### Tests which test nothing

The tests are as **important** as the code, so you also should check them carefully. Each **test name** should indicate
*what is tested*, so if you are unsure, maybe it's time to ask an author. Also there should be at least
**one assertion** in each test.

Sometimes everything looks good, like here:

{% highlight python %}

    def test_get_names_every_user_has_name(self):
        users = self.service.get_users()
        for user in users:
            self.assertIsNotNone(user.name)
{% endhighlight %}

It happens that this test tests exactly **nothing**. What will happen if `self.service.get_users()` returns
an **empty collection**? No assertion will be checked, so what's the point of this test?
There is an easy fix:

{% highlight python %}
    def test_get_names_every_user_has_name(self):
        users = self.service.get_users()
        self.assertEquals(len(users), 10) # or self.assertNotNone(users)
        for user in users:
            self.assertIsNotNone(user.name)
{% endhighlight %}


When doing assertions in a loop it's important to check if looped collections has an expected size.
{: .notice--danger}

## Summing up

**Code review** is an *art* and it takes time to become experienced enough to become proficient in it. There are
plenty *details*, *corner cases*, things which should be well thought out. On the other side there
is **time** - we don't want to spend more time than the author...
The most beautiful thing about checking ones code is whenever you're experienced or not, you can still **learn many things** from **reading code**.