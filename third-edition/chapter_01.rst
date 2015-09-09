==========================
Chapter 1: Getting Started
==========================

There are two very important things you need to do to get started with Django:

1. Install Django (obviously); and
2. Get a good understanding of the Model-View-Controller (MVC) design pattern.

The first, installing Django, is really simple and detailed in the first part of
this chapter.

The second is just as important, especially if you are a new programmer or 
coming from using a programming language that does not clearly separate the data and logic
behind your website from the way it is displayed.

Django's entire philosophy is based on *loose coupling*, which is the underlying
philosophy of MVC. We will be discussing loose coupling and MVC in much more
detail as we go along, but if you don't know much about MVC, then you best not
skip the second half of this chapter, because understanding MVC will make
understanding Django *so* much easier.

Installing Django
=================

There are a few steps to installing Django, but they are all very straight forward. In this chapter, we'll walk you through how to install the framework and its few dependencies.

This chapter assumes you're installing Django on a
desktop/laptop machine and will be using the development server and SQLite to
run all the example code in this book.

This is by far the easiest, and best way to setup Django when you are first
starting out. If you do want to go to a more advanced installation of Django,
your options are covered in :Chapter 13 - Deploying Django: `/chapter_13.rst`, Chapter 22 - Complete
Installation Guide and Chapter 23 - Advanced Database Management.

Installing Python
-----------------

Django itself is written purely in Python, so the first step in installing the
framework is to make sure you have Python installed.

Python Versions
~~~~~~~~~~~~~~~

Django version 1.8 LTS works with Python version 2.7, 3.3 and 3.4. For each
version of Python, only the latest micro release (A.B.C) is supported.

.. admonition:: Which Python version should I use?

    Django 1.8 LTS works with the latest releases of both Python 2 and Python 3.

    If you are just trialling Django, it does not really matter - either will
    work as well as the other.

    **NOTE: All of the code samples in this book are written in Python 3**

    If, however, you are planning on eventually deploying code to a live
    website, Python 3 should be your first choice. The Python wiki puts the
    reason behind this very succintly:

    *Short version: Python 2.x is legacy, Python 3.x is the present and future
    of the language*

    Unless you have a very good reason to use Python 2 (e.g. legacy
    libraries), Python 3 is the way to go.

Installation
~~~~~~~~~~~~

If you're on Linux or Mac OS X, you probably have Python already installed.
Type ``python`` at a command prompt (or in Applications/Utilities/Terminal, in
OS X). If you see something like this, then Python is installed::

    Python 2.7.5 (default, June 27 2015, 13:20:20)
    [GCC x.x.x] on xxx
    Type "help", "copyright", "credits" or "license" for more information.
    >>>

Otherwise, you'll need to download and install Python. It's fast and easy, and
detailed instructions are available at http://www.python.org/download/

**WARNING**: You can see that, in the above example, Python interactive mode is
running Python 2.7. This is a trap for inexperienced users. On Linux and
Mac OS X machines, it is common for both Python 2 and Python 3 to be
installed. If your system is like this, you need to type ``python3`` in
front of all your commands, rather than ``python`` to run Django with Python 3.


Installing Django
-----------------

.. admonition:: Using ``virtualenv``

    Before you install Django, it is worth considering whether you want to work
    within a virtual environment while you learn Django. 
    
    ``virtualenv`` is a Python tool that is used to create isolated Python
    environments. It is easy to set up and will ensure that any other
    applications on your computer that depend on Python don't get messed up if
    you accidentally overwrite something important. Setup and use of
    ``virtualenv`` is detailed in Chapter 22.

At any given time, two distinct versions of Django are available to you: the
latest official release and the bleeding-edge development version. The version you
decide to install depends on your priorities. Do you want a stable and tested
version of Django, or do you want a version containing the latest features,
perhaps so you can contribute to Django itself, at the expense of stability?

This book only deals with installing an official release of Django (in this
case Django 1.8 LTS). Instructions on installing the development version can
be found in Chapter 22. It is recommended that you stick
with the latest official release, however it's important to know the
development version exists as it is often mentioned in the Django
documentation and by members of the community.

