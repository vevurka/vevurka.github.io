---
layout: single
title:  Setting up frisor application
date:   2017-03-03 12:20:02 +0100
categories: [dsp17, python]
excerpt: Django application set up.
---

To set up my application I followed [Django tutorial](https://docs.djangoproject.com/en/1.10/intro/tutorial01/).
It was pretty straight forward and allowed me to set up my application easily.

When working with python it's usually a good idea to use `virtualenv` - it allows to separate environment for each
project. I have a separate virtualenv for each of my projects.

After installing Django in my virtualenv for frisor project I had run `django-admin startproject frisor` which
generated a frisor project for me - the most important file genereted by it is `manage.py`. When you run
`$ python manage.py` you can see available subcommands. I used:

* `$ python manage.py startproject` and subcommand `startapp` to create project and application. It's important to
add application to `settings.py` and create url for it in `urls.py`
* `$ python manage.py runserver 8080` to start a server on port 8080.
* `$ python manage.py migrate` - in basic setup it creates sqlite3 database and runs migrations which weren't
applied yet. Applied migrations are stored in `django_migrations` table.

After setting up database and application I created a basic ORM model for urls I want to store in database:
{% highlight python %}
from django.db import models


class Url(models.Model):
    url = models.CharField(max_length=200)
    publish_date = models.DateTimeField('date published')
    title = models.CharField(max_length=200)
    creator = models.CharField(max_length=200)
{% endhighlight %}

Django is smart and it can generate migrations for created or updated models. Simply run
`$ python makemigrations $your-app-name` to create migration files. Remember it doesn't apply migrations - to apply
them run again `$ python manage.py migrate`.

Before going public with Django application it's important to run `$ python manage.py check --deploy` - it
helps with security checks. It's good to look into `settings.py` file and read carefully
[deployment checklist](https://docs.djangoproject.com/en/1.10/howto/deployment/checklist/) - the link to this checklist
is in one of comments in settings file. From running this command I've learnt I should store my secret key in
environmental variable (or in a file), so I created a production_settings file which will be also in my .gitignore to
prevent accidental publishing. For now my internal settings file looks like:

{% highlight bash %}
export SECRET_KEY="############### secret #############"
export DEBUG=True
{% endhighlight %}

Maybe later I'll think about creating an ansible script for deployment, but it's not necessary for now.
