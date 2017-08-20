---
layout: single
title: Customizing select widget in Django
date:   2017-07-08 12:49:02 +0200
categories: [python, django]
tags: [django, python, programming, how-to]
excerpt: How to add styles (colors) to options in dropdown in Django.
header:
    teaser: /assets/images/cat_computer_wtf.jpg
---

Yesterday I started to work on some *application* for managing my
**climbing progress**.
Currently it does what it is supposed to do (I can *add* a new boulder,
*add* my attempt of it and *edit* these things), when using **Django**
this kind of *application* can be done quite fast.
It's because **Django** has **django-admin**.

I didn't need to write any **views** and **templates** by myself to
be able to *manage* my data
(but probably will do that in the future, have to check if
other *climbers* will want to use it).

I had some issues with implementing one of my **requirements**
and this is what I'm going to describe in this *post*.

## Troublesome requirement - colors!

This troublesome **requirement** was about *styling* **dropdown list**
of **boulders**
(these are just *climbing problems* - they are described
by **color**, **level** and **location**). I wanted to add
a **color** to each **option** field of **dropdown** to be
able to easily pick a *boulder* I've done - like this:

![screenshot]({{ site.url }}/assets/images/2017-07-09_color_select.png){: .align-center}

On the list I wanted to have a **location** and a **level** of hardness (for
example `J 4` - means that boulder is in *J sector* and has *level 4*),
but not the **color** hash. This **color** hash still had to be passed
to the **dropdown** option, because I needed it to set a *background color*
of **option** field.

### How to add colors to option in select

Beware - my solution is a bit dirty, in *production code*
I would just write my own **widget** for this!
{: .notice--danger}

As I stated above - writing my own **widget** wasn't an option for me, I
wanted to have results quicker, even when they are ugly. If I would go
for writing my own **widget** for this, it could be similar to
`django.forms.ChoiceWidget`, but with different `create_option` method.

#### Model

First let's take a look at relevant part of my **models**:

{% highlight python %}
    # models.py
    from django.db import models
    from django.utils import timezone

    #...

    class Boulder(models.Model):
        color = models.CharField(max_length=10)
        level = models.CharField(max_length=10)
        place = models.ForeignKey(to=ClimbingWallPlace)

        def __str__(self):
            return self.color + " " + self.place.place + " " + self.level

    class Tick(models.Model):
        boulder = models.ForeignKey(to=Boulder)
        tries = models.IntegerField()
        success = models.BooleanField()
        date = models.DateField(default=timezone.now)

{% endhighlight %}

`Boulder` is a **model** for a climbing problem - as I mentioned
before it has *color*, *level* and *place*.
There is a strange method `__str__` in `Boulder` *class*,
but we will come back to it later.

`Tick` is a **model** of *attempt* - for this one I want to be able to
pick a *boulder* I've done, *date* and *number of tries*. For this **model**
I want to have a **form** in which I'll be able to easily pick
a *boulder* from **dropdown** by it's **color**.

So now we need to create a form with customized **widget** for generating
a **dropdown** for *boulder* field in `Tick`.

### My approaches to customize dropdown

I had quite a few approaches for customizing this dropdown for *boulders*.
All of them were related to `django.form.Select` **widget**, because
it's used by default for this kind of fields.

Without any customizations it looked like this:

![screenshot]({{ site.url }}/assets/images/2017-07-09_select.png){: .align-center}

This could be acceptable for some color hash master, but not for me...

So let's start from my failed approaches, if you are not interested just
scroll down to: TODOOOOO

#### My failed approaches

##### Choices

Select widget in Django can get a choices as tuple of value-label tuples,
for example:
((1, 'my-value1'), (2, 'my-value2')).

My my first idea was to allow passing choices as tuple of three elements
into
the select widget:
((1, 'my-label1', '#FFFFFF'), (2, 'my-label2', '#000000'), ...)
but it would require some rewriting of select widget.
That's because when using Model based form (so form which derives from django.forms.ModelForm)
Select widget doesn't allow to override choices field - it just uses
fields from Model and it generates widget.label using __str__ method
in model.