You've got two easy options to install Django:

#. Install a version of Django provided by your operating system distribution.

#. Install an official release from the Django Project website.

Installing OS Distribution Version
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Many third-party distributors are now providing versions of Django integrated
with their package-management systems. These can make installation and upgrading
much easier for users of Django since the integration includes the ability to
automatically install dependencies (like database adapters) that Django
requires.

Typically, these packages are based on the latest stable release of Django, but not always. If your distro version is 1.8 or later, you are ok - all the code in this book should work (check the release notes if you are using a later version though, some functions are deprecated over time.). If your distro uses a version of Django older than 1.8, you will need to install an official release from the Django Project website.

If you're using Linux or a Unix installation, such as OpenSolaris,
check with your distributor to see if they already package Django. If
you're using a Linux distro and don't know how to find out if a package
is available, then now is a good time to learn.  The Django Wiki contains
a list of `Third Party Distributions`_ to help you out.

.. _`Third Party Distributions`: https://code.djangoproject.com/wiki/Distributions


Installing an Official Release
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The recommended way to install Django is with `pip`. Using `pip` to install
Django is an easy two step process:

1. Install pip_. The easiest is to use the `standalone pip installer`_. If your
   distribution already has ``pip`` installed, you might need to update it if
   it's outdated. (If it's outdated, you'll know because installation won't
   work.)

2. If you're using Linux, Mac OS X or some other flavor of Unix, enter the
   command ``sudo pip install Django`` at the shell prompt. If you're using
   Windows, start a command shell with administrator privileges and run
   the command ``pip install Django``. This will install Django in your Python
   installation's ``site-packages`` directory.

.. _pip: http://www.pip-installer.org/
.. _standalone pip installer: http://www.pip-installer.org/en/latest/installing.html#install-pip

There are other ways to install Django that are not covered here. If you have
previously experimented with Django without using `pip` you will also need to
uninstall any old versions of Django. For more information, see the Complete
Installation Guide in Chapter 22.

Testing the Django installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For some post-installation positive feedback, take a moment to test whether the
installation worked. In a command shell, change into your home directory and start the
Python interactive interpreter by typing ``python`` (or ``python3`` if your
system has two versions of Python installed). If the installation was
successful, you should be able to import the module ``django``:

    >>> import django
    >>> print(django.get_version())
    1.8.2

**NOTE:** You may have another version of Django installed.

.. admonition:: Interactive Interpreter Examples

    The Python interactive interpreter is a command-line program that lets you
    write a Python program interactively. To start it, run the command
    ``python`` or ``python3`` at the command line.

    Throughout this book, we feature example Python interactive interpreter
    sessions. You can recognize these examples by the triple
    greater-than signs (``>>>``), which designate the interpreter's prompt. If
    you're copying examples from this book, don't copy those greater-than signs.

    Multiline statements in the interactive interpreter are padded with three
    dots (``...``). For example::

        >>> print ("""This is a
        ... string that spans
        ... three lines.""")
        This is a
        string that spans
        three lines.
        >>> def my_function(value):
        ...     print (value)
        >>> my_function('hello')
        hello

    Those three dots at the start of the additional lines are inserted by the
    Python shell -- don't type them in. They are included to be faithful to
    the actual output of the interpreter. If you copy any examples from
    this book while following along, don't copy those dots.

Setting Up a Database
---------------------

This step is not necessary in order to complete any of the examples in this
book. Django comes with SQLite installed by default. SQLite requires no
configuration on your part.

If you would like to work with a "large" database engine like PostgreSQL, MySQL, or Oracle, see 
Chapter 23.

Starting a Project
------------------

Once you've installed Python, Django and (optionally) your database
server/library, you can take the first step in developing a Django application
by creating a *project*.

A project is a collection of settings for an instance of Django, including
database configuration, Django-specific options and application-specific
settings.

