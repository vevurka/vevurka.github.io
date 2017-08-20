---
layout: single
title: Select widget in Django
date:   2017-07-08 12:48:02 +0200
categories: [python, django]
tags: [django, python, programming, how-to]
excerpt: How does select widget in Django works?
header:
    teaser: /assets/images/cat_computer_wtf.jpg
---


This widget from a tuple of id-labels tuples (called choices)
generates a dropdown with options - id are values send when submiting
form on which this dropdown is, labels are rendered as options.

# Select widget

## Options

It derives from ChoiceWidget, so it uses its method for creating options:

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



## Attributes

## Fields

When I looked to the source of `django.forms.Select` I saw that
it has fields:

{% highlight python %}
    # django.forms.widgets.py - version 1.11.3

    # ...

    class Select(ChoiceWidget):
        input_type = 'select'
        template_name = 'django/forms/widgets/select.html'
        option_template_name = 'django/forms/widgets/select_option.html'
        add_id_index = False
        checked_attribute = {'selected': True}
        option_inherits_attrs = False

{% endhighlight %}



## Templates

It uses two templates:
* select
* option

Default `select_option.html` template used in **Django** looks like this:

{% highlight jinja %}
{% raw %}
    <option value="{{ widget.value|stringformat:'s' }}"
        {% include "django/forms/widgets/attrs.html" %}>{{ widget.label }}
    </option>

{% endraw %}
{% endhighlight %}

