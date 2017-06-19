---
layout: single
title: Basic data structures - list
date:   2017-06-16 00:49:02 +0200
categories: [cs, basics, algorithms]
tags: [computer-science, data-structures, algorithms, overview]
excerpt: What is a list? Examples of lists in some programming languages.
header:
    teaser: /assets/images/2017-06-16_train.jpg
---

Here is very basic description on what is an **list** (actually *single*
**linked list**). I tried to describe
things as *simple as possible*, sometimes skipping some details.
This post is meant to be for
people who just started programming.

My previous post was about
[an array]({{ site.baseurl }}{% link _posts/2017-06-08-data_structures_array.md %}),
which is sometimes
used for *implementing* a **list** data structure. The most important difference
between these two is that **array** has **fixed length** and **list** has **dynamic one**.

# List

**List** is a *collection*, so we use it to store more than
one *thing*.

**Intuitively** you can think that **list** is a **train**:

![screenshot]({{ site.url }}/assets/images/2017-06-16_train.jpg){: .align-center}.

We use this **train** to store our **boxes** - we don't care about their size.
Because they can be big or small, in every **train car** we can store
**only one box**.
Starting from locomotive **each train car** is *connected* to the **next one**.

The same with list - it starts from node called **head** (*locomotive*).
**Head** is connected with next element (next *train car*).
Next element is connected with the next one, and so on...
Speaking a bit more **formally**:

**List** is a *collection* of countable number of **ordered** values stored
in nodes, in which each node is *pointing* to the next node.
{: .notice--success}

## What you can do with a list?

Let's start on what we can do with a **train**? We can:

* **create** a *train*
* **append** a *train car* in which we want to store some *box*
* **search** for a specific *box* - find a *train car* in which it is

The same with list, we can:

* **create** a list
* **append** element to list
* **search** for specific element - we want to know its *index* (location)

In some programming languages **string** data type is implemented as **list** -
it's **list of characters**.
{: .notice--info}

This time code examples are in **Python**, because in
*plain* **JavaScript** there is no properly
implemented **list**.

### Create a list

In **Python** creating an *empty list* is very simple:

{% highlight python %}

    my_list = []

{% endhighlight %}

We can also create a **list** with some *initial elements*:

{% highlight python %}
    my_list = ["a", "b", "c"]
    print(my_list) # prints ["a", "b", "c"]
{% endhighlight %}

Intuitively - it's a **train** with **3 train cars** - in the first one
there is box with letter *a*, in the second one - *b* and in the third one - *c*.

### Append to list

When we have a **train** it's quite easy to append another *train car with box*
to it. We always append *new train cars* to the end of the **train**.
The same is when *appending an element* to **list** - in **Python** we use `add`
method for this (in other programming languages it can be also called
`append` or `push`):

{% highlight python %}
    my_list = ["a", "b", "c"]
    my_list.add("d")
    print(my_list) # prints ["a", "b", "c", "d"]

{% endhighlight %}

Of course it's possible to **add** an element in the *middle* of the **list**,
but it's not a standard list operation.

**Append** operation makes **lists** having **dynamic length**, because we can change
size of this structure.

### Search list

We sometimes want to know in which **train car** is one of *boxes*.
For **list** we can also check
on which *index* it's a certain element. In **Python** there is `index` method
for this:

{% highlight python %}
    my_list = ["a", "b", "c"]

    i = my_list.index("a")
    print(i) # prints 0, because "a" is the first element in my_list

    my_list.index("d") # ValueError: "d" is not in list

{% endhighlight %}

We can also check what we have in certain *train car*. In **list**,
elements are **enumerated** starting from **0** - the same as in **arrays**.

 {% highlight python %}
     my_list = ["a", "b", "c"]

     i = my_list[0]
     print(i) # prints "a", because "a" is the first element in my_list

     my_list[5] # IndexError: list index out of range

 {% endhighlight %}

Beware that **get** operations aren't in standard library in all languages,
for example in **C++** or **Erlang** we always have to search for element
by looking to *all elements* starting from the **head** of the **list**.
{: .notice--error}


