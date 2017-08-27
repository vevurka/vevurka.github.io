---
layout: single
title: Cubic root from negative number in Python
date:   2017-06-17 12:40:02 +0100
categories: [cs, python]
tags: [programming, computer-science, python, hacking, debug-story]
excerpt: A hacky solution of how to calculate cubic root from negative number.
---

Quite recently I've encountered a weird issue - I tried to **calculate** a **cubic root** from a **negative** number in **Python**:

{% highlight python %}
    >>> import math
    >>> math.pow(8, 1/3)
    2.0
    >>> math.pow(-8, 1/3)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ValueError: math domain error
{% endhighlight %}

For **positive** numbers it works good. But it's kinda obvious
that $$-8 ^ (1/3) = (-2) * (-2) * (-2)$$. So why the *second statement* causes an `ValueError`?

I tried to the same in **JavaScript** (it's easy to find a *console* in browser :wink ):

{% highlight javascript %}
    console.log(Math.pow(8, 1/3)); // 2
    console.log(Math.pow(-8, 1/3)); // NaN
{% endhighlight %}

For both **Python** and **JavaScript**
clearly the issue is in the *underlying implementation*. I investigated this a bit, but let's take a quick glance
on the **workaround** solution first:

## Workaround solution

Anyway I needed to calculate that cubic root, so I used this workaround:

{% highlight python %}
    import math

    def cubic_pow(n):
        if n < 0:
            return -math.pow(-n, 1/3)
        else:
            return math.pow(n, 1/3)

{% endhighlight %}

That's because it's $$ -(-n) ^ \frac{1}{3} = n ^ \frac{1}{3} $$ for **negative** numbers.
It's not a big deal to write a separate function for calculating a **cubic root**
from **negative** numbers, but still why do we need to do this? Why wasn't it
implemented in a way, which allows to *calculate* such **roots**?

## Reason

I started to be curious about the reason why it works in this way. It's always good to look into the source code - 
I decided to start with sources of **Python**, but I was quite sure that I will end up with some **C** code.

### Code sources of power implementation

According to
[python-dev-guide](https://docs.python.org/devguide/) the repository of **Python** is
here: [cpython](https://github.com/python/cpython).  


I looked for some files with word `math` in the name (it would be reasonable to have **math** stuff from **math** lib in for example
`math.c`...).
I decided to start with **header** files, because they contain only **declarations**:
[mathmodule.c.h](https://github.com/python/cpython/blob/master/Modules/clinic/mathmodule.c.h):

{% highlight C %}
    // https://github.com/python/cpython/blob/master/Modules/clinic/mathmodule.c.h
    
    PyDoc_STRVAR(math_pow__doc__,
    "pow($module, x, y, /)\n"
    "--\n"
    "\n"
    "Return x**y (x to the power of y).");
    
    #define MATH_POW_METHODDEF    \
        {"pow", (PyCFunction)math_pow, METH_FASTCALL, math_pow__doc__},
    
    static PyObject *
    math_pow_impl(PyObject *module, double x, double y);
    
    static PyObject *
    math_pow(PyObject *module, PyObject **args, Py_ssize_t nargs)
    {
        PyObject *return_value = NULL;
        double x;
        double y;
    
        if (!_PyArg_ParseStack(args, nargs, "dd:pow",
            &x, &y)) {
            goto exit;
        }
        return_value = math_pow_impl(module, x, y);
    
    exit:
        return return_value;
    }

{% endhighlight %}

So I learnt I'm actually looking for `math_pow_impl` function. Let's go to the implementation I found in 
[mathmodule.c](https://github.com/python/cpython/blame/master/Modules/mathmodule.c):

{% highlight C %}

    // https://github.com/python/cpython/blame/master/Modules/mathmodule.c
    
    Return x**y (x to the power of y).
    [clinic start generated code]*/
    
    static PyObject *
    math_pow_impl(PyObject *module, double x, double y)
    /*[clinic end generated code: output=fff93e65abccd6b0 input=c26f1f6075088bfd]*/
    {
        double r;
        int odd_y;
    
        /* deal directly with IEEE specials, to cope with problems on various
           platforms whose semantics don't exactly match C99 */
        r = 0.; /* silence compiler warning */
        if (!Py_IS_FINITE(x) || !Py_IS_FINITE(y)) {
            errno = 0;
            if (Py_IS_NAN(x))
                r = y == 0. ? 1. : x; /* NaN**0 = 1 */
            else if (Py_IS_NAN(y))
                r = x == 1. ? 1. : y; /* 1**NaN = 1 */
            else if (Py_IS_INFINITY(x)) {
                odd_y = Py_IS_FINITE(y) && fmod(fabs(y), 2.0) == 1.0;
                if (y > 0.)
                    r = odd_y ? x : fabs(x);
                else if (y == 0.)
                    r = 1.;
                else /* y < 0. */
                    r = odd_y ? copysign(0., x) : 0.;
            }
            else if (Py_IS_INFINITY(y)) {
                if (fabs(x) == 1.0)
                    r = 1.;
                else if (y > 0. && fabs(x) > 1.0)
                    r = y;
                else if (y < 0. && fabs(x) < 1.0) {
                    r = -y; /* result is +inf */
                    if (x == 0.) /* 0**-inf: divide-by-zero */
                        errno = EDOM;
                }
                else
                    r = 0.;
            }
        }
        else {
            /* let libm handle finite**finite */
            errno = 0;
            PyFPE_START_PROTECT("in math_pow", return 0);
            r = pow(x, y);
            PyFPE_END_PROTECT(r);
            /* a NaN result should arise only from (-ve)**(finite
               non-integer); in this case we want to raise ValueError. */
            if (!Py_IS_FINITE(r)) {
                if (Py_IS_NAN(r)) {
                    errno = EDOM;
                }
                /*
                   an infinite result here arises either from:
                   (A) (+/-0.)**negative (-> divide-by-zero)
                   (B) overflow of x**y with x and y finite
                */
                else if (Py_IS_INFINITY(r)) {
                    if (x == 0.)
                        errno = EDOM;
                    else
                        errno = ERANGE;
                }
            }
        }
    
        if (errno && is_error(r))
            return NULL;
        else
            return PyFloat_FromDouble(r);
    }

{% endhighlight %}

After quick look at `git blame` (actually I used this nice feature on [**github**](github.com) instead of *cloning* repo). I concluded that this code
wasn't changed since years, so it's not an issue that I'm looking at current master. What is actually interesting in that code?


* there is no special case for cubic roots from negative numbers - the ValueError is thrown even if the result is a real number.

* there are `PyFPE_START_PROTECT` and `PyFPE_END_PROTECT` **macros** used around the `pow` function. Here is a description
of these **macros** from *documentation*:

        Setting up a given processor to trap IEEE-754 floating point errors currently
        requires custom code on a per-architecture basis. You may have to modify
        :mod:`fpectl` to control your particular hardware.

        Conversion of an IEEE-754 exception to a Python exception requires that the
        wrapper macros ``PyFPE_START_PROTECT`` and ``PyFPE_END_PROTECT`` be inserted
        into your code in an appropriate fashion.  Python itself has been modified to
        support the :mod:`fpectl` module, but many other codes of interest to numerical
        analysts have not.

        The :mod:`fpectl` module is not thread-safe.

For actual *calculation* of `pow` there are some functions in `mpdecimal.c`: `_mpd_qpow_int`, `_mpd_qpow_mpd` and `_mpd_qpow_uint`.
I didn't find anything interesting in them.

### What about pow in C++?

But anyway it will be nice to look it up in some **C** or **C++** reference: http://en.cppreference.com/w/cpp/numeric/math/pow
(implementation is done in **C**, because it's from `cmath` module):
 

 ===
 Error handling
 Errors are reported as specified in math_errhandling
 If base is finite and negative and exp is finite and non-integer, a domain error occurs and a range error may occur.
 If base is zero and exp is zero, a domain error may occur.
 If base is zero and exp is negative, a domain error or a pole error may occur.
 If the implementation supports IEEE floating-point arithmetic (IEC 60559),
 ===
    
Actually in **C++11** there is a special function for **cubic roots** called [`cbrt`](http://en.cppreference.com/w/cpp/numeric/math/cbrt), which can handle
a **negative** argument, but they didn't decide to use that one in `pow` in both **Python** and **JavaScript**.


