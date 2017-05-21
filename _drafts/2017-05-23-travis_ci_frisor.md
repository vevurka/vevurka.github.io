---
layout: single
title: Travis CI for Python
date:   2017-05-21 21:49:02 +0200
categories: [dsp17, architecture]
excerpt: Setting up Travis CI for github repo in Python
---

As introducing **django-channels** to my application - **frisor** will take more time,
I decided to write this short post about how easy was to add **Travis CI**
to my project repository.

### Travis CI configuration for Python

I'll describe .travis.yml for my project in Python a bit more:

First line should define language you use

    language: python

Then define a Python versions:

    python:
        - 3.4
        - 3.5


After that we need to write commends which should be executed when
installing. Travis CI uses virtualenv for python environment (for
each Python version you define there will be a separate one). It's almost
required to install necessary dependencies by using pip, not apt-get.
If you need apt-get you need to use before_install:

    before_install:
      - sudo apt-get -qq update
      - sudo apt-get install -y randomlib

This is default instruction for installing part:

    install: pip install -r requirements.txt




