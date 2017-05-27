---
layout: single
title: Travis CI for Python project
date:   2017-05-27 21:58:02 +0200
categories: [dsp17, git, quality, django, python]
excerpt: Setting up Travis CI for github repo in Python
---

In my last post I wrote about **Travis CI**:
[Travis CI overview]({{ site.baseurl }}{% link _posts/2017-05-21-travis_ci.md %}).
In this post I want to write how to use **Travis CI** for **Python/Django** project.

I started to be interested in **Travis CI** when I decided it's time for
some continuous integration for my recent side project -
*frisor* ([firsor repo](https://github.com/vevurka/frisor)).

### Quick reminder what is **frisor**

**Frisor** is a web application in **Python/Django** which is just a box for interesting urls. When using untrusted network
I don't want to login anywhere to save interesting content I found. More about this is here:
[Introduction to frisor]({{ site.baseurl }}{% link _posts/2017-03-01-introduction_to_frisor.md %}).

Here is **Travis CI** support file for my project:
[travis support file](https://github.com/vevurka/frisor/blob/master/.travis.yml).

And here is my **Travis CI** build status:
[![Build Status](https://travis-ci.org/vevurka/frisor.svg?branch=master)](https://travis-ci.org/vevurka/frisor)

I also configured **code coverage** for my build:

![screenshot]({{ site.url }}/assets/images/2017-05-27_coveralls.png)

## Travis CI configuration for Python language

To define how application build should be performed,
we need to create `.travis.yml` file. It has to contain three parts:
1. Declaration of **language** or technologies used
2. **Install** instructions - for installing necessary dependencies
3. **Script** instructions - for building

### How to declare Python language in .travis.yml?

In `.travis.yml` you should define language you use using `language` tag:

{% highlight yaml %}
    language: python
{% endhighlight %}

Then define **Python versions** - default is **2.7**:

{% highlight yaml %}
    python:
        - 3.4
        - 3.5
        - 2.7
{% endhighlight %}

Build will be performed for all declared versions. **Travis CI** uses **virtualenv**
for **Python** *environments*
(for each declared **Python version** there will be a separate one).
After declaring a **language**, we need to write commands which should be executed when
installing *dependencies*.

### Installation part of .travis.yml

It's recommended to *install* necessary **dependencies** by `pip`, not `apt-get`.
If you want `apt-get` (for example for **numpy** or **matplotlib** libraries) you need to
use `before_install` tag, for example:

{% highlight yaml %}
    before_install:
      - sudo apt-get -qq update
      - sudo apt-get install -y python-numpy
{% endhighlight %}

By default these libraries are preinstalled:
* **pytest**
* **nose**
* **mock**

Default instruction for **installing** part will be performed when
you don't write it at all:

{% highlight yaml %}
    install: pip install -r requirements.txt
{% endhighlight %}

For some small projects it can be enough, so don't forget to add `requirements.txt` file
to your repository!

#### Multiple Django versions support in .travis.yml

**Python** projects quite often come with **Django** framework. If you are using only
one version of that library, it's enough to **install** it by adding it
to `requirements.txt`.

But if you want to test against **multiple** versions,
you can add **environment** variables (remember, **Travis CI** will create a build
matrix for all **Python versions** and `env` variables for your project):

{% highlight yaml %}
    env:
      - DJANGO_VERSION=1.7.8
      - DJANGO_VERSION=1.8.2
{% endhighlight %}

Then in your `install` instructions you should use `DJANGO_VERSION` environment variable
for installing proper version of **Django**, for example:

{% highlight yaml %}
    install:
      - pip install -q Django==$DJANGO_VERSION
      - pip install -r requirements.txt
{% endhighlight %}

You can use the same way for managing multiple versions of other libraries too.

### Script part of .travis.yml for Python

There is no default script part - build will fail if your `.travis.yml` doesn't contain it.
Usually you just want to **run tests**:

{% highlight yaml %}
    script: # pick one you use for running tests
        - pytest
        - nosetests
        - python -m unittest
{% endhighlight %}

For project in **Django** it can look like this:

{% highlight yaml %}
    script: python manage.py test
{% endhighlight %}

### Code coverage in Travis CI for Python

For **code coverage** you will need to install necessary tools first:

{% highlight yaml %}
    install:
      - ...
      - pip install coverage
      - ...
{% endhighlight %}

Then, after successful build run code coverage tools:

{% highlight yaml %}
    after_success: coverage report
{% endhighlight %}

It'll print **coverage** results into **Travis CI** console:

![screenshot]({{ site.url }}/assets/images/2017-05-27_travis_console.png)

Of course you can't forget to use `coverage run` when running your tests!
Usually it means that you should change **Travis CI** `script` commands a bit, for me
it looks like:

{% highlight yaml %}
    script: coverage run manage.py test
{% endhighlight %}

There are tools
like **coveralls** which can help you with showing coverage results for your
repository. It comes with some nice badges:
[![Coverage Status](https://coveralls.io/repos/github/vevurka/frisor/badge.svg?branch=master)](https://coveralls.io/github/vevurka/frisor?branch=master)

It can also put information about **code coverage** in **pull requests**.
More about it on [coveralls](https://coveralls.io/).

### Summing up

Adding **Travis CI** support for **Python/Django** project is easy, especially
if you follow good practices for this technologies like using
**virtualenv** and having dependencies in some `requirements` file.

There are plenty other useful tools which can be *integrated* with
**Travis CI** like tools for **code coverage**, **slack** integrations,
**email** notifications, etc. For side projects it is more than enough.
For commercial ones I've never used **Travis CI**, but it happen I used other
integrations existing on **github**.
