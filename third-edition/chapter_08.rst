=============================
Chapter 8: Advanced Templates
=============================

Although most of your interactions with Django's template language will be in
the role of template author, you may want to customize and extend the template
engine -- either to make it do something it doesn't already do, or to make your
job easier in some other way.

This chapter delves deep into the guts of Django's template system. It covers
what you need to know if you plan to extend the system or if you're just
curious about how it works. It also covers the auto-escaping feature, a
security measure you'll no doubt notice over time as you continue to use
Django.

Template Language Review
========================

First, let's quickly review a number of terms introduced in Chapter 3:

* A *template* is a text document, or a normal Python string, that is
  marked up using the Django template language. A template can contain
  template tags and variables.

* A *template tag* is a symbol within a template that does something. This
  definition is deliberately vague. For example, a template tag can produce
  content, serve as a control structure (an ``if`` statement or ``for``
  loop), grab content from a database, or enable access to other template
  tags.

  Template tags are surrounded by ``{%`` and ``%}``::

      {% if is_logged_in %}
          Thanks for logging in!
      {% else %}
          Please log in.
      {% endif %}

* A *variable* is a symbol within a template that outputs a value.

  Variable tags are surrounded by ``{{`` and ``}}``::

      My first name is {{ first_name }}. My last name is {{ last_name }}.

* A *context* is a `name->value` mapping (similar to a Python
  dictionary) that is passed to a template.

* A template *renders* a context by replacing the variable "holes" with
  values from the context and executing all template tags.

For more details about the basics of these terms, refer back to Chapter 3.

The rest of this chapter discusses ways of extending the template engine. First,
though, let's take a quick look at a few internals left out of Chapter 3 for
simplicity.

RequestContext and Context Processors
=====================================

When rendering a template, you need a context. This can be an instance of
``django.template.Context``, but Django also comes with a subclass,
``django.template.RequestContext``, that acts slightly differently.
``RequestContext`` adds a bunch of variables to your template context by
default -- things like the ``HttpRequest`` object or information about the
currently logged-in user. The ``render()`` shortcut creates a ``RequestContext`` 
unless it is passed a different context instance explicitly.


Use ``RequestContext`` when you don't want to have to specify the same set of
variables in a series of templates. For example, consider these two views::

    from django.template import loader, Context

    def view_1(request):
        # ...
        t = loader.get_template('template1.html')
        c = Context({
            'app': 'My app',
            'user': request.user,
            'ip_address': request.META['REMOTE_ADDR'],
            'message': 'I am view 1.'
        })
        return t.render(c)

    def view_2(request):
        # ...
        t = loader.get_template('template2.html')
        c = Context({
            'app': 'My app',
            'user': request.user,
            'ip_address': request.META['REMOTE_ADDR'],
            'message': 'I am the second view.'
        })
        return t.render(c)

(Note that we're deliberately *not* using the ``render()`` shortcut
in these examples -- we're manually loading the templates, constructing the
context objects and rendering the templates. We're "spelling out" all of the
steps for the purpose of clarity.)

Each view passes the same three variables -- ``app``, ``user`` and
``ip_address`` -- to its template. Wouldn't it be nice if we could remove that
redundancy?

``RequestContext`` and **context processors** were created to solve this
problem. Context processors let you specify a number of variables that get set
in each context automatically -- without you having to specify the variables in
each ``render()`` call. The catch is that you have to use
``RequestContext`` instead of ``Context`` when you render a template.

The most low-level way of using context processors is to create some processors
and pass them to ``RequestContext``. Here's how the above example could be
written with context processors::

    from django.template import loader, RequestContext

    def custom_proc(request):
        "A context processor that provides 'app', 'user' and 'ip_address'."
        return {
            'app': 'My app',
            'user': request.user,
            'ip_address': request.META['REMOTE_ADDR']
        }

    def view_1(request):
        # ...
        t = loader.get_template('template1.html')
        c = RequestContext(request, {'message': 'I am view 1.'},
                processors=[custom_proc])
        return t.render(c)

    def view_2(request):
        # ...
        t = loader.get_template('template2.html')
        c = RequestContext(request, {'message': 'I am the second view.'},
                processors=[custom_proc])
        return t.render(c)

Let's step through this code:

* First, we define a function ``custom_proc``. This is a context processor
  -- it takes an ``HttpRequest`` object and returns a dictionary of
  variables to use in the template context. That's all it does.

* We've changed the two view functions to use ``RequestContext`` instead
  of ``Context``. There are two differences in how the context is
  constructed. One, ``RequestContext`` requires the first argument to be an
  ``HttpRequest`` object -- the one that was passed into the view function
  in the first place (``request``). Two, ``RequestContext`` takes an
  optional ``processors`` argument, which is a list or tuple of context
  processor functions to use. Here, we pass in ``custom_proc``, the custom
  processor we defined above.

* Each view no longer has to include ``app``, ``user`` or ``ip_address`` in
  its context construction, because those are provided by ``custom_proc``.

* Each view *still* has the flexibility to introduce any custom template
  variables it might need. In this example, the ``message`` template
  variable is set differently in each view.

In Chapter 3, we introduced the ``render()`` shortcut, which saves
you from having to call ``loader.get_template()``, then create a ``Context``,
then call the ``render()`` method on the template. In order to demonstrate the
lower-level workings of context processors, the above examples didn't use
``render()``, . But it's possible -- and preferable -- to use
context processors with ``render()``. Do this with the
``context_instance`` argument, like so::

    from django.shortcuts import render
    from django.template import RequestContext

    def custom_proc(request):
        "A context processor that provides 'app', 'user' and 'ip_address'."
        return {
            'app': 'My app',
            'user': request.user,
            'ip_address': request.META['REMOTE_ADDR']
        }

    def view_1(request):
        # ...
        return render(request, 'template1.html',
            {'message': 'I am view 1.'},
            context_instance=RequestContext(request, processors=[custom_proc]))

    def view_2(request):
        # ...
        return render(request, 'template2.html',
            {'message': 'I am the second view.'},
            context_instance=RequestContext(request, processors=[custom_proc]))

Here, we've trimmed down each view's template rendering code to a single
(wrapped) line.

This is an improvement, but, evaluating the conciseness of this code, we have
to admit we're now almost overdosing on the *other* end of the spectrum. We've
removed redundancy in data (our template variables) at the cost of adding
redundancy in code (in the ``processors`` call). Using context processors
doesn't save you much typing if you have to type ``processors`` all the time.

For that reason, Django provides support for *global* context processors. The
``context_processors`` setting (in your ``settings.py``) designates
which context processors should *always* be applied to ``RequestContext``. This
removes the need to specify ``processors`` each time you use
``RequestContext``.

By default, ``context_processors`` is set to the following::

    'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],

This setting is a list of callables that use the same interface as our
``custom_proc`` function above -- functions that take a request object as their
argument and return a dictionary of items to be merged into the context. Note
that the values in ``context_processors`` are specified as *strings*,
which means the processors are required to be somewhere on your Python path
(so you can refer to them from the setting).

Each processor is applied in order. That is, if one processor adds a variable
to the context and a second processor adds a variable with the same name, the
second will override the first.

Django provides a number of simple context processors, including the ones that
are enabled by default:

auth
----

``django.contrib.auth.context_processors.auth``

If this processor is enabled, every ``RequestContext`` will contain these
variables:

* ``user`` -- An ``auth.User`` instance representing the currently
  logged-in user (or an ``AnonymousUser`` instance, if the client isn't
  logged in).

* ``perms`` -- An instance of
  ``django.contrib.auth.context_processors.PermWrapper``, representing the
  permissions that the currently logged-in user has.

