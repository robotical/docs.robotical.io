---
title: MartyPy Library
breadcrumbs:
  - name: 'Python APIs'
    url: '/python'
layout: article
---

Installation
===

The easiest way to install *martypy* is using `pip`, the Python package manager:
{:.feature}

    $ pip install martypy
{:.feature}

Also, when starting a new Python project it is **strongly** recommended that you start it
using a *virtual environment*, with `pyvenv` for Python 3 like so:
{:.feature}


    $ pyvenv VENV
    $ source VENV/bin/activate
    (VENV) $ pip install martypy
{:.feature}

Instead, for Python 2 (although we recommend you don't start using Python 2 for
new projects) use `virtualenv`:

    $ virtualenv VENV
    $ source VENV/bin/activate
    (VENV) $ pip install martypy

This prevents you from having to intall martypy for *all* users on your computer, and lets you
install specific versions of packages withough having confilcts with system packages.

The source code form `martypy` is on [GitHub](https://github.com/robotical/martypy), if you need to
install the beeding-edge in-development code, or want to fiddle with the library, submit issues
or contibute pull requests.



Basic Usage
===

The `martypy` library has a core client class, `Marty` that provides
all the methods used to control a robot. The class takes one required argument,
which is a URL-like string that lets it know where to find your Marty.

{% highlight python %}
>>> from martypy import Marty
>>> mymarty = Marty('socket://marty.local')
>>> mymarty.hello()
True
{% endhighlight %}

If you're using the Marty remotely, over WiFi, that'll be `socket://` followed by the **IP Address**
of your robot, or a resolvable name, like `socket://192.168.0.42` or `socket://marty.local`. If your
robot can't be found, this step may throw an Exception.


Functions and Members
===


**TODO**
{:.bigger.text-danger}



`Marty(url='socket://marty.local',*args, **kwargs)`
{:.docitem}

High-level client library for Marty the Robot.


<br>