## List in different programming languages

Lists are implemented in many different manners. Here are some examples
from **objective** and **functional** languages.

### Java

There are a few ways of creating an **array** in **Java**, I think this is
the simplest one:
{% highlight java %}
    List myList = Arrays.asList(1, 2, 3);
    myList.get(1); // returns 2 because it's the second element
{% endhighlight %}
but the *downside* is that it cannot be modified
(it's a *immutable* **list**).
Thanks to Jakub Dyszkiewicz for pointing that out!

The other way:

{% highlight java %}
        List myList = new ArrayList();
        myList.add(1);
        myList.add(2);
        myList.add(3);
        System.out.println(myList.get(0)); //prints first element: 1
{% endhighlight %}

This time `myList` is **mutable**, so we can *add elements* after it is created.
I will not cover here more about *mutability* and *immutability*, but
it's an interesting topic - maybe in future posts?

### **C#**

**C#** is quite similar to **Java**:

{% highlight c# %}
        List<string> myList = new List<string>();
        myList.Add(1);
        myList.Add(2);
        myList[1]; // returns 2, because it's the second element

{% endhighlight %}

### C++

**C++** is one these languages which doesn't provide a special
method for accessing elements of a **list** - we have to *iterate*
over this structure, it's not covered in an example.

{% highlight c++ %}
    std::list<int> mylist;
    mylist.push_back(1);
    mylist.push_back(2);
    mylist.front(); // access the first element

{% endhighlight %}

### Erlang

In **functional paradigm** **lists** are very useful - I can even say they are
the **core** of functional programming. They are still the same data structure,
but we don't think about *indexes* at all. The truth is that **lists**
are completely **not about indexes**. We treat **list** as structure
which has a **head** and a **tail** -
**head** is our **locomotive** (and the first train car in the same time)
and **tail** is **the rest** of the train.

{% highlight erlang %}
    L = [1,2,3],
    L1 = [4|L]  %% we can append easily to the beginning of the list
    %% L1 is [4,1,2,3]

    %% this is called pattern matching:
    [H|T] = L1
    %% H is 4 and T is [1,2,3]

{% endhighlight %}

### Further reading

There are more interesting operations on **lists** like:

* filter
* map
* reduce (or fold)

You can also read a *documentation* on **lists** in your favorite
programming language. Also it's nice to read a bit
what is *pattern matching* and *list comprehensions*.
I think it's also good to know more on *mutability*.

If you have any questions - leave a comment!

### Difference between array and list

If you've read a previous post about
[arrays]({{ site.baseurl }}{% link _posts/2017-06-08-data_structures_array.md %}) then you probably
noticed some *similarities* between **array** and **list**, so I want to describe
what are the *differences* between them. Remember, **array** was intuitively
a *shelf with drawers*.

#### Length difference

The most important difference
between these two is that **array** has **fixed length** and **list** has **dynamic one**.

It means that **we cannot change a size of an array** - it's hard to add a *new drawer*
to an *existing shelf*! We would actually need to buy a **new shelf** and put all our
drawers into it (for changing a size of an **array** our program needs to
**reallocate memory**).
On the other hand it's easy to append *new train car* to
a *train* - the same as it is easy to **append** new element to **list**.

#### Accessing time difference

There is also difference in **time** which is needed for certain operations -
for example if we have a very very **long list**, *accessing* its last element probably
will take **much more time** than accessing last element of an **array** of the same
length.

That's because in **list** we always **start from the beginning** and
in **array** we can easily count which place in computer memory should be accessed.
More *intuitively*: **list** is a *train*, so as a conductor we have to walk
from *train car* to
next *train car* and a next one and next... On the other hand
**array** is a *shelf with drawers*, that's why we can easily find
a *drawer* even when *shelf* is really big.

### Summing up

Hopefully you understand, what is a **list** and why it works this way.
You also have seen some examples in other programming languages and you
are aware that **list** can look in very different manners.
Also you know a bit about the differences between **arrays** and **lists**.

Next time
we will try to understand what is a *data structure* called **binary tree**!

**Having some thoughts? Leave a comment!**
