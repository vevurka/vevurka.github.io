---
layout: single
title:  Introduction to frisor
date:   2017-03-01 12:20:01 +0100
categories: dsp17
tags: [programming, django, python, design, frisor]
excerpt: About my dsp17 project
---

I decided to participate in [Get Noticed!](http://devstyle.pl/daj-sie-poznac/) competition (in polish "Daj się poznać" so category name is dsp17).
My project for dsp17 is an application in Python/Django for my own use. User story which
is behind my project is when using untrusted network I don't want to login to my personal accounts to store interesting stuff I've found. So I want to have a "box" for interesting urls. Another user story it that sometimes when using untrusted network I want to be able to chat with my boyfriend -
one solution is using qTox, but it requires an installation. So I want to chat using frisor application.

### Must requirements
As user I want to be able to:
* put url with title
* search urls by titles

### Could requirements
* login - authorization
* have url categories
* add comment/edit url (after login)
* have a secret chat with other user (it might require login)

### Design
When following only must requirements it's enough to say that it will be a web application which reads and writes to database. Maybe later it'll use some
cache server to speed up searching urls.
For "could requirements" I probably will need to have some event queue.

### Technology
I'm following KISS principle, so I don't want to overkill this very simple task. On the other hand I want to have a space
for improvements and new features.

I know Python, but I've never done anything in Django. My friends are claiming that Django is a solid web framework, so I want to check it.
I'm planning to do frontend using Bootstrap.
For storing records I'm going to use SQLite database (I want to check how easy is migrate it to Postgres later) - it's easy to set up and
it's enough for now.
For chat I want to use WebSockets. Maybe chat backend will be in Erlang or Elixir, but I'll start with chat when I cover must requirements. Maybe I'll need to introduce
some event queue like RabbitMQ, but it's too early to tell.



