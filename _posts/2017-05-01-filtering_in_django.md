---
layout: single
title: Filtering in Django
date:   2017-05-01 21:34:02 +0100
categories: [dsp17, django, python]
excerpt: About filtering and managing multiple third party libraries in Django
header:
    teaser: /assets/images/cat_computer_wtf.jpg
---

## Quick reminder what is **frisor**

**Frisor** is a web application in **Python/Django** which is just a box for interesting urls. When using untrusted network
I don't want to login anywhere to save interesting content I found. More about this is here:
[Introduction to frisor]({{ site.baseurl }}{% link _posts/2017-03-01-introduction_to_frisor.md %}).

This week I introduced *filtering* for my urls table. I decided to use **django-filter** for it and
I had some issues with integrating it with other *third-party* libraries I use.

**Frisor** on github: [firsor repo](https://github.com/vevurka/frisor).

Currently *filtering* in my application looks like this:

![screenshot]({{ site.url }}/assets/images/2017-05-01_screen.png)


## Setting up filtering

Setting up filtering with **django-filter** is quite straight forward. It has to be installed via `pip`
then added to `INSTALLED_APPS` in `settings.py`. After that you need to create an equivalent of `form`
for filtering, manage it in your view and put generated form in your template.

So let's start from the beginning:

### Filtering abstraction

Here is an *abstraction* for **filtering**, which will generate a filtering **form** for me,
it derives from `django_filters.FilterSet`:

{% highlight python %}
    # filters.py
    from django_filters import FilterSet

    from .models import Url


    class UrlFilter(FilterSet):
        class Meta:
            model = Url
            fields = ['url', 'title', 'tags__name']
{% endhighlight %}

`UrlFilter` will generate a simple form, which will allow to *filter* by exact match of *tag name*,
*title* or *url*.

It uses my  `Url` model and its fields: `url`, `title` and `tags__name`.
This last field contains `__` - it's **field lookup** which
is used in **Django** to access *subfields* of a structure. In my `Url` model I've
a field `tags` - it has type `tagulous_models.TagField`, so it has `name` field. That's why in
`UrlFilter` form I can access it via `tags__name`.


### Manage filtering in view

My **view** which manages *filtering*, *pagination* and showing *list of urls* looks like this:
{% highlight python %}
    # views.py
    class UrlView(FormView):
        # ...

        def get_context_data(self, **kwargs):
            context = super().get_context_data()
            url_list = Url.objects.order_by('-publish_date').all()
            url_filter = UrlFilter(self.request.GET, queryset=url_list)
            context['url_list'] = self._get_url_page(url_filter.qs,
                                                     self.request.GET.get('page'))
            context['url_filter'] = url_filter
            return context

{% endhighlight %}

The most important is `url_filter = UrlFilter(self.request.GET, queryset=url_list)`, which
creates an `UrlFilter` object which uses filtering parameters from `GET` request and list of urls as **queryset**.
For example for `GET` request:

> `/?url=vevurka.github.io&title=vevurka&tags__name=github`

it will contain only urls with **url** *vevurka.github.io*, **title** *vevurka* and
**tag name** *github*. We can access its **queryset** by `url_filter.qs`.

We have to put `UrlFilter` object to rendering **context** to be able to use its data
in *template*. Now we can set up a template:

### Filtering in template

As I'm using **django-bootstrap3** for rendering forms I thought it will be simple
to use nice *bootstrap form* in my template. Because I added `url_filter` to
view context, I can access filter form by `url_filter.form`:

{% highlight django %}
{% raw %}
    <!-- index.html -->
    <form action="" method="get">
        <legend>Filtering</legend>
        {% bootstrap_form url_filter.form %}
      <div class="form-group">
        <button type="submit" class="btn btn-primary">Submit</button>
      </div>
    </form>
    <!-- url list -->
    <!-- ... -->

    {% bootstrap_pagination url_list %}
{% endraw %}
{% endhighlight %}

That was it, but it happen that it broke my **pagination** - after changing page, my *filter
query* disappeared (both from form and url)!

After a bit of debugging and looking
into the source of **django-bootstrap3** library, I saw that when tag
`bootstrap_pagination` creates a links for pages, it uses current `GET` request to do it, but
**clears** other parameters except page one... I don't know why the designers of **django-bootstrap3**
decided to do this like this:

{% highlight python %}
    # bootstrap3.py from django-bootstrap3 library
    def get_pagination_context(page, pages_to_show=11,
                               url=None, size=None, extra=None,
                               parameter_name='page'):
        # ...
        if url:
            # Remove existing page GET parameters
            url = force_text(url)
            url = re.sub(r'\?{0}\=[^\&]+'.format(parameter_name), '?', url)
            url = re.sub(r'\&{0}\=[^\&]+'.format(parameter_name), '', url)
{% endhighlight %}

But I saw hope in `extra` parameter of the same `get_pagination_context` function:

{% highlight python %}
    # bootstrap3.py from django-bootstrap3 library
    def get_pagination_context(page, pages_to_show=11,
                               url=None, size=None, extra=None,
                               parameter_name='page'):
        # ...
        if extra:
            if not url:
                url = '?'
            url += force_text(extra) + '&'
{% endhighlight %}

After some googling I came up this solution - adding an `extra` parameter to
`bootstrap_pagination` tag with all parameters from request:

{% highlight python %}
{% raw %}
    {% bootstrap_pagination url_list extra=request.GET.urlencode %}
{% endraw %}
{% endhighlight %}

### Summing up

So far for me it's impossible to use any *third-party* **Django** libraries without
looking to their source.
Documentation usually doesn't cover all special cases. On the other side life is to short
to read whole documentation without need :wink:... These libraries present different levels of
*quality*, code sometimes is quite bad or designed in strange way. Don't expect they
will work well together.