##### Custom fields

Passing custom fields from widget is not a good idea also, especially
that it would be passed to select template, instead of select_option template.
So, doing something like this is sadly impossible:
{% highlight jinja %}
    {% raw %}
        <!-- IMPOSSIBLE! -->
        <option style='background-color: {{ widget.color }}'
                value="{{ widget.value|stringformat:'s' }}"
                {% include "django/forms/widgets/attrs.html" %}>
          {{ widget.label }}
        </option>
    {% endraw %}
{% endhighlight %}

That's because Select derives from ChoiceWidget which does
create_option method which returns:

{% highlight python %}
    # django.forms.widgets.py in version 1.11.3

    #...
    class ChoiceWidget(Widget):
        # ...

        def create_option(self, name, value, label, selected, index, subindex=None, attrs=None):

        # ...
        # ... some more code

        return {
            'name': name,
            'value': value,
            'label': label,
            'selected': selected,
            'index': index,
            'attrs': option_attrs,
            'type': self.input_type,
            'template_name': self.option_template_name,
        }

{% endhighlight %}



#### Color customization for select widget

I wrote a `TickForm` to add a customization for a **widget** used by
*boulder* field. By default for *picking things from a list* there is a
`django.forms.Select` **widget** used,
so I decided that my *custom widget* - `ColorSelect` will **derive** from it:

{% highlight python %}
    # forms.py
    from django.forms import ModelForm, Select
    from django.forms.widgets import TextInput
    from .models import Boulder, Tick

    # ...

    class ColorSelect(Select):
        option_template_name = 'select_color_option.html'


    class TickForm(ModelForm):
        class Meta:
            model = Tick
            fields = '__all__'
            widgets = {
                'boulder': ColorSelect(),
            }

{% endhighlight %}

So it seemed that to add some *styles* to **options** in `ColorSelect` **widget**
I had to change the **template** used for generating *options*.
That's why I've overridden `option_template_name` in `ColorSelect` with name of
my custom template.

The only thing which was left was creating a `select_color_option` **template**.
I needed
to put somehow all info I wanted to have to `widget.label`, that's way I
implemented `__str__` method in my **model** of `Boulder`:

{% highlight python %}
    # models.py
    from django.db import models

    #...

    class Boulder(models.Model):
        color = models.CharField(max_length=10)
        level = models.CharField(max_length=10)
        place = models.ForeignKey(to=ClimbingWallPlace)

        def __str__(self):
            return self.color + " " + self.place.place + " " + self.level

{% endhighlight %}

Here - as string I'm showing *color* with *name of a sector* and *level*,
for example: '#0123456 J 4'.
In `select_option` template I've to extract this data using a very
strange way. I'm very surprised that **Django** doesn't have a *tag*
for **splitting** *strings* in templates... Here is my template:

{% highlight jinja %}
    {% raw %}
        <!-- select_color_option.html -->
        <option style='background-color: {{ widget.label | make_list | slice:":7" | join:""}}'
                value="{{ widget.value|stringformat:'s' }}"
                {% include "django/forms/widgets/attrs.html" %}>
          {{ widget.label | make_list | slice:"7:" | join:""}}
        </option>
    {% endraw %}
{% endhighlight %}

To extract **color** from *boulder label* (this **string** which
is returned from `__str__` method from `Boulder` model) I use:

    {% raw %} {{ widget.label | make_list | slice:":7" | join:""}} {% endraw %}

Tag `make_list` makes a char list from string. From char list I cut the color name
(first 7 characters) and then I do join to make it string again:

#0123456 J 4 -> ['#', '0','1','2','3','4','5','6',' ','J',' ','4']
->  ['#', '0','1','2','3','4','5','6'] -> "#0123456"

For having the label without a color:
{% raw %} {{ widget.label | make_list | slice:"7:" | join:""}} {% endraw %}
- the difference is in slice - there is 7: instead of :7, so it means
I cut string from the 7-th character till the end.
