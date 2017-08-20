---
layout: single
title: Customizing admin in Django
date:   2017-07-08 12:49:02 +0200
categories: [python, django]
tags: [django, python, programming, how-to]
excerpt: How to start with customizing django-admin.
header:
    teaser: /assets/images/cat_computer_wtf.jpg
---

## What is django-admin?

django-admin is just a **set** of **templates** and **views** for
**managing models**
{: .notice--info}

So when you use it you don't need to write any
**views** and **templates** to
be able to *manage* the data. But of course it's only when you
want to have a **superuser** access to it. For **regular**
users you should implement your own **views**.

#### How to manage your data in django-admin?

To manage your data using **django-admin** you have to
**register** each *model* you want to use. There is a special
file for it - `admin.py`. When creating new application using
`manage.py` it is created *automatically*.

To add a *default model view* to **admin panel** it's enough to
add `admin.site.register(ModelName)` in `admin.py`. This will create
two views for you:

* list view - for looking at the data
* form view - for adding or editing the data

For example for one of my models in my progression application
it generated a **form** for adding like this:

![screenshot]({{ site.url }}/assets/images/2017-07-09_form_view.png){: .align-center}.

Not customized list view can look like this:

![screenshot](({{ site.url }}/assets/images/2017-07-09_list_view.png){: .align-center}.)

#### A bit about customizing list views in admin

#### A bit about customizing forms in admin

To **customize** **form** for adding an *entity* it's enough to create
a custom form for it and create a *corresponding* class in `admin.py`, which
has to be **registered** along with the **model**.
For my `Tick` **model** it looked
like this:

{% highlight python %}
{% raw %}
    from django.contrib import admin

    from .forms import TickForm
    from .models import Tick

    class TickAdmin(admin.ModelAdmin):
        form = TickForm # TickForm derives from django.forms.ModelForm

    admin.site.register(Tick, TickAdmin)

{% endraw %}
{% endhighlight %}

Default **form** looks like this:

{% highlight python %}
    # forms.py
    from django.forms import ModelForm
    from .models import Tick

    # ...

    class TickForm(ModelForm):
        class Meta:
            model = Tick
            fields = '__all__'

{% endhighlight %}

By the way - this `__all__` is quite bad practice - it means I want to
use **all fields** of my **model** in this **form**. Fields used in **form**
should be written *explicitly*, but on stage of *designing* and *hacking*
it's just easier.

So after you list your fields explicitly you can start to add your custom
widgets to them.

TODO: find a post about it.

