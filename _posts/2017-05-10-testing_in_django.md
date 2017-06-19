---
layout: single
title: Where are my tests?
date:   2017-05-10 20:36:02 +0200
categories: [dsp17, django, python]
excerpt: About some issues you may have with Django test framework
tags: [testing, django, python, quality, frisor]
header:
    teaser: /assets/images/cat_computer_wtf.jpg
---

I decided to add some tests to my **frisor** application, so that my
**Travis CI** could actually be useful. Testing *web applications* is very
annoying for me, I'm not sure what I should **test**. There are some *methods* or *views*,
which are really simple and I don't feel it's possible to write a meaningful
**test** for them. Anyway, as I played more with tests in my side project **frisor**
(more about it here:
[Introduction to frisor]({{ site.baseurl }}{% link _posts/2017-03-01-introduction_to_frisor.md %}))
here are some of my thoughts.

### Where are my tests?

My requirements for **testing** framework were at least:

* tests are in `tests` subdirectory in my application
* having more than one file with **tests**
* readable output with summary

In **Django** there is a builtin *testing* engine which does *test discovery* (so finds tests
locations) and runs tests. It uses `unittest` library for running tests.
To run tests use `manage.py` script with command `test`. When none of your tests will be
found, there can be a few reasons of it:

#### Running manage.py from invalid location

If you are running `manage.py` from invalid location, tests may not be run.
For example: I was running it from as `$ python frisor/manage.py test`,
so not in place where `manage.py` is located. There are two ways of fixing it:

1. use your application name (but it tests only for this application),
in my case:
> `$ python frisor/manage.py test frisor_urls`
2. run `manage.py` when being in place where it is located, so:
> `$ python manage.py test`

#### Subdirectory for tests should be a python package

If you are using `tests` subdirectory it's possible that you forgot to make it
**Python** *package* - this subdirectory has to contain `__init__.py` file as it defines a *package*.

#### Tests have a naming convention

All tests names should start from `test_` prefix to be found. If there are no **tests**
found, check if they are named properly and if your **test class**
derives from `django.test.TestCase`

### How to do something for all tests in class?

Let's say we want to have a default variables or do some actions before for each **test**,
for example I wanted to use the same url for **POST** requests in **all tests**
in my `AddUrlViewTest`:

{% highlight python %}
    class AddUrlViewTest(django.test.TestCase):

        def test_valid_url_form_is_saved_in_database(self):
            data = {
                'url': 'http://valid.se',
                'title': 'short title',
                'nick': 'short nick',
                'tags': 'tag1, tag2'
            }

            resp = self.client.post('/add_url', data=data)
            # I want to use /add_url in other tests too
            # ...

{% endhighlight %}

It can be done using two strategies:
1. running some method *before* **each test**
2. running some method *before* **all tests** (once)

Usually using the first one is better, because **tests** will be **independent**
from each other. When using the second one approach, there might appear some
**dependency** between tests - *you should avoid it at all costs*! When tests are
**dependent** on each other, they can't be run in **parallel**, they are also much harder
to **debug**.

#### Doing something before each test

Implementing `setUp` method can help you set up variables before each **test**,
so before every **test** you can have a fresh instance of variables - `setUp` method
is run before each **test**.

Here I added `setUp` method where I define `url_add_url` variable. I can use
it in all my **tests** to create a **POST** request to that url. It will be easy to change
it in the future.

{% highlight python %}
    class AddUrlViewTest(TestCase):

        def setUp(self):
            self.url_add_url = '/add_url'

        def test_valid_url_form_is_saved_in_database(self):
            data = {
                'url': 'http://valid.se',
                'title': 'short title',
                'nick': 'short nick',
                'tags': 'tag1, tag2'
            }

            resp = self.client.post(self.url_add_url, data=data)
{% endhighlight %}

Of course there is also a method which runs **after** each **test**, it's
called `tearDown`.

#### Doing something before all tests

Sometimes you want to do something before all **tests** in your **test class**.
For example I wanted to create some urls for testing list view of urls
in my `UrlView`.

To achieve this it's enough to implement `setUpClass` method, but you have to remember
it's a `classmethod`, not an instance method. It will be run before **class**
instance creation. For example:

{% highlight python %}
    class UrlViewTest(django.test.TestCase):

        @classmethod
        def setUpClass(cls):
            cls.urls = []
            for i in range(0, 10):
                url = Url.create('http://valid' + str(i) + '.se',
                                 'title' + str(i),
                                 'nick' + str(i),
                                 range(0, i))
                url.save()
                cls.urls.append(url)

        @classmethod
        def tearDownClass(cls):
            for url in cls.urls:
                url.delete()

{% endhighlight %}

In example above I'm creating some urls before running other tests in
my application.
Method `tearDownClass` is run after all tests in **test class** are run.

There are also similar methods for whole modules (`setUpModule` and `tearDownModule`).


You can find my tests in my frisor **github**: [firsor tests](https://github.com/vevurka/frisor/blob/master/frisor/frisor_urls/tests/test_views.py).