.. currentmodule:: django.template.context_processors

debug
-----

``django.template.context_processors.debug``

If this processor is enabled, every ``RequestContext`` will contain these two
variables -- but only if your ``DEBUG`` setting is set to ``True`` and
the request's IP address (``request.META['REMOTE_ADDR']``) is in the
``INTERNAL_IPS`` setting:

* ``debug`` -- ``True``. You can use this in templates to test whether
  you're in ``DEBUG`` mode.
* ``sql_queries`` -- A list of ``{'sql': ..., 'time': ...}`` dictionaries,
  representing every SQL query that has happened so far during the request
  and how long it took. The list is in order by query and lazily generated
  on access.

i18n
----

``django.template.context_processors.i18n``

If this processor is enabled, every ``RequestContext`` will contain these two
variables:

* ``LANGUAGES`` -- The value of the ``LANGUAGES`` setting.
* ``LANGUAGE_CODE`` -- ``request.LANGUAGE_CODE``, if it exists. Otherwise,
  the value of the ``LANGUAGE_CODE`` setting.

media
-----

``django.template.context_processors.media``

If this processor is enabled, every ``RequestContext`` will contain a variable
``MEDIA_URL``, providing the value of the ``MEDIA_URL`` setting.

static
------

``django.template.context_processors.static``

If this processor is enabled, every ``RequestContext`` will contain a variable
``STATIC_URL``, providing the value of the ``STATIC_URL`` setting.

csrf
----

``django.template.context_processors.csrf``

This processor adds a token that is needed by the ``csrf_token`` template
tag for protection against Cross Site Request Forgeries (see chapter 21).

request
-------

``django.template.context_processors.request``

If this processor is enabled, every ``RequestContext`` will contain a variable
``request``, which is the current :class:`~django.http.HttpRequest`.

messages
--------

``django.contrib.messages.context_processors.messages``

If this processor is enabled, every ``RequestContext`` will contain these two
variables:

* ``messages`` -- A list of messages (as strings) that have been set
  via the messages framework (see Appendix H).
* ``DEFAULT_MESSAGE_LEVELS`` -- A mapping of the message level names to
  their numeric value.

Guidelines for Writing Your Own Context Processors
==================================================

A context processor has a very simple interface: It's just a Python function
that takes one argument, an :class:`~django.http.HttpRequest` object, and
returns a dictionary that gets added to the template context. Each context
processor *must* return a dictionary.

Here are a few tips for rolling your own:

* Make each context processor responsible for the smallest subset of
  functionality possible. It's easy to use multiple processors, so you
  might as well split functionality into logical pieces for future reuse.

* Keep in mind that any context processor in ``TEMPLATE_CONTEXT_PROCESSORS``
  will be available in *every* template powered by that settings file, so
  try to pick variable names that are unlikely to conflict with variable
  names your templates might be using independently. As variable names are
  case-sensitive, it's not a bad idea to use all caps for variables that a
  processor provides.

* Custom context processors can live anywhere in your code base. All Django
  cares about is that your custom context processors are pointed to by the
  ``'context_processors'`` option in your ``TEMPLATES`` setting — or
  the ``context_processors`` argument of :class:`~django.template.Engine` if
  you're using it directly.  With that said, the convention is to save them in
  a file called ``context_processors.py`` within your app or project.

Automatic HTML Escaping
=======================

When generating HTML from templates, there's always a risk that a variable will
include characters that affect the resulting HTML. For example, consider this
template fragment::

    Hello, {{ name }}.

At first, this seems like a harmless way to display a user's name, but consider
what would happen if the user entered his name as this::

    <script>alert('hello')</script>

With this name value, the template would be rendered as::

    Hello, <script>alert('hello')</script>

...which means the browser would pop-up a JavaScript alert box!

Similarly, what if the name contained a ``'<'`` symbol, like this?

::

    <b>username

That would result in a rendered template like this::

    Hello, <b>username

...which, in turn, would result in the remainder of the Web page being bolded!

Clearly, user-submitted data shouldn't be trusted blindly and inserted directly
into your Web pages, because a malicious user could use this kind of hole to
do potentially bad things. This type of security exploit is called a
Cross Site Scripting (XSS) attack. (For more on security, see Chapter 21.)

To avoid this problem, you have two options:

* One, you can make sure to run each untrusted variable through the
  ``escape`` filter, which converts potentially harmful HTML characters to
  unharmful ones. This was the default solution in Django for its first few
  years, but the problem is that it puts the onus on *you*, the developer /
  template author, to ensure you're escaping everything. It's easy to forget
  to escape data.

* Two, you can take advantage of Django's automatic HTML escaping. The
  remainder of this section describes how auto-escaping works.

By default in Django, every template automatically escapes the output
of every variable tag. Specifically, these five characters are
escaped:

* ``<`` is converted to ``&lt;``
* ``>`` is converted to ``&gt;``
* ``'`` (single quote) is converted to ``&#39;``
* ``"`` (double quote) is converted to ``&quot;``
* ``&`` is converted to ``&amp;``

Again, we stress that this behavior is on by default. If you're using Django's
template system, you're protected.

How to Turn it Off
------------------

If you don't want data to be auto-escaped, on a per-site, per-template level or
per-variable level, you can turn it off in several ways.

Why would you want to turn it off? Because sometimes, template variables
contain data that you *intend* to be rendered as raw HTML, in which case you
don't want their contents to be escaped. For example, you might store a blob of
trusted HTML in your database and want to embed that directly into your
template. Or, you might be using Django's template system to produce text that
is *not* HTML -- like an e-mail message, for instance.

For Individual Variables
------------------------

To disable auto-escaping for an individual variable, use the ``safe`` filter::

    This will be escaped: {{ data }}
    This will not be escaped: {{ data|safe }}

Think of *safe* as shorthand for *safe from further escaping* or *can be
safely interpreted as HTML*. In this example, if ``data`` contains ``'<b>'``,
the output will be::

    This will be escaped: &lt;b&gt;
    This will not be escaped: <b>

For Template Blocks
-------------------

To control auto-escaping for a template, wrap the template (or just a
particular section of the template) in the ``autoescape`` tag, like so::

    {% autoescape off %}
        Hello {{ name }}
    {% endautoescape %}

The ``autoescape`` tag takes either ``on`` or ``off`` as its argument. At
times, you might want to force auto-escaping when it would otherwise be
disabled. Here is an example template::

    Auto-escaping is on by default. Hello {{ name }}

    {% autoescape off %}
        This will not be auto-escaped: {{ data }}.

        Nor this: {{ other_data }}
        {% autoescape on %}
            Auto-escaping applies again: {{ name }}
        {% endautoescape %}
    {% endautoescape %}

The auto-escaping tag passes its effect on to templates that extend the
current one as well as templates included via the ``include`` tag, just like
all block tags. For example:

.. code-block:: python

    # base.html

    {% autoescape off %}
    <h1>{% block title %}{% endblock %}</h1>
    {% block content %}
    {% endblock %}
    {% endautoescape %}

.. code-block:: python
   
    # child.html

    {% extends "base.html" %}
    {% block title %}This &amp; that{% endblock %}
    {% block content %}{{ greeting }}{% endblock %}

Because auto-escaping is turned off in the base template, it will also be
turned off in the child template, resulting in the following rendered
HTML when the ``greeting`` variable contains the string ``<b>Hello!</b>``::

    <h1>This & that</h1>
    <b>Hello!</b>

Generally, template authors don't need to worry about auto-escaping very much.
Developers on the Python side (people writing views and custom filters) need to
think about the cases in which data shouldn't be escaped, and mark data
appropriately, so things work in the template.

