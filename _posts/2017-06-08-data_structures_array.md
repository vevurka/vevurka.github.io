---
layout: single
title: Basic data structures - array
date:   2017-06-08 23:48:02 +0200
categories: [cs, basics, algorithms]
tags: [programming, overview, computer-science, algorithms, data-structures]
excerpt: What is an array? Examples of arrays in some programming languages.
header:
    teaser: /assets/images/cat_computer.jpg
---

Here is very basic description on what is an **array**. I tried to describe
things as simple as possible. This post is meant to be for
people who just started programming.

# Array

**Intuitively** you can think that **array** is a shelf with multiple
enumerated **drawers**, something like this:

![screenshot]({{ site.url }}/assets/images/2017-06-08/2017-06-08_array.jpg){: .align-center}.

Yep, it's that simple. But there is a catch - you can put **only
one** thing into each drawer...

Speaking a bit more **formally**:

**Array** is a *collection* of elements ordered using **indexes**.
{: .notice--success}

## What I can do with an array?

Let's start from *what you can do with shelf?*. You can:

* **create** it (or buy one)
* **get** thing from a drawer
* change (**set**) thing which is in some drawer (but remember to keep **only one**
thing there!)
* check **how many** drawers you have
* etc...

The same with **arrays** - you can **create**, **get** and **set** elements.
There are more operations you can do, but it depends on *programming
language* you use, so we are not going there in this post.

### Create an array

Now, it's time for some code examples.

*I decided to go with **JavaScript** in
code snippets, sorry for this!*. That's because you can easily try them in your
browser's **console**.

Intuitively **creating** an **array** is like creating a shelf with drawers. We
usually have to
tell **how many** drawers we need, before we can start putting things in them.
Or we can just say:

> I want to put *that thing*, *this another thing* and also *that one* into my
drawers, so please create a shelf in which I can store these things.

To **create** an **array** we usually have to just **declare** it and ascribe it to
some **variable** (in this case it's called `myArray`):

{% highlight javascript %}
    // create an array
    var myArray = new Array("a", "b", "c"); // I want to have a, b and c in my drawers!

    console.log(myArray); // will print ["a", "b", "c"]
{% endhighlight %}

In code snippet above `myArray` is a shelf with drawers indexed from **0** to **2** (because I
wanted to have *3 elements* in my **array**):

<table>
    <thead>
        <tr>
            <th>0</th>
            <th>1</th>
            <th>2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>a</td>
            <td>b</td>
            <td>c</td>
        </tr>
    </tbody>
</table>


Beware - arrays are indexed starting from **0** instead of
**1** in almost all programming languages (except **Pascal**).
{: .notice--danger}

### Access an array

Now you want to **check** what you put in one of your drawers.
For **accessing** arrays elements usually we use `[]` in which we put
number of **index** we want to access. Actually, I know only one example
of programming language where accessing elements looks differently - **Erlang**.
But for **Java**, **Golang**, **JavaScript**, **C++**, **C**, **C#**
and plenty other it is the same.

{% highlight javascript %}
    var myArray = new Array("a", "b", "c");

    // access first element
    console.log(myArray[0]); // prints a

    // change first element
    myArray[0] = "aa";

    console.log(myArray[0]); // prints aa
{% endhighlight %}

So writing `myArray[0]` means - I want to **get** this thing, which is
in my drawer number **0**.
Writing `myArray[0] = "aa";` means I want to put
`aa` into my drawer number **0**.

There are more operations you can do for **arrays**, but these
are the basic ones and the most important ones, so let's not go further
now.

## Array in different programming languages

Here are some examples of an **array** in some other *programming languages*.
I present here how to **create**
an **array** of five integers (numbers) and how to **get** and **set** an
element.

### Java

In **Java** declaring **array** for primitive data types (for example an
*integer*, we don't want to go further in it, don't we?)
is quite simple - almost the same as in **JavaScript**. The difference is that
you have to **declare a type**. In this example:

{% highlight java %}
    int[] myArray = {1, 2, 3, 4, 5};
    System.out.println(myArray[0]); // prints 1
    myArray[0] = 11;
    System.out.println(myArray[0]); // prints 11

{% endhighlight %}

`int[]` means that we declare an **array of integers**. **Setting** and
**getting** element looks like in **JavaScript**.

#### **C#**

**C#** in many ways is quite similar to **Java**. We just need to specify
**type** of our array *twice* in the declaration, because we need to specify
size of our **array**:

{% highlight C# %}
    int[] myArray = new int[5] {1, 2, 3, 4, 5};
    Console.WriteLine(myArray[1]); // prints 2
    myArray[1] = 22;
    Console.WriteLine(myArray[1]); // prints 22

{% endhighlight %}

#### C++

Almost the same as in **Java**, but in **array** *declaration* we have to put
*square brackets*
after **variable** name instead of writing `int[]` as its **type**.
It's because *memory allocation* works a bit differently in **C++**
(we don't want to go any further in it, don't we?).

It's *not necessary* to put
a *number* of elements into these square brackets if we declare all elements
in an **array** like this:

{% highlight C++ %}
    int myArray[5] = {1, 2, 3, 4, 5};
    printf("%d\n", myArray[4]); // prints 5
    myArray[4] = 55;
    printf("%d\n", myArray[4]); //prints 55

{% endhighlight %}

#### Golang

This time number of elements is going to be in square brackets before
**type declaration**, so it's `[5]int`:

{% highlight golang %}
	myArray := [5]int{1, 2, 3, 4, 5}
	fmt.Println(myArray[2])  // prints 3
	myArray[2] = 3
	fmt.Println(myArray[2]) // prints 33

{% endhighlight %}

#### Python

In **Python** it's better to use `list` instead of **array**, but
still it's possible to have an **array** (actually I've never used it!).
We have to import module `array` then we can create an **array** using
*constructor* from this module.
This `'i'` in **array declaration** means that we want to have numbers in it.
You can put other things in it, check documentation.

{% highlight python %}
    import array

    my_array = array.array('i', [1,2,3,4,5])
    print(my_array[3]) # prints 4
    my_array[3] = 44
    print(my_array[3] # prints 44

{% endhighlight %}

#### Erlang

**Erlang** is a bit different language, it uses different paradigm - *functional*
one.
Anyway the same as in **Python**, better to use `lists` instead of **arrays**.

{% highlight erlang %}
    %% array with five zeros - [0,0,0,0,0]
    Array = array:new(5, {default, 0}).
    Elem = array:get(0, Array). %% it's 0
    Array1 = array:set(0, 11, Array).
    Elem1 = array:get(0, Array1). %% it's 11

{% endhighlight %}

### Further reading

To learn more about **arrays** you can read (and code!) how to:

* check arrays **length**
* how to **iterate** over all arrays elements (for example you want to
*print* each of elements in separate line).

I don't give links here, because I don't know which programming language
you prefer (anyway it doesn't really matter...). On the other hand
being a software developer means that you have to be able to **find
answers** on your own.

If you want to share your discovers or issues - please do so in comments!

### Summing up

Here you had a description of what is an **array** and how it looks
like in some *programming languages*.
Next time another important data-structure - **list**! It's a bit similar
to **array**, but there are some differences. And it has more of standard
operations.

**Having some thoughts? Leave a comment!**