If this is your first time using Django, you'll have to take care of some
initial setup. Namely, you'll need to auto-generate some code that establishes a
Django `project` -- a collection of settings for an instance of Django,
including database configuration, Django-specific options and
application-specific settings.

From the command line, change into a directory where you'd like to store your
code, then run the following command:

.. code-block:: bash

   $ django-admin startproject mysite

This will create a ``mysite`` directory in your current directory. 

.. note::

    You'll need to avoid naming projects after built-in Python or Django
    components. In particular, this means you should avoid using names like
    ``django`` (which will conflict with Django itself) or ``test`` (which
    conflicts with a built-in Python package).

.. admonition:: Where should this code live?

    If your background is in plain old PHP (with no use of modern frameworks),
    you're probably used to putting code under the Web server's document root
    (in a place such as ``/var/www``). With Django, you don't do that. It's
    not a good idea to put any of this Python code within your Web server's
    document root, because it risks the possibility that people may be able
    to view your code over the Web. That's not good for security.

    Put your code in some directory **outside** of the document root, such as
    ``/home/mycode``.

	If you are following along and using the development server, this does not
	matter right now, but it is important that you remember this when you go to
	deploy your Django project to a production server.

Let's look at what `startproject` created::

    mysite/
        manage.py
        mysite/
            __init__.py
            settings.py
            urls.py
            wsgi.py

These files are:

* The outer ``mysite/`` root directory is just a container for your
  project. Its name doesn't matter to Django; you can rename it to anything
  you like.

* ``manage.py``: A command-line utility that lets you interact with this
  Django project in various ways. You can read all the details about
  ``manage.py`` in Appendix F. 

* The inner ``mysite/`` directory is the actual Python package for your
  project. Its name is the Python package name you'll need to use to import
  anything inside it (e.g. ``mysite.urls``).