If you're creating a template that might be used in situations where you're
not sure whether auto-escaping is enabled, then add an ``escape`` filter to any
variable that needs escaping. When auto-escaping is on, there's no danger of
the ``escape`` filter *double-escaping* data -- the ``escape`` filter does not
affect auto-escaped variables.

Automatic Escaping of String Literals in Filter Arguments
---------------------------------------------------------

As we mentioned earlier, filter arguments can be strings::

    {{ data|default:"This is a string literal." }}

All string literals are inserted *without* any automatic escaping into the
template -- they act as if they were all passed through the ``safe`` filter.
The reasoning behind this is that the template author is in control of what
goes into the string literal, so they can make sure the text is correctly
escaped when the template is written.

This means you would write ::

    {{ data|default:"3 &lt; 2" }}

...rather than ::

    {{ data|default:"3 < 2" }}  <-- Bad! Don't do this.

This doesn't affect what happens to data coming from the variable itself.
The variable's contents are still automatically escaped, if necessary, because
they're beyond the control of the template author.

Inside Template Loading
=======================

Generally, you'll store templates in files on your filesystem rather than
using the low-level :class:`~django.template.Template` API yourself. Save
templates in a directory specified as a **template directory**.

Django searches for template directories in a number of places, depending on
your template loading settings (see "Loader types" below), but the most basic
way of specifying template directories is by using the ``DIRS`` option.

The ``DIRS`` option
-------------------

Tell Django what your template directories are by using the ``DIRS`` option in the ``TEMPLATES`` setting in your settings
file — or the ``dirs`` argument of :class:`~django.template.Engine`. This
should be set to a list of strings that contain full paths to your template
directories::

    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [
                '/home/html/templates/lawrence.com',
                '/home/html/templates/default',
            ],
        },
    ]

Your templates can go anywhere you want, as long as the directories and
templates are readable by the Web server. They can have any extension you want,
such as ``.html`` or ``.txt``, or they can have no extension at all.

Note that these paths should use Unix-style forward slashes, even on Windows.

.. _template-loaders:

Loader types
------------

By default, Django uses a filesystem-based template loader, but Django comes
with a few other template loaders, which know how to load templates from other
sources.

Filesystem loader
~~~~~~~~~~~~~~~~~

.. class:: filesystem.Loader

    Loads templates from the filesystem, according to
    ``DIRS <TEMPLATES-DIRS>``.

    This loader is enabled by default. However it won't find any templates
    until you set ``DIRS <TEMPLATES-DIRS>`` to a non-empty list::

        TEMPLATES = [{
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [os.path.join(BASE_DIR, 'templates')],
        }]
        
App directories loader
~~~~~~~~~~~~~~~~~~~~~~

.. class:: app_directories.Loader

    Loads templates from Django apps on the filesystem. For each app in
    ``INSTALLED_APPS``, the loader looks for a ``templates``
    subdirectory. If the directory exists, Django looks for templates in there.

    This means you can store templates with your individual apps. This also
    makes it easy to distribute Django apps with default templates.

    For example, for this setting::

        INSTALLED_APPS = ['myproject.polls', 'myproject.music']

    ...then ``get_template('foo.html')`` will look for ``foo.html`` in these
    directories, in this order:

    * ``/path/to/myproject/polls/templates/``
    * ``/path/to/myproject/music/templates/``

    ... and will use the one it finds first.

    The order of ``INSTALLED_APPS`` is significant! For example, if you
    want to customize the Django admin, you might choose to override the
    standard ``admin/base_site.html`` template, from ``django.contrib.admin``,
    with your own ``admin/base_site.html`` in ``myproject.polls``. You must
    then make sure that your ``myproject.polls`` comes *before*
    ``django.contrib.admin`` in ``INSTALLED_APPS``, otherwise
    ``django.contrib.admin``’s will be loaded first and yours will be ignored.

    Note that the loader performs an optimization when it first runs:
    it caches a list of which ``INSTALLED_APPS`` packages have a
    ``templates`` subdirectory.

    You can enable this loader simply by setting
    ``APP_DIRS`` to ``True``::

        TEMPLATES = [{
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'APP_DIRS': True,
        }]
        
Other loaders
~~~~~~~~~~~~~
The remaining template loaders are:

* ``django.template.loaders.eggs.Loader``
* ``django.template.loaders.cached.Loader``
* ``django.template.loaders.locmem.Loader``

These loaders are disabled by default, but you can activate them
by adding a ``'loaders'`` option to your ``DjangoTemplates`` backend in the
``TEMPLATES`` setting or passing a ``loaders`` argument to
:class:`~django.template.Engine`. Details on these advanced loaders, as well as building your own
custom loader, can be found on the Django Project website.

Extending the Template System
=============================

Now that you understand a bit more about the internals of the template system,
let's look at how to extend the system with custom code.

Most template customization comes in the form of custom template tags and/or
filters. Although the Django template language comes with many built-in tags and
filters, you'll probably assemble your own libraries of tags and filters that
fit your own needs. Fortunately, it's quite easy to define your own
functionality.

Code layout
-----------

Custom template tags and filters must live inside a Django app. If they relate
to an existing app it makes sense to bundle them there; otherwise, you should
create a new app to hold them.

The app should contain a ``templatetags`` directory, at the same level as
``models.py``, ``views.py``, etc. If this doesn't already exist, create it -
don't forget the ``__init__.py`` file to ensure the directory is treated as a
Python package. After adding this module, you will need to restart your server
before you can use the tags or filters in templates.

Your custom tags and filters will live in a module inside the ``templatetags``
directory. The name of the module file is the name you'll use to load the tags
later, so be careful to pick a name that won't clash with custom tags and
filters in another app.

For example, if your custom tags/filters are in a file called
``poll_extras.py``, your app layout might look like this::

    polls/
        __init__.py
        models.py
        templatetags/
            __init__.py
            poll_extras.py
        views.py

And in your template you would use the following:

.. code-block:: html+django

    {% load poll_extras %}

The app that contains the custom tags must be in ``INSTALLED_APPS`` in
order for the ``{% load %}<load>`` tag to work. This is a security feature:
It allows you to host Python code for many template libraries on a single host
machine without enabling access to all of them for every Django installation.

There's no limit on how many modules you put in the ``templatetags`` package.
Just keep in mind that a ``{% load %}<load>`` statement will load
tags/filters for the given Python module name, not the name of the app.

To be a valid tag library, the module must contain a module-level variable
named ``register`` that is a ``template.Library`` instance, in which all the
tags and filters are registered. So, near the top of your module, put the
following::

    from django import template

    register = template.Library()

.. admonition:: Behind the scenes

    For a ton of examples, read the source code for Django's default filters
    and tags. They're in ``django/template/defaultfilters.py`` and
    ``django/template/defaulttags.py``, respectively.

    For more information on the ``load`` tag, read its documentation.


Creating a Template Library
---------------------------

Whether you're writing custom tags or filters, the first thing to do is to
create a **template library** -- a small bit of infrastructure Django can hook
into.

Creating a template library is a two-step process:

* First, decide which Django application should house the template library.
  If you've created an app via ``manage.py startapp``, you can put it in
  there, or you can create another app solely for the template library.
  We'd recommend the latter, because your filters might be useful to you
  in future projects.

  Whichever route you take, make sure to add the app to your
  ``INSTALLED_APPS`` setting. We'll explain this shortly.

