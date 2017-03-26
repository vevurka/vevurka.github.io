
# Testing in **Django**

## Where are my tests?

My requirements for testing framework at least:

* tests are in `tests` subdirectory in my application
* having more than one file with tests
* readable output with summary

In **Django** there is a builtin *testing* engine which does *test discovery* (so finds tests
locations) and runs tests. It uses `unittest` library for running tests.
To run tests use `manage.py` script with command `test`. When none of your tests will be
found, there can be a few reasons of it:

* you are running `manage.py` from invalid location - that was in my case.
I was running it from as `$ python frisor/manage.py test`, so not in place where `manage.py`
is located. There are two ways of fixing it:
    1. use your application name (but it tests only for this application):
    in my case  `$ python frisor/manage.py test frisor_urls`
    2. run `manage.py` when being in place where it is located, so: `$ python manage.py test`

* if you are using `tests` subdirectory it's possible that you forgot to make it
**Python** *package* - this subdirectory has to contain `__init__.py` file as it defines a *package*)

* all tests names should start from `test_` prefix to be found