* ``mysite/__init__.py``: An empty file that tells Python that this
  directory should be considered a Python package. (Read `more about
  packages`_ in the official Python docs if you're a Python beginner.)

* ``mysite/settings.py``: Settings/configuration for this Django
  project. Appendix D will tell you all about how settings
  work.

* ``mysite/urls.py``: The URL declarations for this Django project; a
  "table of contents" of your Django-powered site. You can read more about
  URLs in Chapters 2 and 7.

* ``mysite/wsgi.py``: An entry-point for WSGI-compatible web servers to
  serve your project. See Chapter 13 for more details.

.. _more about packages: https://docs.python.org/tutorial/modules.html#packages

Django settings
---------------

Now, edit ``mysite/settings.py``. It's a normal Python module with
module-level variables representing Django settings.

First step while you're editing ``mysite/settings.py``, is to set ``TIME_ZONE`` to
your time zone.

Note the ``INSTALLED_APPS`` setting at the top of the file. That
holds the names of all Django applications that are activated in this Django
instance. Apps can be used in multiple projects, and you can package and
distribute them for use by others in their projects.

By default, ``INSTALLED_APPS`` contains the following apps, all of which
come with Django:

* ``django.contrib.admin`` -- The admin site. 

* ``django.contrib.auth`` -- An authentication system.

* ``django.contrib.contenttypes`` -- A framework for content types.

* ``django.contrib.sessions`` -- A session framework.

* ``django.contrib.messages`` -- A messaging framework.

* ``django.contrib.staticfiles`` -- A framework for managing
  static files.

These applications are included by default as a convenience for the common case.

Some of these applications makes use of at least one database table, though,
so we need to create the tables in the database before we can use them. To do
that, run the following command:

.. code-block:: bash

    $ python manage.py migrate

The ``migrate`` command looks at the ``INSTALLED_APPS`` setting
and creates any necessary database tables according to the database settings
in your ``mysite/settings.py`` file and the database migrations shipped
with the app (we'll cover those later). You'll see a message for each
migration it applies. 

The development server
----------------------

Let's verify your Django project works. Change into the outer ``mysite`` directory, if
you haven't already, and run the following commands:

.. code-block:: bash

   $ python manage.py runserver

You'll see the following output on the command line:

.. parsed-literal::

    Performing system checks...

    0 errors found
    June 27, 2015 - 15:50:53
    Django version 1.8.2, using settings 'mysite.settings'
    Starting development server at http://127.0.0.1:8000/
    Quit the server with CONTROL-C.

You've started the Django development server, a lightweight Web server written
purely in Python. We've included this with Django so you can develop things
rapidly, without having to deal with configuring a production server -- such as
Apache -- until you're ready for production.

Now's a good time to note: **don't** use this server in anything resembling a
production environment. **It's intended only for use while developing**. 

Now that the server's running, visit http://127.0.0.1:8000/ with your Web
browser. You'll see a "Welcome to Django" page, in pleasant, light-blue pastel.
It worked!

.. figure:: graphics/chapter_01/welcome2django.png

   Figure 1-1. Django's welcome page

Automatic reloading of `runserver`
----------------------------------

The development server automatically reloads Python code for each request
as needed. You don't need to restart the server for code changes to take
effect. However, some actions like adding files don't trigger a restart,
so you'll have to restart the server in these cases.

The Model-View-Controller (MVC) design pattern
==============================================

MVC has been around as a concept for a long time, but has seen exponential
growth since the advent of the Internet because it is the best way to design
client-server applications. All of the best web frameworks are built around the
MVC concept. At the risk of starting a flame war, I contest that if you are not
using MVC to design web apps, you are doing it wrong.

As concept, the MVC design pattern is really simple to understand:

* The **model(M)** is a model or representation of your data. It is
  not the actual data, but an interface to the data. The model allows you to
  pull data from your database without having to know the intricacies of the
  underlying database. The model usually also provides an *abstraction* layer
  with your database, so that you can use the same model with multiple databases.

* The **view(V)** is what you see. It is the presentation layer for your model.
  On your computer, the view is what you see in the browser for a Web app, or the UI
  for a desktop app. The view also provides an interface to collect user input.

* The **controller(C)** controls the flow of information between the model and
  the view. It uses programmed logic to decide what information is pulled from
  the database via the model and what information is passed to the view. It also
  gets information from the user via the view and implements business logic:
  either by changing the view, or modifying data through the model, or both.

Where it gets difficult is the vastly different interpretation of what actually
happens at each layer - different frameworks implement the same functionality in
different ways. One framework "guru" might say a certain function belongs in a view, while an
other might vehemently defend the need for it to be in the controller.

You, as a budding programmer who Gets Stuff Done, do not have to care about this
because in the end, it *doesn't matter*. As long as you understand how Django
implements the MVC pattern, you are free to move on and get some real work done.
Although, watching a flame war in a comment thread can be a highly amusing
distraction...

Django follows the MVC pattern closely, however it does implement it's own logic
in the implementation. Because the "C" is handled by the framework itself and
most of the excitement in Django happens in models, templates and views, Django
is often referred to as an *MTV framework*. 

In the MTV development pattern:

* *M* stands for "Model," the data access layer. This layer contains
  anything and everything about the data: how to access it, how to validate
  it, which behaviors it has, and the relationships between the data. We will be
  looking closely at Django's models in Chapter 4.

* *T* stands for "Template," the presentation layer. This layer contains
  presentation-related decisions: how something should be displayed on a
  Web page or other type of document. We will explore Django's templates in
  Chapter 3.

* *V* stands for "View," the business logic layer. This layer contains the
  logic that access the model and defers to the appropriate template(s).
  You can think of it as the bridge between models and templates. We will be
  checking out Django's views in the next chapter.

This is probably the only unfortunate bit of naming in Django, because Django's
view is more like the controller in MVC, and MVC's view is actually a Template in
Django. It is a little confusing at first, but as a programmer getting a job
done, you really won't care for long. It is only a problem for those of us who
have to teach it. 

Oh, and to the flamers of course.

What's Next?
============

Now that you have everything installed and the development server running,
you're ready to move on to Django views and learning the basics of serving Web pages with Django.