* Second, create a ``templatetags`` directory in the appropriate Django
  application's package. It should be on the same level as ``models.py``,
  ``views.py``, and so forth. For example::

      books/
          __init__.py
          models.py
          templatetags/
          views.py

  Create two empty files in the ``templatetags`` directory: an ``__init__.py``
  file (to indicate to Python that this is a package containing Python code)
  and a file that will contain your custom tag/filter definitions. The name
  of the latter file is what you'll use to load the tags later. For example,
  if your custom tags/filters are in a file called ``poll_extras.py``, you'd
  write the following in a template::

      {% load poll_extras %}

  The ``{% load %}`` tag looks at your ``INSTALLED_APPS`` setting and only
  allows the loading of template libraries within installed Django
  applications. This is a security feature; it allows you to host Python
  code for many template libraries on a single computer without enabling
  access to all of them for every Django installation.

If you write a template library that isn't tied to any particular models/views,
it's valid and quite normal to have a Django application package that contains
only a ``templatetags`` package. There's no limit on how many modules you put in
the ``templatetags`` package. Just keep in mind that a ``{% load %}`` statement
will load tags/filters for the given Python module name, not the name of the
application.

Once you've created that Python module, you'll just have to write a bit of
Python code, depending on whether you're writing filters or tags.

To be a valid tag library, the module must contain a module-level variable named
``register`` that is an instance of ``template.Library``. This is the data
structure in which all the tags and filters are registered. So, near the top of
your module, insert the following::

    from django import template

    register = template.Library()

.. note::

    For a fine selection of examples, read the source code for Django's default
    filters and tags. They're in ``django/template/defaultfilters.py`` and
    ``django/template/defaulttags.py``, respectively. Some applications in
    ``django.contrib`` also contain template libraries.

Once you've created this ``register`` variable, you'll use it to create template
filters and tags.

Custom template tags and filters
================================

[TODO - check this section against new code above for relevance]

Django's template language comes with a wide variety of built-in
tags and filters designed to address the
presentation logic needs of your application. Nevertheless, you may
find yourself needing functionality that is not covered by the core
set of template primitives. You can extend the template engine by
defining custom tags and filters using Python, and then make them
available to your templates using the ``{% load %}<load>`` tag.

.. _howto-writing-custom-template-filters:

Writing Custom Template Filters
-------------------------------

Custom filters are just Python functions that take one or two arguments:

* The value of the variable (input) -- not necessarily a string.
* The value of the argument -- this can have a default value, or be left
  out altogether.

For example, in the filter ``{{ var|foo:"bar" }}``, the filter ``foo`` would be
passed the variable ``var`` and the argument ``"bar"``.

Since the template language doesn't provide exception handling, any exception
raised from a template filter will be exposed as a server error. Thus, filter
functions should avoid raising exceptions if there is a reasonable fallback
value to return. In case of input that represents a clear bug in a template,
raising an exception may still be better than silent failure which hides the
bug.

Here's an example filter definition::

    def cut(value, arg):
        """Removes all values of arg from the given string"""
        return value.replace(arg, '')

And here's an example of how that filter would be used:

.. code-block:: html+django

    {{ somevariable|cut:"0" }}

Most filters don't take arguments. In this case, just leave the argument out of
your function. Example::

    def lower(value): # Only one argument.
        """Converts a string into all lowercase"""
        return value.lower()

Registering custom filters
--------------------------

.. method:: django.template.Library.filter()

Once you've written your filter definition, you need to register it with
your ``Library`` instance, to make it available to Django's template language::

    register.filter('cut', cut)
    register.filter('lower', lower)

The ``Library.filter()`` method takes two arguments:

1. The name of the filter -- a string.
2. The compilation function -- a Python function (not the name of the
   function as a string).

You can use ``register.filter()`` as a decorator instead::

    @register.filter(name='cut')
    def cut(value, arg):
        return value.replace(arg, '')

    @register.filter
    def lower(value):
        return value.lower()

If you leave off the ``name`` argument, as in the second example above, Django
will use the function's name as the filter name.

Finally, ``register.filter()`` also accepts three keyword arguments,
``is_safe``, ``needs_autoescape``, and ``expects_localtime``. These arguments
are described in filters and auto-escaping and
filters and time zones below.

Template filters that expect strings
------------------------------------

.. method:: django.template.defaultfilters.stringfilter()

If you're writing a template filter that only expects a string as the first
argument, you should use the decorator ``stringfilter``. This will
convert an object to its string value before being passed to your function::

    from django import template
    from django.template.defaultfilters import stringfilter

    register = template.Library()

    @register.filter
    @stringfilter
    def lower(value):
        return value.lower()

This way, you'll be able to pass, say, an integer to this filter, and it
won't cause an ``AttributeError`` (because integers don't have ``lower()``
methods).

.. _filters-auto-escaping:

Filters and auto-escaping
-------------------------

When writing a custom filter, give some thought to how the filter will interact
with Django's auto-escaping behavior. Note that three types of strings can be
passed around inside the template code:

* **Raw strings** are the native Python ``str`` or ``unicode`` types. On
  output, they're escaped if auto-escaping is in effect and presented
  unchanged, otherwise.

* **Safe strings** are strings that have been marked safe from further
  escaping at output time. Any necessary escaping has already been done.
  They're commonly used for output that contains raw HTML that is intended
  to be interpreted as-is on the client side.

  Internally, these strings are of type ``SafeBytes`` or ``SafeText``.
  They share a common base class of ``SafeData``, so you can test
  for them using code like::

      if isinstance(value, SafeData):
          # Do something with the "safe" string.
          ...

* **Strings marked as "needing escaping"** are *always* escaped on
  output, regardless of whether they are in an ``autoescape`` block or
  not. These strings are only escaped once, however, even if auto-escaping
  applies.

  Internally, these strings are of type ``EscapeBytes`` or
  ``EscapeText``. Generally you don't have to worry about these; they
  exist for the implementation of the ``escape`` filter.

Template filter code falls into one of two situations:

1. Your filter does not introduce any HTML-unsafe characters (``<``, ``>``,
   ``'``, ``"`` or ``&``) into the result that were not already present. In
   this case, you can let Django take care of all the auto-escaping
   handling for you. All you need to do is set the ``is_safe`` flag to ``True``
   when you register your filter function, like so::

       @register.filter(is_safe=True)
       def myfilter(value):
           return value

   This flag tells Django that if a "safe" string is passed into your
   filter, the result will still be "safe" and if a non-safe string is
   passed in, Django will automatically escape it, if necessary.

   You can think of this as meaning "this filter is safe -- it doesn't
   introduce any possibility of unsafe HTML."

   The reason ``is_safe`` is necessary is because there are plenty of
   normal string operations that will turn a ``SafeData`` object back into
   a normal ``str`` or ``unicode`` object and, rather than try to catch
   them all, which would be very difficult, Django repairs the damage after
   the filter has completed.

   For example, suppose you have a filter that adds the string ``xx`` to
   the end of any input. Since this introduces no dangerous HTML characters
   to the result (aside from any that were already present), you should
   mark your filter with ``is_safe``::

       @register.filter(is_safe=True)
       def add_xx(value):
           return '%sxx' % value

   When this filter is used in a template where auto-escaping is enabled,
   Django will escape the output whenever the input is not already marked
   as "safe".

   By default, ``is_safe`` is ``False``, and you can omit it from any filters
   where it isn't required.

   Be careful when deciding if your filter really does leave safe strings
   as safe. If you're *removing* characters, you might inadvertently leave
   unbalanced HTML tags or entities in the result. For example, removing a
   ``>`` from the input might turn ``<a>`` into ``<a``, which would need to
   be escaped on output to avoid causing problems. Similarly, removing a
   semicolon (``;``) can turn ``&amp;`` into ``&amp``, which is no longer a
   valid entity and thus needs further escaping. Most cases won't be nearly
   this tricky, but keep an eye out for any problems like that when
   reviewing your code.

   Marking a filter ``is_safe`` will coerce the filter's return value to
   a string.  If your filter should return a boolean or other non-string
   value, marking it ``is_safe`` will probably have unintended
   consequences (such as converting a boolean False to the string
   'False').

2. Alternatively, your filter code can manually take care of any necessary
   escaping. This is necessary when you're introducing new HTML markup into
   the result. You want to mark the output as safe from further
   escaping so that your HTML markup isn't escaped further, so you'll need
   to handle the input yourself.

   To mark the output as a safe string, use
   :func:`django.utils.safestring.mark_safe`.

   Be careful, though. You need to do more than just mark the output as
   safe. You need to ensure it really *is* safe, and what you do depends on
   whether auto-escaping is in effect. The idea is to write filters than
   can operate in templates where auto-escaping is either on or off in
   order to make things easier for your template authors.

   In order for your filter to know the current auto-escaping state, set the
   ``needs_autoescape`` flag to ``True`` when you register your filter function.
   (If you don't specify this flag, it defaults to ``False``). This flag tells
   Django that your filter function wants to be passed an extra keyword
   argument, called ``autoescape``, that is ``True`` if auto-escaping is in
   effect and ``False`` otherwise.

   For example, let's write a filter that emphasizes the first character of
   a string::

      from django import template
      from django.utils.html import conditional_escape
      from django.utils.safestring import mark_safe

      register = template.Library()

      @register.filter(needs_autoescape=True)
      def initial_letter_filter(text, autoescape=None):
          first, other = text[0], text[1:]
          if autoescape:
              esc = conditional_escape
          else:
              esc = lambda x: x
          result = '<strong>%s</strong>%s' % (esc(first), esc(other))
          return mark_safe(result)

   The ``needs_autoescape`` flag and the ``autoescape`` keyword argument mean
   that our function will know whether automatic escaping is in effect when the
   filter is called. We use ``autoescape`` to decide whether the input data
   needs to be passed through ``django.utils.html.conditional_escape`` or not.
   (In the latter case, we just use the identity function as the "escape"
   function.) The ``conditional_escape()`` function is like ``escape()`` except
   it only escapes input that is **not** a ``SafeData`` instance. If a
   ``SafeData`` instance is passed to ``conditional_escape()``, the data is
   returned unchanged.

   Finally, in the above example, we remember to mark the result as safe
   so that our HTML is inserted directly into the template without further
   escaping.

   There's no need to worry about the ``is_safe`` flag in this case
   (although including it wouldn't hurt anything). Whenever you manually
   handle the auto-escaping issues and return a safe string, the
   ``is_safe`` flag won't change anything either way.

.. warning:: Avoiding XSS vulnerabilities when reusing built-in filters

    Be careful when reusing Django's built-in filters. You'll need to pass
    ``autoescape=True`` to the filter in order to get the proper autoescaping
    behavior and avoid a cross-site script vulnerability.

    For example, if you wanted to write a custom filter called
    ``urlize_and_linebreaks`` that combined the ``urlize`` and
    ``linebreaksbr`` filters, the filter would look like::

        from django.template.defaultfilters import linebreaksbr, urlize

        @register.filter
        def urlize_and_linebreaks(text):
            return linebreaksbr(urlize(text, autoescape=True), autoescape=True)

    Then:

    .. code-block:: html+django

        {{ comment|urlize_and_linebreaks }}

    would be equivalent to:

    .. code-block:: html+django

        {{ comment|urlize|linebreaksbr }}

.. _filters-timezones:

Filters and time zones
----------------------

If you write a custom filter that operates on :class:`~datetime.datetime`
objects, you'll usually register it with the ``expects_localtime`` flag set to
``True``::

    @register.filter(expects_localtime=True)
    def businesshours(value):
        try:
            return 9 <= value.hour < 17
        except AttributeError:
            return ''

When this flag is set, if the first argument to your filter is a time zone
aware datetime, Django will convert it to the current time zone before passing
it to your filter when appropriate, according to rules for time zones
conversions in templates.

Writing custom template tags
----------------------------

Tags are more complex than filters, because tags can do anything. Django
provides a number of shortcuts that make writing most types of tags easier.
First we'll explore those shortcuts, then explain how to write a tag from
scratch for those cases when the shortcuts aren't powerful enough.

.. _howto-custom-template-tags-simple-tags:

Simple tags
-----------

.. method:: django.template.Library.simple_tag()

Many template tags take a number of arguments -- strings or template variables
-- and return a result after doing some processing based solely on
the input arguments and some external information. For example, a
``current_time`` tag might accept a format string and return the time as a
string formatted accordingly.

To ease the creation of these types of tags, Django provides a helper function,
``simple_tag``. This function, which is a method of
``django.template.Library``, takes a function that accepts any number of
arguments, wraps it in a ``render`` function and the other necessary bits
mentioned above and registers it with the template system.

Our ``current_time`` function could thus be written like this::

    import datetime
    from django import template

    register = template.Library()

    @register.simple_tag
    def current_time(format_string):
        return datetime.datetime.now().strftime(format_string)

A few things to note about the ``simple_tag`` helper function:

* Checking for the required number of arguments, etc., has already been
  done by the time our function is called, so we don't need to do that.
* The quotes around the argument (if any) have already been stripped away,
  so we just receive a plain string.
* If the argument was a template variable, our function is passed the
  current value of the variable, not the variable itself.

If your template tag needs to access the current context, you can use the
``takes_context`` argument when registering your tag::

    @register.simple_tag(takes_context=True)
    def current_time(context, format_string):
        timezone = context['timezone']
        return your_get_current_time_method(timezone, format_string)

Note that the first argument *must* be called ``context``.

For more information on how the ``takes_context`` option works, see the section
on inclusion tags.

If you need to rename your tag, you can provide a custom name for it::

    register.simple_tag(lambda x: x - 1, name='minusone')

    @register.simple_tag(name='minustwo')
    def some_function(value):
        return value - 2

``simple_tag`` functions may accept any number of positional or keyword
arguments. For example::

    @register.simple_tag
    def my_tag(a, b, *args, **kwargs):
        warning = kwargs['warning']
        profile = kwargs['profile']
        ...
        return ...

Then in the template any number of arguments, separated by spaces, may be
passed to the template tag. Like in Python, the values for keyword arguments
are set using the equal sign ("``=``") and must be provided after the
positional arguments. For example:

.. code-block:: html+django

    {% my_tag 123 "abcd" book.title warning=message|lower profile=user.profile %}

.. _howto-custom-template-tags-inclusion-tags:

Inclusion tags
--------------

.. method:: django.template.Library.inclusion_tag()

Another common type of template tag is the type that displays some data by
rendering *another* template. For example, Django's admin interface uses custom
template tags to display the buttons along the bottom of the "add/change" form
pages. Those buttons always look the same, but the link targets change
depending on the object being edited -- so they're a perfect case for using a
small template that is filled with details from the current object. (In the
admin's case, this is the ``submit_row`` tag.)

These sorts of tags are called *inclusion tags*. Writing inclusion tags is
probably best demonstrated by example. Let's write a tag that produces a list
of books for a given ``Author`` object. We'll use the tag like this::

    {% books_for_author author %}

The result will be something like this::

    <ul>
        <li>The Cat In The Hat</li>
        <li>Hop On Pop</li>
        <li>Green Eggs And Ham</li>
    </ul>

First, we define the function that takes the argument and produces a
dictionary of data for the result. Notice that we need to return only a
dictionary, not anything more complex. This will be used as the context for
the template fragment::

    def books_for_author(author):
        books = Book.objects.filter(authors__id=author.id)
        return {'books': books}

Next, we create the template used to render the tag's output. Following our
example, the template is very simple::

    <ul>
    {% for book in books %}
        <li>{{ book.title }}</li>
    {% endfor %}
    </ul>

Finally, we create and register the inclusion tag by calling the
``inclusion_tag()`` method on a ``Library`` object.

Following our example, if the preceding template is in a file called
``book_snippet.html``, we register the tag like this::

    register.inclusion_tag('book_snippet.html')(books_for_author)

Sometimes, your inclusion tags need access to values from the parent template's
context. To solve this, Django provides a ``takes_context`` option for
inclusion tags. If you specify ``takes_context`` in creating an inclusion tag,
the tag will have no required arguments, and the underlying Python function
will have one argument: the template context as of when the tag was called.

For example, say you're writing an inclusion tag that will always be used in a
context that contains ``home_link`` and ``home_title`` variables that point
back to the main page. Here's what the Python function would look like::

    @register.inclusion_tag('link.html', takes_context=True)
    def jump_link(context):
        return {
            'link': context['home_link'],
            'title': context['home_title'],
        }

(Note that the first parameter to the function *must* be called ``context``.)

The template ``link.html`` might contain the following::

    Jump directly to <a href="{{ link }}">{{ title }}</a>.

Then, anytime you want to use that custom tag, load its library and call it
without any arguments, like so::

    {% jump_link %}

Assignment tags
---------------

.. method:: django.template.Library.assignment_tag()

To ease the creation of tags setting a variable in the context, Django provides
a helper function, ``assignment_tag``. This function works the same way as
:meth:`~django.template.Library.simple_tag` except that it stores the tag's
result in a specified context variable instead of directly outputting it.

Our earlier ``current_time`` function could thus be written like this::

    @register.assignment_tag
    def get_current_time(format_string):
        return datetime.datetime.now().strftime(format_string)

You may then store the result in a template variable using the ``as`` argument
followed by the variable name, and output it yourself where you see fit:

.. code-block:: html+django

    {% get_current_time "%Y-%m-%d %I:%M %p" as the_time %}
    <p>The time is {{ the_time }}.</p>

Advanced custom template tags
-----------------------------

Sometimes the basic features for custom template tag creation aren't enough.
Don't worry, Django gives you complete access to the internals required to build
a template tag from the ground up.

A quick overview
----------------

The template system works in a two-step process: compiling and rendering. To
define a custom template tag, you specify how the compilation works and how
the rendering works.

When Django compiles a template, it splits the raw template text into
''nodes''. Each node is an instance of ``django.template.Node`` and has
a ``render()`` method. A compiled template is, simply, a list of ``Node``
objects. When you call ``render()`` on a compiled template object, the template
calls ``render()`` on each ``Node`` in its node list, with the given context.
The results are all concatenated together to form the output of the template.

Thus, to define a custom template tag, you specify how the raw template tag is
converted into a ``Node`` (the compilation function), and what the node's
``render()`` method does.

Writing the compilation function
--------------------------------

For each template tag the template parser encounters, it calls a Python
function with the tag contents and the parser object itself. This function is
responsible for returning a ``Node`` instance based on the contents of the tag.

For example, let's write a full implementation of our simple template tag,
``{% current_time %}``, that displays the current date/time, formatted according
to a parameter given in the tag, in :func:`~time.strftime` syntax. It's a good
idea to decide the tag syntax before anything else. In our case, let's say the
tag should be used like this:

.. code-block:: html+django

    <p>The time is {% current_time "%Y-%m-%d %I:%M %p" %}.</p>

The parser for this function should grab the parameter and create a ``Node``
object::

    from django import template

    def do_current_time(parser, token):
        try:
            # split_contents() knows not to split quoted strings.
            tag_name, format_string = token.split_contents()
        except ValueError:
            raise template.TemplateSyntaxError("%r tag requires a single argument" % token.contents.split()[0])
        if not (format_string[0] == format_string[-1] and format_string[0] in ('"', "'")):
            raise template.TemplateSyntaxError("%r tag's argument should be in quotes" % tag_name)
        return CurrentTimeNode(format_string[1:-1])

Notes:

* ``parser`` is the template parser object. We don't need it in this
  example.

* ``token.contents`` is a string of the raw contents of the tag. In our
  example, it's ``'current_time "%Y-%m-%d %I:%M %p"'``.

* The ``token.split_contents()`` method separates the arguments on spaces
  while keeping quoted strings together. The more straightforward
  ``token.contents.split()`` wouldn't be as robust, as it would naively
  split on *all* spaces, including those within quoted strings. It's a good
  idea to always use ``token.split_contents()``.

* This function is responsible for raising
  ``django.template.TemplateSyntaxError``, with helpful messages, for
  any syntax error.

* The ``TemplateSyntaxError`` exceptions use the ``tag_name`` variable.
  Don't hard-code the tag's name in your error messages, because that
  couples the tag's name to your function. ``token.contents.split()[0]``
  will ''always'' be the name of your tag -- even when the tag has no
  arguments.

* The function returns a ``CurrentTimeNode`` with everything the node needs
  to know about this tag. In this case, it just passes the argument --
  ``"%Y-%m-%d %I:%M %p"``. The leading and trailing quotes from the
  template tag are removed in ``format_string[1:-1]``.

* The parsing is very low-level. The Django developers have experimented
  with writing small frameworks on top of this parsing system, using
  techniques such as EBNF grammars, but those experiments made the template
  engine too slow. It's low-level because that's fastest.

Writing the renderer
--------------------

The second step in writing custom tags is to define a ``Node`` subclass that
has a ``render()`` method.

Continuing the above example, we need to define ``CurrentTimeNode``::

    import datetime
    from django import template

    class CurrentTimeNode(template.Node):
        def __init__(self, format_string):
            self.format_string = format_string

        def render(self, context):
            return datetime.datetime.now().strftime(self.format_string)

Notes:

* ``__init__()`` gets the ``format_string`` from ``do_current_time()``.
  Always pass any options/parameters/arguments to a ``Node`` via its
  ``__init__()``.

* The ``render()`` method is where the work actually happens.

* ``render()`` should generally fail silently, particularly in a production
  environment where ``DEBUG`` and ``TEMPLATE_DEBUG`` are
  ``False``. In some cases however, particularly if ``TEMPLATE_DEBUG`` is
  ``True``, this method may raise an exception to make debugging easier. For
  example, several core tags raise ``django.template.TemplateSyntaxError``
  if they receive the wrong number or type of arguments.

Ultimately, this decoupling of compilation and rendering results in an
efficient template system, because a template can render multiple contexts
without having to be parsed multiple times.

Auto-escaping considerations
----------------------------

The output from template tags is **not** automatically run through the
auto-escaping filters. However, there are still a couple of things you should
keep in mind when writing a template tag.

If the ``render()`` function of your template stores the result in a context
variable (rather than returning the result in a string), it should take care
to call ``mark_safe()`` if appropriate. When the variable is ultimately
rendered, it will be affected by the auto-escape setting in effect at the
time, so content that should be safe from further escaping needs to be marked
as such.

Also, if your template tag creates a new context for performing some
sub-rendering, set the auto-escape attribute to the current context's value.
The ``__init__`` method for the ``Context`` class takes a parameter called
``autoescape`` that you can use for this purpose. For example::

    from django.template import Context

    def render(self, context):
        # ...
        new_context = Context({'var': obj}, autoescape=context.autoescape)
        # ... Do something with new_context ...

This is not a very common situation, but it's useful if you're rendering a
template yourself. For example::

    def render(self, context):
        t = context.engine.get_template('small_fragment.html')
        return t.render(Context({'var': obj}, autoescape=context.autoescape))

If we had neglected to pass in the current ``context.autoescape`` value to our
new ``Context`` in this example, the results would have *always* been
automatically escaped, which may not be the desired behavior if the template
tag is used inside a ``{% autoescape off %}<autoescape>`` block.

.. _template_tag_thread_safety:

Thread-safety considerations
----------------------------

Once a node is parsed, its ``render`` method may be called any number of times.
Since Django is sometimes run in multi-threaded environments, a single node may
be simultaneously rendering with different contexts in response to two separate
requests. Therefore, it's important to make sure your template tags are thread
safe.

To make sure your template tags are thread safe, you should never store state
information on the node itself. For example, Django provides a builtin
``cycle`` template tag that cycles among a list of given strings each time
it's rendered:

.. code-block:: html+django

    {% for o in some_list %}
        <tr class="{% cycle 'row1' 'row2' %}>
            ...
        </tr>
    {% endfor %}

A naive implementation of ``CycleNode`` might look something like this::

    import itertools
    from django import template

    class CycleNode(template.Node):
        def __init__(self, cyclevars):
            self.cycle_iter = itertools.cycle(cyclevars)

        def render(self, context):
            return next(self.cycle_iter)

But, suppose we have two templates rendering the template snippet from above at
the same time:

1. Thread 1 performs its first loop iteration, ``CycleNode.render()``
   returns 'row1'
2. Thread 2 performs its first loop iteration, ``CycleNode.render()``
   returns 'row2'
3. Thread 1 performs its second loop iteration, ``CycleNode.render()``
   returns 'row1'
4. Thread 2 performs its second loop iteration, ``CycleNode.render()``
   returns 'row2'

The CycleNode is iterating, but it's iterating globally. As far as Thread 1
and Thread 2 are concerned, it's always returning the same value. This is
obviously not what we want!

To address this problem, Django provides a ``render_context`` that's associated
with the ``context`` of the template that is currently being rendered. The
``render_context`` behaves like a Python dictionary, and should be used to
store ``Node`` state between invocations of the ``render`` method.

Let's refactor our ``CycleNode`` implementation to use the ``render_context``::

    class CycleNode(template.Node):
        def __init__(self, cyclevars):
            self.cyclevars = cyclevars

        def render(self, context):
            if self not in context.render_context:
                context.render_context[self] = itertools.cycle(self.cyclevars)
            cycle_iter = context.render_context[self]
            return next(cycle_iter)

Note that it's perfectly safe to store global information that will not change
throughout the life of the ``Node`` as an attribute. In the case of
``CycleNode``, the ``cyclevars`` argument doesn't change after the ``Node`` is
instantiated, so we don't need to put it in the ``render_context``. But state
information that is specific to the template that is currently being rendered,
like the current iteration of the ``CycleNode``, should be stored in the
``render_context``.

.. note::
    Notice how we used ``self`` to scope the ``CycleNode`` specific information
    within the ``render_context``. There may be multiple ``CycleNodes`` in a
    given template, so we need to be careful not to clobber another node's
    state information. The easiest way to do this is to always use ``self`` as
    the key into ``render_context``. If you're keeping track of several state
    variables, make ``render_context[self]`` a dictionary.

Registering the tag
-------------------

Finally, register the tag with your module's ``Library`` instance, as explained
in "Writing custom template filters" above. Example::

    register.tag('current_time', do_current_time)

The ``tag()`` method takes two arguments:

1. The name of the template tag -- a string. If this is left out, the
   name of the compilation function will be used.
2. The compilation function -- a Python function (not the name of the
   function as a string).

As with filter registration, it is also possible to use this as a decorator::

    @register.tag(name="current_time")
    def do_current_time(parser, token):
        ...

    @register.tag
    def shout(parser, token):
        ...

If you leave off the ``name`` argument, as in the second example above, Django
will use the function's name as the tag name.

Passing template variables to the tag
-------------------------------------

Although you can pass any number of arguments to a template tag using
``token.split_contents()``, the arguments are all unpacked as
string literals. A little more work is required in order to pass dynamic
content (a template variable) to a template tag as an argument.

While the previous examples have formatted the current time into a string and
returned the string, suppose you wanted to pass in a
:class:`~django.db.models.DateTimeField` from an object and have the template
tag format that date-time:

.. code-block:: html+django

    <p>This post was last updated at {% format_time blog_entry.date_updated "%Y-%m-%d %I:%M %p" %}.</p>

Initially, ``token.split_contents()`` will return three values:

1. The tag name ``format_time``.
2. The string ``'blog_entry.date_updated'`` (without the surrounding
   quotes).
3. The formatting string ``'"%Y-%m-%d %I:%M %p"'``. The return value from
   ``split_contents()`` will include the leading and trailing quotes for
   string literals like this.

Now your tag should begin to look like this::

    from django import template

    def do_format_time(parser, token):
        try:
            # split_contents() knows not to split quoted strings.
            tag_name, date_to_be_formatted, format_string = token.split_contents()
        except ValueError:
            raise template.TemplateSyntaxError("%r tag requires exactly two arguments" % token.contents.split()[0])
        if not (format_string[0] == format_string[-1] and format_string[0] in ('"', "'")):
            raise template.TemplateSyntaxError("%r tag's argument should be in quotes" % tag_name)
        return FormatTimeNode(date_to_be_formatted, format_string[1:-1])

You also have to change the renderer to retrieve the actual contents of the
``date_updated`` property of the ``blog_entry`` object.  This can be
accomplished by using the ``Variable()`` class in ``django.template``.

To use the ``Variable`` class, simply instantiate it with the name of the
variable to be resolved, and then call ``variable.resolve(context)``. So,
for example::

    class FormatTimeNode(template.Node):
        def __init__(self, date_to_be_formatted, format_string):
            self.date_to_be_formatted = template.Variable(date_to_be_formatted)
            self.format_string = format_string

        def render(self, context):
            try:
                actual_date = self.date_to_be_formatted.resolve(context)
                return actual_date.strftime(self.format_string)
            except template.VariableDoesNotExist:
                return ''

Variable resolution will throw a ``VariableDoesNotExist`` exception if it
cannot resolve the string passed to it in the current context of the page.

Setting a variable in the context
---------------------------------

The above examples simply output a value. Generally, it's more flexible if your
template tags set template variables instead of outputting values. That way,
template authors can reuse the values that your template tags create.

To set a variable in the context, just use dictionary assignment on the context
object in the ``render()`` method. Here's an updated version of
``CurrentTimeNode`` that sets a template variable ``current_time`` instead of
outputting it::

    import datetime
    from django import template

    class CurrentTimeNode2(template.Node):
        def __init__(self, format_string):
            self.format_string = format_string
        def render(self, context):
            context['current_time'] = datetime.datetime.now().strftime(self.format_string)
            return ''

Note that ``render()`` returns the empty string. ``render()`` should always
return string output. If all the template tag does is set a variable,
``render()`` should return the empty string.

Here's how you'd use this new version of the tag:

.. code-block:: html+django

    {% current_time "%Y-%M-%d %I:%M %p" %}<p>The time is {{ current_time }}.</p>

.. admonition:: Variable scope in context

    Any variable set in the context will only be available in the same
    ``block`` of the template in which it was assigned. This behavior is
    intentional; it provides a scope for variables so that they don't conflict
    with context in other blocks.

But, there's a problem with ``CurrentTimeNode2``: The variable name
``current_time`` is hard-coded. This means you'll need to make sure your
template doesn't use ``{{ current_time }}`` anywhere else, because the
``{% current_time %}`` will blindly overwrite that variable's value. A cleaner
solution is to make the template tag specify the name of the output variable,
like so:

.. code-block:: html+django

    {% current_time "%Y-%M-%d %I:%M %p" as my_current_time %}
    <p>The current time is {{ my_current_time }}.</p>

To do that, you'll need to refactor both the compilation function and ``Node``
class, like so::

    import re

    class CurrentTimeNode3(template.Node):
        def __init__(self, format_string, var_name):
            self.format_string = format_string
            self.var_name = var_name
        def render(self, context):
            context[self.var_name] = datetime.datetime.now().strftime(self.format_string)
            return ''

    def do_current_time(parser, token):
        # This version uses a regular expression to parse tag contents.
        try:
            # Splitting by None == splitting by spaces.
            tag_name, arg = token.contents.split(None, 1)
        except ValueError:
            raise template.TemplateSyntaxError("%r tag requires arguments" % token.contents.split()[0])
        m = re.search(r'(.*?) as (\w+)', arg)
        if not m:
            raise template.TemplateSyntaxError("%r tag had invalid arguments" % tag_name)
        format_string, var_name = m.groups()
        if not (format_string[0] == format_string[-1] and format_string[0] in ('"', "'")):
            raise template.TemplateSyntaxError("%r tag's argument should be in quotes" % tag_name)
        return CurrentTimeNode3(format_string[1:-1], var_name)

The difference here is that ``do_current_time()`` grabs the format string and
the variable name, passing both to ``CurrentTimeNode3``.

Finally, if you only need to have a simple syntax for your custom
context-updating template tag, you might want to consider using the
assignment tag shortcut we introduced above.

Parsing until another block tag
-------------------------------

Template tags can work in tandem. For instance, the standard
``{% comment %}<comment>`` tag hides everything until ``{% endcomment %}``.
To create a template tag such as this, use ``parser.parse()`` in your
compilation function.

Here's how a simplified ``{% comment %}`` tag might be implemented::

    def do_comment(parser, token):
        nodelist = parser.parse(('endcomment',))
        parser.delete_first_token()
        return CommentNode()

    class CommentNode(template.Node):
        def render(self, context):
            return ''

.. note::
    The actual implementation of ``{% comment %}<comment>`` is slightly
    different in that it allows broken template tags to appear between
    ``{% comment %}`` and ``{% endcomment %}``. It does so by calling
    ``parser.skip_past('endcomment')`` instead of ``parser.parse(('endcomment',))``
    followed by ``parser.delete_first_token()``, thus avoiding the generation of a
    node list.

``parser.parse()`` takes a tuple of names of block tags ''to parse until''. It
returns an instance of ``django.template.NodeList``, which is a list of
all ``Node`` objects that the parser encountered ''before'' it encountered
any of the tags named in the tuple.

In ``"nodelist = parser.parse(('endcomment',))"`` in the above example,
``nodelist`` is a list of all nodes between the ``{% comment %}`` and
``{% endcomment %}``, not counting ``{% comment %}`` and ``{% endcomment %}``
themselves.

After ``parser.parse()`` is called, the parser hasn't yet "consumed" the
``{% endcomment %}`` tag, so the code needs to explicitly call
``parser.delete_first_token()``.

``CommentNode.render()`` simply returns an empty string. Anything between
``{% comment %}`` and ``{% endcomment %}`` is ignored.

Parsing until another block tag, and saving contents
----------------------------------------------------

In the previous example, ``do_comment()`` discarded everything between
``{% comment %}`` and ``{% endcomment %}``. Instead of doing that, it's
possible to do something with the code between block tags.

For example, here's a custom template tag, ``{% upper %}``, that capitalizes
everything between itself and ``{% endupper %}``.

Usage:

.. code-block:: html+django

    {% upper %}This will appear in uppercase, {{ your_name }}.{% endupper %}

As in the previous example, we'll use ``parser.parse()``. But this time, we
pass the resulting ``nodelist`` to the ``Node``::

    def do_upper(parser, token):
        nodelist = parser.parse(('endupper',))
        parser.delete_first_token()
        return UpperNode(nodelist)

    class UpperNode(template.Node):
        def __init__(self, nodelist):
            self.nodelist = nodelist
        def render(self, context):
            output = self.nodelist.render(context)
            return output.upper()

The only new concept here is the ``self.nodelist.render(context)`` in
``UpperNode.render()``.

For more examples of complex rendering, see the source code of
``{% for %}<for>`` in ``django/template/defaulttags.py`` and
``{% if %}<if>`` in ``django/template/smartif.py``.

Writing Custom Template Loaders
===============================

Django's built-in template loaders (described in the "Inside Template Loading"
section above) will usually cover all your template-loading needs, but it's
pretty easy to write your own if you need special loading logic. For example,
you could load templates from a database, or directly from a Subversion
repository using Subversion's Python bindings, or (as shown shortly) from a ZIP
archive.

A template loader -- that is, each entry in the ``TEMPLATE_LOADERS`` setting
-- is expected to be a callable object with this interface::

    load_template_source(template_name, template_dirs=None)

The ``template_name`` argument is the name of the template to load (as passed
to ``loader.get_template()`` or ``loader.select_template()``), and
``template_dirs`` is an optional list of directories to search instead of
``TEMPLATE_DIRS``.

If a loader is able to successfully load a template, it should return a tuple:
``(template_source, template_path)``. Here, ``template_source`` is the
template string that will be compiled by the template engine, and
``template_path`` is the path the template was loaded from. That path might be
shown to the user for debugging purposes, so it should quickly identify where
the template was loaded from.

If the loader is unable to load a template, it should raise
``django.template.TemplateDoesNotExist``.

Each loader function should also have an ``is_usable`` function attribute.
This is a Boolean that informs the template engine whether this loader
is available in the current Python installation. For example, the eggs loader
(which is capable of loading templates from Python eggs) sets ``is_usable``
to ``False`` if the ``pkg_resources`` module isn't installed, because
``pkg_resources`` is necessary to read data from eggs.

An example should help clarify all of this. Here's a template loader function
that can load templates from a ZIP file. It uses a custom setting,
``TEMPLATE_ZIP_FILES``, as a search path instead of ``TEMPLATE_DIRS``, and it
expects each item on that path to be a ZIP file containing templates::

    from django.conf import settings
    from django.template import TemplateDoesNotExist
    import zipfile

    def load_template_source(template_name, template_dirs=None):
        "Template loader that loads templates from a ZIP file."

        template_zipfiles = getattr(settings, "TEMPLATE_ZIP_FILES", [])

        # Try each ZIP file in TEMPLATE_ZIP_FILES.
        for fname in template_zipfiles:
            try:
                z = zipfile.ZipFile(fname)
                source = z.read(template_name)
            except (IOError, KeyError):
                continue
            z.close()
            # We found a template, so return the source.
            template_path = "%s:%s" % (fname, template_name)
            return (source, template_path)

        # If we reach here, the template couldn't be loaded
        raise TemplateDoesNotExist(template_name)

    # This loader is always usable (since zipfile is included with Python)
    load_template_source.is_usable = True

The only step left if we want to use this loader is to add it to the
``TEMPLATE_LOADERS`` setting. If we put this code in a package called
``mysite.zip_loader``, then we add
``mysite.zip_loader.load_template_source`` to ``TEMPLATE_LOADERS``.

What's Next
===========

Continuing this section's theme of advanced topics, the next chapter covers
advanced usage of Django models.
