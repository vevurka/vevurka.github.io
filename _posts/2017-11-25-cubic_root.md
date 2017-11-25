---
layout: single
title: Cubic root from negative number in Python
date:   2017-11-25 22:20:02 +0200
categories: [cs, python]
tags: [programming, computer-science, python, hacking]
excerpt: Some troubles with math.
header:
    teaser: /assets/images/teasers/2017-11-25_cubic_root.jpg
---

Quite recently I've encountered a weird issue. I tried to calculate a **cubic root** from a **negative number** in **Python**:

{% highlight python %}
    $ python3
    Python 3.5.3 (default, May 10 2017, 15:05:55) 
    [GCC 6.3.1 20161221 (Red Hat 6.3.1-1)] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import math
    >>> math.pow(8, 1/3)
    2.0
    >>> math.pow(-8, 1/3)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ValueError: math domain error
 
{% endhighlight %}

But it's kind of *obvious* that $$-8 = (-2)^{1/3} = (-2) * (-2) * (-2)$$. So why is this error?

Next thing I tried to the same in **JavaScript** (it's easy to find a console in browser...):

{% highlight javascript %}
    console.log(Math.pow(8, 1/3)); // 2
    console.log(Math.pow(-8, 1/3)); // NaN
{% endhighlight %}

It seems that it worked the same, clearly the issue was in the underlying implementation. But first quick glance 
on the workaround solution:

## Workaround solution

I needed to calculate that **cubic root** and I knew that there has to be *at least one real solution*, so I used this workaround:

{% highlight python %}
    
    def cubic_pow(n):
        if n < 0:
            return -(-n) ** (1.0 / 3.0)
        else:
            return n ** (1.0 / 3.0)

{% endhighlight %}

That's because it's $$ -(-n) ^ \frac{1}{3} = n ^ \frac{1}{3} $$ for **negative** numbers.

Note that there are usually $$ 3 $$ roots of **cubic root**: two *complex* ones and one *real*.
{: .notice--warning}

## Reason of issues with cubic root

I was wondering about the reason behind that behaviour. To find out I decided to check the documentation 
of **Python** and source code of **Python** implementation and see where it can lead me next. 

### Power function in Python documentation and source code

In [python doc](https://docs.python.org/3/library/math.html#math.pow) it's clearly written:
    
    If both x and y are finite, `x` is negative, and `y` is not an integer then `pow(x, y)` is undefined, and raises `ValueError`.
    
The source code of **cpython** only confirmed that [mathmodule.c.h](https://github.com/python/cpython/blob/master/Modules/clinic/mathmodule.c.h)
behaviour. But there is no explicit reason why mentioned. It's also written:

    Exceptional cases follow Annex ‘F’ of the C99 standard as far as possible.
    
It seemed that **C99** standard is a good lead for next steps.

### Annex F of the C99 standard

Here is a link to the standard: [C99 standard](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf)
In F.9.4.4 it's clearly written:

    pow(x, y) returns a NaN and raises the ‘‘invalid’’ floating-point exception for
    finite x < 0 and finite non-integer y.
    
So it seems that when designing **floating-point arithmetic** they just made a decision to raise exception when it's
not obvious when the result of **power** function will be *complex* or not. I think it might be good decision - the
**C99 standard** is already quite complicated and adding more corner cases there could create a nightmare for 
people who wants to implement **math** functions using that standard. And finding a workaround is really easy in 
cases when this is obvious that **real** roots exist.

This is also complaint with [IEEE Standard 754](http://steve.hollasch.net/cgindex/coding/ieeefloat.html).

### What about pow in C++?

From **C++** reference [math pow](http://en.cppreference.com/w/cpp/numeric/math/pow):
 
    If base is finite and negative and exp is finite and non-integer, a domain 
        error occurs and a range error may occur.
    (...)
    If the implementation supports IEEE floating-point arithmetic (IEC 60559)
    
Actually in **C++11** and later ones there is a special function for 
cubic roots [cbrt](http://en.cppreference.com/w/cpp/numeric/math/cbrt), which can handle
the **negative** argument, but they didn't decide to use that one in **cpython** 
implementation of **power** function.
