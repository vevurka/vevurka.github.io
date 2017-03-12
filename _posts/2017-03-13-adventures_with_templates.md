---
layout: single
title:  Adventures with templates
date:   2017-03-13 00:00:01 +0100
categories: [dsp17, python]
excerpt: About adding first django template to my urls box - frisor.
---

## Quick reminder what is **frisor**

**Frisor** is a web application in **Python/Django** which is just a box for interesting urls. When using untrusted network
I don't want to login anywhere to save interesting content I found. More about this is here:
[Introduction to frisor]({{ site.baseurl }}{% link _posts/2017-03-01-introduction_to_frisor.md %}).

This week I added a simple page on which we can see added urls and add url. Also, I introduced logging to my
application.

My github repo: [firsor repo](https://github.com/vevurka/frisor).

Currently my application looks like this:


![screenshot]({{ site.url }}/assets/images/2017-03-13_screen.png)

## Templates in **frisor** with **Django**

**Django** templates are for creating web page html with dynamic content. **Django** uses its own template language called
**DTL**(Django Template Language) which is quite similar to **jinja**. Anyway, it's possible to change template engine,
but I decided to use default one. Using templates is usually pretty straight-forward - at backend side create
a context dictionary with names and values of variables which should be returned in response to fill dynamic parts
of template. It's a common practice to create template engines this way - the same method is used
in **Erlang's ChicagoBoss** and **Zotonic** frameworks or **Ruby's** **jekyll**.

In **Django** you can set up location of your templates in `settings.py` - default location is:
`[app-dir]/templates/[name-of-app]/[name-of-template].html`. I decided to keep that as I want to see how defaults
are working in this framework.

My first template of `index.html` page was about showing list of urls. It looked like this:

{% highlight django %}
{% raw %}
<!-- index.html -->
<ul>
{% for url in url_list %}
    <li>{{ url.id }}, {{url.publish_date}}, {{url.title}}, {{url.creator}}
        <a target="_blank" href="{{url.url}}">{{url.url}}</a>
    </li>
{% endfor %}
</ul>
{% endraw %}
{% endhighlight %}

It uses context dictionary in backend `views.py` which looks like:

{% highlight python %}
    # views.py
    from django.shortcuts import render

    def index():
        context = {
            'url_list': url_list,
        }
        return render(request, 'frisor_urls/index.html', context)
{% endhighlight %}

After lunching my application I saw ugly html page, so it was time to make it look better.
I'm really bad at designing frontend, so I decided to use **bootstrap** for styles - it's simple and looks good without
much effort. That's really great that such things as forms and simple web pages can be created without any
**JavaScript**. I'm considering using **django-bootstrap** library to not care about putting styles into my html code.

Currently frisor is a single page application and I would like to have the same header and footer for other pages.
It can be done with `extend` tag in **DTL**.

### How to put an external url in Django template

My first try with putting url into my template was like this - `url.url` is a value of `url` field of `url` object.
Yeah, I should change name of my model to something else.

    {% highlight django %}
        <!-- index.html -->
        <a href={% raw %}{{url.url}}{% endraw %}>{% raw %}{{url.url}}{% endraw %}</a>
    {% endhighlight %}

#### What's an issue here?
When I put url: `www.example.com` and clicked it I was redirected to `http://localhost:8080/www.example.com`. It appears
that default redirecting is appending url to base server address.

#### Solution
To solve it I should append protocol prefix (`http://`) if it's not provided by user. I'll do this on server side.


### Using forms in **Django**

After fixing bad look of my urls table time had come to create a form to be able to add some urls to my application.
To do this there are a few things needed:

#### Form in html file

On frontend side - form html with **POST** method (because it's going to create a resource):

{% highlight django %}
{% raw %}
    <!-- index.html -->
    <form action="{% url 'frisor_urls:index' %}" method="post">
    {% csrf_token %}
        <div class="form-group">
            <label for="url">URL:</label>
            <input name="url" id="url" type="text" class="form-control"/>
        </div>
        <div class="form-group">
            <label for="title">Title:</label>
            <input name="title" id="title" type="text" class="form-control"/>
        </div>
        <div class="form-group">
            <label for="nick">Nick:</label>
            <input name="nick" id="nick" type="text" class="form-control"/>
        </div>
    <input type="submit" class="btn btn-primary"/>
    </form>
{% endraw %}
{% endhighlight %}

Nothing surprising here except `{% raw %}{% csrf_token %}{% endraw %}` tag. It's for *Cross Site Request Forgery*
protection. *CSRF* attacks can occur when you have a session in web application and somebody will send you a malicious
website which will perform an action using your credentials and session in this application. Currently I don't have
any sessions and users, but better to just add this tag now.

#### Handler for a **POST** request

On backend side - handler for a **POST** request:

{% highlight python %}
    # views.py in index() method
    if request.method == 'POST':
        nick = request.POST['nick']
        url = request.POST['url']
        title = request.POST['title']

        # TODO: do something about this data
{% endhighlight %}

I was wondering how to get to data sent by **POST** request. At first I tried `request.body`, `request.read`,
but it happens that Django puts all data into dictionary variable named as received request.
My form sends a **POST** request so dictionary at backend side is called **POST** and I can get to
field in data like this: `request.POST['name_of_field']`.

### Store url in database

In my model of `Url` (in `models.py`) I had to add a method for creating an url in database. Of course I could create an `Url` object
using a constructor, but then I had to worry about putting unnecessary data like `publish_date`, which should always
be set to current time when creating this object. My `Url.create` method:

{% highlight python %}
    # models.py
    from datetime import datetime
    from django.db import models


    class Url(models.Model):
        url = models.CharField(max_length=200)
        publish_date = models.DateTimeField('date published')
        title = models.CharField(max_length=200)
        creator = models.CharField(max_length=200)

        @classmethod
        def create(cls, url="", title="", creator=""):
            url = cls(title=title, url=url, creator=creator, publish_date=datetime.now())
            return url
{% endhighlight %}

To create an `Url` object in my view I had to do:

{% highlight python %}
    # views.py in index() method
    url = Url.create(url=url, title=title, creator=nick)
    url.save()
{% endhighlight %}

It's obvious that `save` method is needed to save an object in database. Create method doesn't perform saving
and in my opinion it shouldn't. Sometimes it's needed to create a temporary object to do some actions using it
and after them throw it away. Saving logic should be extracted to some service - it shouldn't be done in view.
But as I'm following *KISS* principle this can be done later.


### How to display *Django* template code on my blog?

At my blog I'm using **jekyll** with **liquid** as template language. Actually **liquid** is quite similar
to **jinja** which is similar to **DTL**. So there was a funny issue with this. I wanted to display some of
my template code into this post on this blog.
At first it was quite surprising that when putting something like this into my posts markdown:
{% raw %} `{{url.url}}` {% endraw %} it actually became an empty string. It's because **jekyll** is interpreting this
as **liquids** filter and `url.url` is not defined so it becomes an empty string.

#### Solution

To be able use **DTL** code in my posts I've to use a special tag:
{% highlight jinja %}
    {% raw %} {% raw %}  {{ url.url }} {% endraw %} {% raw %}{% {% endraw %}{% raw %}endraw %}{% endraw %}
{% endhighlight %}


## Logging

Default logging configuration added to **Django** doesn't log to console anything except **Django** logs, so when
you put something like this into your package:

{% highlight python %}
import logging

logger = logging.getLogger(__name__) # get an instance of logger
{% endhighlight %}

and use it inside your packages as `logger.info("Info log")` it will not display those logs in console after running
`$ python manage.py runserver`.

Solution for this is adding custom logging configuration in `settings.py`.
My logging configuration (see comments in a code):

{% highlight python %}
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,  # it means we don't want to remove Django logging configuration
    'formatters': {  # to format logs
        'verbose': {
            'format':
                '%(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(asctime)s %(module)s %(message)s'
        }
    },
    'handlers': {  # how logs should be handled - they can be written to file, console, etc.
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        }
    },
    'loggers': {
        'django': {
            'handlers': ['console'],
            'propagate': True,
        },
        'django.request': {
            'handlers': ['console'],
            'level': 'ERROR',
            'propagate': False,
        },
        'frisor_urls.views': {  # my custom view
            'handlers': ['console'],
            'level': 'DEBUG',
        }
    }
}
{% endhighlight %}



## My TODO list:

0. Introduce unit tests.
1. When creating append `http://` to url if it's not there - someone told me that I can do
this also with `UrlField` - probably I should've read documentation more carefully.
2. Introduce `django-bootstrap` instead of currently used `bootstrap` to templates.
3. Introduce footers and headers in html pages.
3. Introduce validation to url form.
3. Add comments and tags to urls
4. Refactor database logic from `views.py` to use some `service.py`