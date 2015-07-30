====================
Appendix D: Settings
====================

Your Django settings file contains all the configuration of your Django
installation. This appendix explains how settings work and which settings are
available.

What's a Settings File?
=======================

A settings file is just a Python module with module-level variables.

Here are a couple of example settings::

    ALLOWED_HOSTS = ['www.example.com']
    DEBUG = False
    DEFAULT_FROM_EMAIL = 'webmaster@example.com'

.. note::

    If you set ``DEBUG`` to ``False``, you also need to properly set
    the ``ALLOWED_HOSTS`` setting.

Because a settings file is a Python module, the following apply:

* It doesn't allow for Python syntax errors.
* It can assign settings dynamically using normal Python syntax.
  For example::

      MY_SETTING = [str(i) for i in range(30)]

* It can import values from other settings files.

Default Settings
----------------
A Django settings file doesn't have to define any settings if it doesn't need
to. Each setting has a sensible default value. These defaults live in the
module :file:`django/conf/global_settings.py`.

Here's the algorithm Django uses in compiling settings:

* Load settings from ``global_settings.py``.
* Load settings from the specified settings file, overriding the global
  settings as necessary.

Note that a settings file should *not* import from ``global_settings``, because
that's redundant.

Seeing which settings you've changed
------------------------------------

There's an easy way to view which of your settings deviate from the default
settings. The command ``python manage.py diffsettings`` displays differences
between the current settings file and Django's default settings.

For more, see the ``diffsettings`` documentation.

Using settings in Python code
=============================

In your Django apps, use settings by importing the object
``django.conf.settings``. Example::

    from django.conf import settings

    if settings.DEBUG:
        # Do something

Note that ``django.conf.settings`` isn't a module -- it's an object. So
importing individual settings is not possible::

    from django.conf.settings import DEBUG  # This won't work.

Also note that your code should *not* import from either ``global_settings`` or
your own settings file. ``django.conf.settings`` abstracts the concepts of
default settings and site-specific settings; it presents a single interface.
It also decouples the code that uses settings from the location of your
settings.

Altering settings at runtime
============================

You shouldn't alter settings in your applications at runtime. For example,
don't do this in a view::

    from django.conf import settings

    settings.DEBUG = True   # Don't do this!

The only place you should assign to settings is in a settings file.

Security
========

Because a settings file contains sensitive information, such as the database
password, you should make every attempt to limit access to it. For example,
change its file permissions so that only you and your Web server's user can
read it. This is especially important in a shared-hosting environment.

Creating your own settings
==========================

There's nothing stopping you from creating your own settings, for your own
Django apps. Just follow these conventions:

* Setting names are in all uppercase.
* Don't reinvent an already-existing setting.

For settings that are sequences, Django itself uses tuples, rather than lists,
but this is only a convention.


Designating the Settings: DJANGO_SETTINGS_MODULE
================================================
When you use Django, you have to tell it which settings you're using. Do this
by using an environment variable, ``DJANGO_SETTINGS_MODULE``.

The value of ``DJANGO_SETTINGS_MODULE`` should be in Python path syntax, e.g.
``mysite.settings``. 

The django-admin utility
---------------------------

When using django-admin , you can either set the
environment variable once, or explicitly pass in the settings module each time
you run the utility.

Example (Unix Bash shell)::

    export DJANGO_SETTINGS_MODULE=mysite.settings
    django-admin runserver

Example (Windows shell)::

    set DJANGO_SETTINGS_MODULE=mysite.settings
    django-admin runserver

Use the ``--settings`` command-line argument to specify the settings manually::

    django-admin runserver --settings=mysite.settings

.. _django-admin: ../django-admin/

On the server (mod_wsgi)
--------------------------

In your live server environment, you'll need to tell your WSGI
application what settings file to use. Do that with ``os.environ``::

    import os

    os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'

Read Chapter 22 for more information and other common
elements to a Django WSGI application.

.. _settings-without-django-settings-module:

Using Settings Without Setting DJANGO_SETTINGS_MODULE
=====================================================

In some cases, you might want to bypass the ``DJANGO_SETTINGS_MODULE``
environment variable. For example, if you're using the template system by
itself, you likely don't want to have to set up an environment variable
pointing to a settings module.

In these cases, you can configure Django's settings manually. Do this by
calling:

.. function:: django.conf.settings.configure(default_settings, \*\*settings)

Example::

    from django.conf import settings

    settings.configure(DEBUG=True, TEMPLATE_DEBUG=True)

Pass ``configure()`` as many keyword arguments as you'd like, with each keyword
argument representing a setting and its value. Each argument name should be all
uppercase, with the same name as the settings described above. If a particular
setting is not passed to ``configure()`` and is needed at some later point,
Django will use the default setting value.

Configuring Django in this fashion is mostly necessary -- and, indeed,
recommended -- when you're using a piece of the framework inside a larger
application.

Consequently, when configured via ``settings.configure()``, Django will not
make any modifications to the process environment variables (see the
documentation of ``TIME_ZONE`` for why this would normally occur). It's
assumed that you're already in full control of your environment in these
cases.

Custom default settings
-----------------------

If you'd like default values to come from somewhere other than
``django.conf.global_settings``, you can pass in a module or class that
provides the default settings as the ``default_settings`` argument (or as the
first positional argument) in the call to ``configure()``.

In this example, default settings are taken from ``myapp_defaults``, and the
``DEBUG`` setting is set to ``True``, regardless of its value in
``myapp_defaults``::

    from django.conf import settings
    from myapp import myapp_defaults

    settings.configure(default_settings=myapp_defaults, DEBUG=True)

The following example, which uses ``myapp_defaults`` as a positional argument,
is equivalent::

    settings.configure(myapp_defaults, DEBUG=True)

Normally, you will not need to override the defaults in this fashion. The
Django defaults are sufficiently tame that you can safely use them. Be aware
that if you do pass in a new default module, it entirely *replaces* the Django
defaults, so you must specify a value for every possible setting that might be
used in that code you are importing. Check in
``django.conf.settings.global_settings`` for the full list.

Either configure() or DJANGO_SETTINGS_MODULE is required
--------------------------------------------------------

If you're not setting the ``DJANGO_SETTINGS_MODULE`` environment variable, you
*must* call ``configure()`` at some point before using any code that reads
settings.

If you don't set ``DJANGO_SETTINGS_MODULE`` and don't call ``configure()``,
Django will raise an ``ImportError`` exception the first time a setting
is accessed.

If you set ``DJANGO_SETTINGS_MODULE``, access settings values somehow, *then*
call ``configure()``, Django will raise a ``RuntimeError`` indicating
that settings have already been configured. There is a property just for this
purpose:

.. attribute:: django.conf.settings.configured

For example::

    from django.conf import settings
    if not settings.configured:
        settings.configure(myapp_defaults, DEBUG=True)

Also, it's an error to call ``configure()`` more than once, or to call
``configure()`` after any setting has been accessed.

It boils down to this: Use exactly one of either ``configure()`` or
``DJANGO_SETTINGS_MODULE``. Not both, and not neither.

.. _@login_required: ../authentication/#the-login-required-decorator

Available Settings
==================

.. warning::

    Be careful when you override settings, especially when the default value
    is a non-empty list or dictionary, such as ``MIDDLEWARE_CLASSES``
    and ``STATICFILES_FINDERS``. Make sure you keep the components
    required by the features of Django you wish to use.

Core settings
=============

Here's a list of settings available in Django core and their default values.
Settings provided by contrib apps are listed below, followed by a topical index
of the core settings. 

ABSOLUTE_URL_OVERRIDES
----------------------

Default: ``{}`` (Empty dictionary)

A dictionary mapping ``"app_label.model_name"`` strings to functions that take
a model object and return its URL. This is a way of inserting or overriding
``get_absolute_url()`` methods on a per-installation basis. Example::

    ABSOLUTE_URL_OVERRIDES = {
        'blogs.weblog': lambda o: "/blogs/%s/" % o.slug,
        'news.story': lambda o: "/stories/%s/%s/" % (o.pub_year, o.slug),
    }

Note that the model name used in this setting should be all lower-case, regardless
of the case of the actual model class name.

ADMINS
------

Default: ``[]`` (Empty list)

A list of all the people who get code error notifications. When
``DEBUG=False`` and a view raises an exception, Django will email these people
with the full exception information. Each item in the list should be a tuple
of (Full name, email address). Example::

    [('John', 'john@example.com'), ('Mary', 'mary@example.com')]

Note that Django will email *all* of these people whenever an error happens.

ALLOWED_HOSTS
-------------

Default: ``[]`` (Empty list)

A list of strings representing the host/domain names that this Django site can
serve. This is a security measure to prevent an attacker from poisoning caches
and password reset emails with links to malicious hosts by submitting requests
with a fake HTTP ``Host`` header, which is possible even under many
seemingly-safe web server configurations.

Values in this list can be fully qualified names (e.g. ``'www.example.com'``),
in which case they will be matched against the request's ``Host`` header
exactly (case-insensitive, not including port). A value beginning with a period
can be used as a subdomain wildcard: ``'.example.com'`` will match
``example.com``, ``www.example.com``, and any other subdomain of
``example.com``. A value of ``'*'`` will match anything; in this case you are
responsible to provide your own validation of the ``Host`` header (perhaps in a
middleware; if so this middleware must be listed first in
``MIDDLEWARE_CLASSES``).

Django also allows the `fully qualified domain name (FQDN)`_ of any entries.
Some browsers include a trailing dot in the ``Host`` header which Django
strips when performing host validation.

.. _`fully qualified domain name (FQDN)`: http://en.wikipedia.org/wiki/Fully_qualified_domain_name

If the ``Host`` header (or ``X-Forwarded-Host`` if
``USE_X_FORWARDED_HOST`` is enabled) does not match any value in this
list, the :meth:`django.http.HttpRequest.get_host()` method will raise
:exc:`~django.core.exceptions.SuspiciousOperation`.

When ``DEBUG`` is ``True`` or when running tests, host validation is
disabled; any host will be accepted. Thus it's usually only necessary to set it
in production.

This validation only applies via :meth:`~django.http.HttpRequest.get_host()`;
if your code accesses the ``Host`` header directly from ``request.META`` you
are bypassing this security protection.

APPEND_SLASH
------------

Default: ``True``

When set to ``True``, if the request URL does not match any of the patterns
in the URLconf and it doesn't end in a slash, an HTTP redirect is issued to the
same URL with a slash appended. Note that the redirect may cause any data
submitted in a POST request to be lost.

The ``APPEND_SLASH`` setting is only used if
:class:`~django.middleware.common.CommonMiddleware` is installed
. See also ``PREPEND_WWW``.

CACHES
------

Default::

    {
        'default': {
            'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        }
    }

A dictionary containing the settings for all caches to be used with
Django. It is a nested dictionary whose contents maps cache aliases
to a dictionary containing the options for an individual cache.

The ``CACHES`` setting must configure a ``default`` cache;
any number of additional caches may also be specified. If you
are using a cache backend other than the local memory cache, or
you need to define multiple caches, other options will be required.
The following cache options are available.

BACKEND
~~~~~~~

Default: ``''`` (Empty string)

The cache backend to use. The built-in cache backends are:

* ``'django.core.cache.backends.db.DatabaseCache'``
* ``'django.core.cache.backends.dummy.DummyCache'``
* ``'django.core.cache.backends.filebased.FileBasedCache'``
* ``'django.core.cache.backends.locmem.LocMemCache'``
* ``'django.core.cache.backends.memcached.MemcachedCache'``
* ``'django.core.cache.backends.memcached.PyLibMCCache'``

You can use a cache backend that doesn't ship with Django by setting
``BACKEND <CACHES-BACKEND>`` to a fully-qualified path of a cache
backend class (i.e. ``mypackage.backends.whatever.WhateverCache``).

KEY_FUNCTION
~~~~~~~~~~~~

A string containing a dotted path to a function (or any callable) that defines how to
compose a prefix, version and key into a final cache key. The default
implementation is equivalent to the function::

    def make_key(key, key_prefix, version):
        return ':'.join([key_prefix, str(version), key])

You may use any key function you want, as long as it has the same
argument signature.

KEY_PREFIX
~~~~~~~~~~

Default: ``''`` (Empty string)

A string that will be automatically included (prepended by default) to
all cache keys used by the Django server.

LOCATION
~~~~~~~~

Default: ``''`` (Empty string)

The location of the cache to use. This might be the directory for a
file system cache, a host and port for a memcache server, or simply an
identifying name for a local memory cache. e.g.::

    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
            'LOCATION': '/var/tmp/django_cache',
        }
    }

OPTIONS
~~~~~~~

Default: None

Extra parameters to pass to the cache backend. Available parameters
vary depending on your cache backend.

Some information on available parameters can be found in the
Cache Backends  documentation. For more information,
consult your backend module's own documentation.

TIMEOUT
~~~~~~~

Default: 300

The number of seconds before a cache entry is considered stale. If the value of
this settings is ``None``, cache entries will not expire.

VERSION
~~~~~~~

Default: ``1``

The default version number for cache keys generated by the Django server.

CACHE_MIDDLEWARE_ALIAS
----------------------

Default: ``default``

The cache connection to use for the cache middleware.

CACHE_MIDDLEWARE_KEY_PREFIX
---------------------------

Default: ``''`` (Empty string)

A string which will be prefixed to the cache keys generated by the cache
middleware. This prefix is combined with the
``KEY_PREFIX <CACHES-KEY_PREFIX>`` setting; it does not replace it.

See Chapter 17 for more information on caching in Django.

CACHE_MIDDLEWARE_SECONDS
------------------------

Default: ``600``

The default number of seconds to cache a page for the cache middleware.

See Chapter 17 for more information on caching in Django.

.. _settings-csrf:

CSRF_COOKIE_AGE
---------------

Default: ``31449600`` (1 year, in seconds)

The age of CSRF cookies, in seconds.

The reason for setting a long-lived expiration time is to avoid problems in
the case of a user closing a browser or bookmarking a page and then loading
that page from a browser cache. Without persistent cookies, the form submission
would fail in this case.

Some browsers (specifically Internet Explorer) can disallow the use of
persistent cookies or can have the indexes to the cookie jar corrupted on disk,
thereby causing CSRF protection checks to fail (and sometimes intermittently).
Change this setting to ``None`` to use session-based CSRF cookies, which
keep the cookies in-memory instead of on persistent storage.

CSRF_COOKIE_DOMAIN
------------------

Default: ``None``

The domain to be used when setting the CSRF cookie.  This can be useful for
easily allowing cross-subdomain requests to be excluded from the normal cross
site request forgery protection.  It should be set to a string such as
``".example.com"`` to allow a POST request from a form on one subdomain to be
accepted by a view served from another subdomain.

Please note that the presence of this setting does not imply that Django's CSRF
protection is safe from cross-subdomain attacks by default - please see the
CSRF limitations section.

CSRF_COOKIE_HTTPONLY
--------------------

Default: ``False``

Whether to use ``HttpOnly`` flag on the CSRF cookie. If this is set to
``True``, client-side JavaScript will not to be able to access the CSRF cookie.

This can help prevent malicious JavaScript from bypassing CSRF protection. If
you enable this and need to send the value of the CSRF token with Ajax requests,
your JavaScript will need to pull the value from a hidden CSRF token form input
on the page instead of from the cookie.

See ``SESSION_COOKIE_HTTPONLY`` for details on ``HttpOnly``.

CSRF_COOKIE_NAME
----------------

Default: ``'csrftoken'``

The name of the cookie to use for the CSRF authentication token. This can be whatever you
want. 

CSRF_COOKIE_PATH
----------------

Default: ``'/'``

The path set on the CSRF cookie. This should either match the URL path of your
Django installation or be a parent of that path.

This is useful if you have multiple Django instances running under the same
hostname. They can use different cookie paths, and each instance will only see
its own CSRF cookie.

CSRF_COOKIE_SECURE
------------------

Default: ``False``

Whether to use a secure cookie for the CSRF cookie. If this is set to ``True``,
the cookie will be marked as "secure," which means browsers may ensure that the
cookie is only sent under an HTTPS connection.

CSRF_FAILURE_VIEW
-----------------

Default: ``'django.views.csrf.csrf_failure'``

A dotted path to the view function to be used when an incoming request
is rejected by the CSRF protection.  The function should have this signature::

  def csrf_failure(request, reason="")

where ``reason`` is a short message (intended for developers or logging, not for
end users) indicating the reason the request was rejected.  

DATABASES
---------

Default: ``{}`` (Empty dictionary)

A dictionary containing the settings for all databases to be used with
Django. It is a nested dictionary whose contents maps database aliases
to a dictionary containing the options for an individual database.

The ``DATABASES`` setting must configure a ``default`` database;
any number of additional databases may also be specified.

The simplest possible settings file is for a single-database setup using
SQLite. This can be configured using the following::

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': 'mydatabase',
        }
    }

When connecting to other database backends, such as MySQL, Oracle, or
PostgreSQL, additional connection parameters will be required. See
the ``ENGINE <DATABASE-ENGINE>`` setting below on how to specify
other database types. This example is for PostgreSQL::

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': 'mydatabase',
            'USER': 'mydatabaseuser',
            'PASSWORD': 'mypassword',
            'HOST': '127.0.0.1',
            'PORT': '5432',
        }
    }

The following inner options that may be required for more complex
configurations are available:

ATOMIC_REQUESTS
~~~~~~~~~~~~~~~

Default: ``False``

Set this to ``True`` to wrap each HTTP request in a transaction on this
database.

AUTOCOMMIT
~~~~~~~~~~

Default: ``True``

Set this to ``False`` if you want to disable Django's transaction
management and implement your own.

ENGINE
~~~~~~

Default: ``''`` (Empty string)

The database backend to use. The built-in database backends are:

* ``'django.db.backends.postgresql_psycopg2'``
* ``'django.db.backends.mysql'``
* ``'django.db.backends.sqlite3'``
* ``'django.db.backends.oracle'``

You can use a database backend that doesn't ship with Django by setting
``ENGINE`` to a fully-qualified path (i.e.
``mypackage.backends.whatever``).

HOST
~~~~

Default: ``''`` (Empty string)

Which host to use when connecting to the database. An empty string means
localhost. Not used with SQLite.

If this value starts with a forward slash (``'/'``) and you're using MySQL,
MySQL will connect via a Unix socket to the specified socket. For example::

    "HOST": '/var/run/mysql'

If you're using MySQL and this value *doesn't* start with a forward slash, then
this value is assumed to be the host.

If you're using PostgreSQL, by default (empty ``HOST``), the connection
to the database is done through UNIX domain sockets ('local' lines in
``pg_hba.conf``). If your UNIX domain socket is not in the standard location,
use the same value of ``unix_socket_directory`` from ``postgresql.conf``.
If you want to connect through TCP sockets, set ``HOST`` to 'localhost'
or '127.0.0.1' ('host' lines in ``pg_hba.conf``).
On Windows, you should always define ``HOST``, as UNIX domain sockets
are not available.

NAME
~~~~

Default: ``''`` (Empty string)

The name of the database to use. For SQLite, it's the full path to the database
file. When specifying the path, always use forward slashes, even on Windows
(e.g. ``C:/homes/user/mysite/sqlite3.db``).

CONN_MAX_AGE
~~~~~~~~~~~~

Default: ``0``

The lifetime of a database connection, in seconds. Use ``0`` to close database
connections at the end of each request — Django's historical behavior — and
``None`` for unlimited persistent connections.

OPTIONS
~~~~~~~

Default: ``{}`` (Empty dictionary)

Extra parameters to use when connecting to the database. Available parameters
vary depending on your database backend.

Some information on available parameters can be found in the
Database Backends  documentation. For more information,
consult your backend module's own documentation.

PASSWORD
~~~~~~~~

Default: ``''`` (Empty string)

The password to use when connecting to the database. Not used with SQLite.

PORT
~~~~

Default: ``''`` (Empty string)

The port to use when connecting to the database. An empty string means the
default port. Not used with SQLite.

USER
~~~~

Default: ``''`` (Empty string)

The username to use when connecting to the database. Not used with SQLite.

TEST
~~~~

Default: ``{}``

A dictionary of settings for test databases. The
following entries are available:

CHARSET
^^^^^^^

Default: ``None``

The character set encoding used to create the test database. The value of this
string is passed directly through to the database, so its format is
backend-specific.

Supported for the PostgreSQL_ (``postgresql_psycopg2``) and MySQL_ (``mysql``)
backends.

.. _PostgreSQL: http://www.postgresql.org/docs/current/static/multibyte.html
.. _MySQL: http://dev.mysql.com/doc/refman/5.6/en/charset-database.html

COLLATION
^^^^^^^^^

Default: ``None``

The collation order to use when creating the test database. This value is
passed directly to the backend, so its format is backend-specific.

Only supported for the ``mysql`` backend (see the `MySQL manual`_ for details).

.. _MySQL manual: MySQL_

DEPENDENCIES
^^^^^^^^^^^^

Default: ``['default']``, for all databases other than ``default``,
which has no dependencies.

The creation-order dependencies of the database. 
MIRROR
^^^^^^

Default: ``None``

The alias of the database that this database should mirror during
testing.

This setting exists to allow for testing of primary/replica
(referred to as master/slave by some databases)
configurations of multiple databases. 

NAME
^^^^

Default: ``None``

The name of database to use when running the test suite.

If the default value (``None``) is used with the SQLite database engine, the
tests will use a memory resident database. For all other database engines the
test database will use the name ``'test_' + DATABASE_NAME``.

SERIALIZE
^^^^^^^^^

Boolean value to control whether or not the default test runner serializes the
database into an in-memory JSON string before running tests (used to restore
the database state between tests if you don't have transactions). You can set
this to ``False`` to speed up creation time if you don't have any test classes
with serialized_rollback=True.

CREATE_DB
^^^^^^^^^

Default: ``True``

This is an Oracle-specific setting.

If it is set to ``False``, the test tablespaces won't be automatically created
at the beginning of the tests and dropped at the end.

CREATE_USER
^^^^^^^^^^^

Default: ``True``

This is an Oracle-specific setting.

If it is set to ``False``, the test user won't be automatically created at the
beginning of the tests and dropped at the end.

USER
^^^^

Default: ``None``

This is an Oracle-specific setting.

The username to use when connecting to the Oracle database that will be used
when running tests. If not provided, Django will use ``'test_' + USER``.

PASSWORD
^^^^^^^^

Default: ``None``

This is an Oracle-specific setting.

The password to use when connecting to the Oracle database that will be used
when running tests. If not provided, Django will use a hardcoded default value.

TBLSPACE
^^^^^^^^

Default: ``None``

This is an Oracle-specific setting.

The name of the tablespace that will be used when running tests. If not
provided, Django will use ``'test_' + USER``.

.. versionchanged:: 1.8

    Previously Django used ``'test_' + NAME`` if not provided.

TBLSPACE_TMP
^^^^^^^^^^^^

Default: ``None``

This is an Oracle-specific setting.

The name of the temporary tablespace that will be used when running tests. If
not provided, Django will use ``'test_' + USER + '_temp'``.

.. versionchanged:: 1.8

    Previously Django used ``'test_' + NAME + '_temp'`` if not provided.

DATAFILE
^^^^^^^^

Default: ``None``

This is an Oracle-specific setting.

The name of the datafile to use for the TBLSPACE. If not provided, Django will
use ``TBLSPACE + '.dbf'``.

DATAFILE_TMP
^^^^^^^^^^^^

Default: ``None``

This is an Oracle-specific setting.

The name of the datafile to use for the TBLSPACE_TMP. If not provided, Django
will use ``TBLSPACE_TMP + '.dbf'``.

DATAFILE_MAXSIZE
^^^^^^^^^^^^^^^^

Default: ``'500M'``

This is an Oracle-specific setting.

The maximum size that the DATAFILE is allowed to grow to.

DATAFILE_TMP_MAXSIZE
^^^^^^^^^^^^^^^^^^^^

Default: ``'500M'``

This is an Oracle-specific setting.

The maximum size that the DATAFILE_TMP is allowed to grow to.

DATABASE_ROUTERS
----------------

Default: ``[]`` (Empty list)

The list of routers that will be used to determine which database
to use when performing a database queries.

DATE_FORMAT
-----------

Default: ``'N j, Y'`` (e.g. ``Feb. 4, 2003``)

The default formatting to use for displaying date fields in any part of the
system. Note that if ``USE_L10N`` is set to ``True``, then the
locale-dictated format has higher precedence and will be applied instead. 

See also ``DATETIME_FORMAT``, ``TIME_FORMAT`` and ``SHORT_DATE_FORMAT``.

DATE_INPUT_FORMATS
------------------

Default::

    [
        '%Y-%m-%d', '%m/%d/%Y', '%m/%d/%y', # '2006-10-25', '10/25/2006', '10/25/06'
        '%b %d %Y', '%b %d, %Y',            # 'Oct 25 2006', 'Oct 25, 2006'
        '%d %b %Y', '%d %b, %Y',            # '25 Oct 2006', '25 Oct, 2006'
        '%B %d %Y', '%B %d, %Y',            # 'October 25 2006', 'October 25, 2006'
        '%d %B %Y', '%d %B, %Y',            # '25 October 2006', '25 October, 2006'
    ]

A list of formats that will be accepted when inputting data on a date field.
Formats will be tried in order, using the first valid one. Note that these
format strings use Python's datetime_ module syntax, not the format strings
from the ``date`` Django template tag.

When ``USE_L10N`` is ``True``, the locale-dictated format has higher
precedence and will be applied instead.

See also ``DATETIME_INPUT_FORMATS`` and ``TIME_INPUT_FORMATS``.

.. _datetime: https://docs.python.org/library/datetime.html#strftime-strptime-behavior

DATETIME_FORMAT
---------------

Default: ``'N j, Y, P'`` (e.g. ``Feb. 4, 2003, 4 p.m.``)

The default formatting to use for displaying datetime fields in any part of the
system. Note that if ``USE_L10N`` is set to ``True``, then the
locale-dictated format has higher precedence and will be applied instead. 

See also ``DATE_FORMAT``, ``TIME_FORMAT`` and ``SHORT_DATETIME_FORMAT``.

DATETIME_INPUT_FORMATS
----------------------

Default::

    [
        '%Y-%m-%d %H:%M:%S',     # '2006-10-25 14:30:59'
        '%Y-%m-%d %H:%M:%S.%f',  # '2006-10-25 14:30:59.000200'
        '%Y-%m-%d %H:%M',        # '2006-10-25 14:30'
        '%Y-%m-%d',              # '2006-10-25'
        '%m/%d/%Y %H:%M:%S',     # '10/25/2006 14:30:59'
        '%m/%d/%Y %H:%M:%S.%f',  # '10/25/2006 14:30:59.000200'
        '%m/%d/%Y %H:%M',        # '10/25/2006 14:30'
        '%m/%d/%Y',              # '10/25/2006'
        '%m/%d/%y %H:%M:%S',     # '10/25/06 14:30:59'
        '%m/%d/%y %H:%M:%S.%f',  # '10/25/06 14:30:59.000200'
        '%m/%d/%y %H:%M',        # '10/25/06 14:30'
        '%m/%d/%y',              # '10/25/06'
    ]

A list of formats that will be accepted when inputting data on a datetime
field. Formats will be tried in order, using the first valid one. Note that
these format strings use Python's datetime_ module syntax, not the format
strings from the ``date`` Django template tag.

When ``USE_L10N`` is ``True``, the locale-dictated format has higher
precedence and will be applied instead.

See also ``DATE_INPUT_FORMATS`` and ``TIME_INPUT_FORMATS``.

.. _datetime: https://docs.python.org/library/datetime.html#strftime-strptime-behavior

DEBUG
-----

Default: ``False``

A boolean that turns on/off debug mode.

Never deploy a site into production with ``DEBUG`` turned on.

Did you catch that? NEVER deploy a site into production with ``DEBUG``
turned on.

One of the main features of debug mode is the display of detailed error pages.
If your app raises an exception when ``DEBUG`` is ``True``, Django will
display a detailed traceback, including a lot of metadata about your
environment, such as all the currently defined Django settings (from
``settings.py``).

As a security measure, Django will *not* include settings that might be
sensitive (or offensive), such as ``SECRET_KEY``. Specifically, it will
exclude any setting whose name includes any of the following:

* ``'API'``
* ``'KEY'``
* ``'PASS'``
* ``'SECRET'``
* ``'SIGNATURE'``
* ``'TOKEN'``

Note that these are *partial* matches. ``'PASS'`` will also match PASSWORD,
just as ``'TOKEN'`` will also match TOKENIZED and so on.

Still, note that there are always going to be sections of your debug output
that are inappropriate for public consumption. File paths, configuration
options and the like all give attackers extra information about your server.

It is also important to remember that when running with ``DEBUG``
turned on, Django will remember every SQL query it executes. This is useful
when you're debugging, but it'll rapidly consume memory on a production server.

Finally, if ``DEBUG`` is ``False``, you also need to properly set
the ``ALLOWED_HOSTS`` setting. Failing to do so will result in all
requests being returned as "Bad Request (400)".

.. _django/views/debug.py: https://github.com/django/django/blob/master/django/views/debug.py

DEBUG_PROPAGATE_EXCEPTIONS
--------------------------

Default: ``False``

If set to True, Django's normal exception handling of view functions
will be suppressed, and exceptions will propagate upwards.  This can
be useful for some test setups, and should never be used on a live
site.

DECIMAL_SEPARATOR
-----------------

Default: ``'.'`` (Dot)

Default decimal separator used when formatting decimal numbers.

Note that if ``USE_L10N`` is set to ``True``, then the locale-dictated
format has higher precedence and will be applied instead.

See also ``NUMBER_GROUPING``, ``THOUSAND_SEPARATOR`` and
``USE_THOUSAND_SEPARATOR``.


DEFAULT_CHARSET
---------------

Default: ``'utf-8'``

Default charset to use for all ``HttpResponse`` objects, if a MIME type isn't
manually specified. Used with ``DEFAULT_CONTENT_TYPE`` to construct the
``Content-Type`` header.

DEFAULT_CONTENT_TYPE
--------------------

Default: ``'text/html'``

Default content type to use for all ``HttpResponse`` objects, if a MIME type
isn't manually specified. Used with ``DEFAULT_CHARSET`` to construct
the ``Content-Type`` header.

DEFAULT_EXCEPTION_REPORTER_FILTER
---------------------------------

Default: :class:`django.views.debug.SafeExceptionReporterFilter`

Default exception reporter filter class to be used if none has been assigned to
the :class:`~django.http.HttpRequest` instance yet.

DEFAULT_FILE_STORAGE
--------------------

Default: :class:`django.core.files.storage.FileSystemStorage`

Default file storage class to be used for any file-related operations that don't
specify a particular storage system.

DEFAULT_FROM_EMAIL
------------------

Default: ``'webmaster@localhost'``

Default email address to use for various automated correspondence from the
site manager(s). This doesn't include error messages sent to ``ADMINS``
and ``MANAGERS``; for that, see ``SERVER_EMAIL``.

DEFAULT_INDEX_TABLESPACE
------------------------

Default: ``''`` (Empty string)

Default tablespace to use for indexes on fields that don't specify
one, if the backend supports it.

DEFAULT_TABLESPACE
------------------

Default: ``''`` (Empty string)

Default tablespace to use for models that don't specify one, if the
backend supports it.

DISALLOWED_USER_AGENTS
----------------------

Default: ``[]`` (Empty list)

List of compiled regular expression objects representing User-Agent strings that
are not allowed to visit any page, systemwide. Use this for bad robots/crawlers.
This is only used if ``CommonMiddleware`` is installed.

EMAIL_BACKEND
-------------

Default: ``'django.core.mail.backends.smtp.EmailBackend'``

The backend to use for sending emails. For the list of available backends see
Appendix H.
EMAIL_FILE_PATH
---------------

Default: Not defined

The directory used by the ``file`` email backend to store output files.

EMAIL_HOST
----------

Default: ``'localhost'``

The host to use for sending email.

See also ``EMAIL_PORT``.

EMAIL_HOST_PASSWORD
-------------------

Default: ``''`` (Empty string)

Password to use for the SMTP server defined in ``EMAIL_HOST``. This
setting is used in conjunction with ``EMAIL_HOST_USER`` when
authenticating to the SMTP server. If either of these settings is empty,
Django won't attempt authentication.

See also ``EMAIL_HOST_USER``.

EMAIL_HOST_USER
---------------

Default: ``''`` (Empty string)

Username to use for the SMTP server defined in ``EMAIL_HOST``.
If empty, Django won't attempt authentication.

See also ``EMAIL_HOST_PASSWORD``.

EMAIL_PORT
----------

Default: ``25``

Port to use for the SMTP server defined in ``EMAIL_HOST``.

EMAIL_SUBJECT_PREFIX
--------------------

Default: ``'[Django] '``

Subject-line prefix for email messages sent with ``django.core.mail.mail_admins``
or ``django.core.mail.mail_managers``. You'll probably want to include the
trailing space.

EMAIL_USE_TLS
-------------

Default: ``False``

Whether to use a TLS (secure) connection when talking to the SMTP server.
This is used for explicit TLS connections, generally on port 587. If you are
experiencing hanging connections, see the implicit TLS setting
``EMAIL_USE_SSL``.

EMAIL_USE_SSL
-------------

Default: ``False``

Whether to use an implicit TLS (secure) connection when talking to the SMTP
server. In most email documentation this type of TLS connection is referred
to as SSL. It is generally used on port 465. If you are experiencing problems,
see the explicit TLS setting ``EMAIL_USE_TLS``.

Note that ``EMAIL_USE_TLS``/``EMAIL_USE_SSL`` are mutually
exclusive, so only set one of those settings to ``True``.

EMAIL_SSL_CERTFILE
------------------

Default: ``None``

If ``EMAIL_USE_SSL`` or ``EMAIL_USE_TLS`` is ``True``, you can
optionally specify the path to a PEM-formatted certificate chain file to use
for the SSL connection.

EMAIL_SSL_KEYFILE
-----------------

Default: ``None``

If ``EMAIL_USE_SSL`` or ``EMAIL_USE_TLS`` is ``True``, you can
optionally specify the path to a PEM-formatted private key file to use for the
SSL connection.

Note that setting ``EMAIL_SSL_CERTFILE`` and ``EMAIL_SSL_KEYFILE``
doesn't result in any certificate checking. They're passed to the underlying SSL
connection. Please refer to the documentation of Python's
:func:`python:ssl.wrap_socket` function for details on how the certificate chain
file and private key file are handled.

EMAIL_TIMEOUT
-------------

Default: ``None``

Specifies a timeout in seconds for blocking operations like the connection
attempt.

FILE_CHARSET
------------

Default: ``'utf-8'``

The character encoding used to decode any files read from disk. This includes
template files and initial SQL data files.

FILE_UPLOAD_HANDLERS
--------------------

Default::

    ["django.core.files.uploadhandler.MemoryFileUploadHandler",
     "django.core.files.uploadhandler.TemporaryFileUploadHandler"]

A list of handlers to use for uploading. Changing this setting allows complete
customization -- even replacement -- of Django's upload process.

FILE_UPLOAD_MAX_MEMORY_SIZE
---------------------------

Default: ``2621440`` (i.e. 2.5 MB).

The maximum size (in bytes) that an upload will be before it gets streamed to
the file system.

FILE_UPLOAD_DIRECTORY_PERMISSIONS
---------------------------------

Default: ``None``

The numeric mode to apply to directories created in the process of uploading
files.

This setting also determines the default permissions for collected static
directories when using the ``collectstatic`` management command. See
``collectstatic`` for details on overriding it.

This value mirrors the functionality and caveats of the
``FILE_UPLOAD_PERMISSIONS`` setting.

FILE_UPLOAD_PERMISSIONS
-----------------------

Default: ``None``

The numeric mode (i.e. ``0o644``) to set newly uploaded files to. For
more information about what these modes mean, see the documentation for
:func:`os.chmod`.

If this isn't given or is ``None``, you'll get operating-system
dependent behavior. On most platforms, temporary files will have a mode
of ``0o600``, and files saved from memory will be saved using the
system's standard umask.

For security reasons, these permissions aren't applied to the temporary files
that are stored in ``FILE_UPLOAD_TEMP_DIR``.

This setting also determines the default permissions for collected static files
when using the ``collectstatic`` management command. See
``collectstatic`` for details on overriding it.

.. warning::

    **Always prefix the mode with a 0.**

    If you're not familiar with file modes, please note that the leading
    ``0`` is very important: it indicates an octal number, which is the
    way that modes must be specified. If you try to use ``644``, you'll
    get totally incorrect behavior.

FILE_UPLOAD_TEMP_DIR
--------------------

Default: ``None``

The directory to store data (typically files larger than
``FILE_UPLOAD_MAX_MEMORY_SIZE``) temporarily while uploading files.
If ``None``, Django will use the standard temporary directory for the operating
system. For example, this will default to ``/tmp`` on \*nix-style operating
systems.

FIRST_DAY_OF_WEEK
-----------------

Default: ``0`` (Sunday)

Number representing the first day of the week. This is especially useful
when displaying a calendar. This value is only used when not using
format internationalization, or when a format cannot be found for the
current locale.

The value must be an integer from 0 to 6, where 0 means Sunday, 1 means
Monday and so on.

FIXTURE_DIRS
-------------

Default: ``[]`` (Empty list)

List of directories searched for fixture files, in addition to the
``fixtures`` directory of each application, in search order.

Note that these paths should use Unix-style forward slashes, even on Windows.

FORCE_SCRIPT_NAME
------------------

Default: ``None``

If not ``None``, this will be used as the value of the ``SCRIPT_NAME``
environment variable in any HTTP request. This setting can be used to override
the server-provided value of ``SCRIPT_NAME``, which may be a rewritten version
of the preferred value or not supplied at all.

FORMAT_MODULE_PATH
------------------

Default: ``None``

A full Python path to a Python package that contains format definitions for
project locales. If not ``None``, Django will check for a ``formats.py``
file, under the directory named as the current locale, and will use the
formats defined on this file.

For example, if ``FORMAT_MODULE_PATH`` is set to ``mysite.formats``,
and current language is ``en`` (English), Django will expect a directory tree
like::

    mysite/
        formats/
            __init__.py
            en/
                __init__.py
                formats.py


You can also set this setting to a list of Python paths, for example::

	FORMAT_MODULE_PATH = [
		'mysite.formats',
		'some_app.formats',
	]

When Django searches for a certain format, it will go through all given
Python paths until it finds a module that actually defines the given
format. This means that formats defined in packages farther up in the list
will take precedence over the same formats in packages farther down.

Available formats are ``DATE_FORMAT``, ``TIME_FORMAT``,
``DATETIME_FORMAT``, ``YEAR_MONTH_FORMAT``,
``MONTH_DAY_FORMAT``, ``SHORT_DATE_FORMAT``,
``SHORT_DATETIME_FORMAT``, ``FIRST_DAY_OF_WEEK``,
``DECIMAL_SEPARATOR``, ``THOUSAND_SEPARATOR`` and
``NUMBER_GROUPING``.

IGNORABLE_404_URLS
------------------

Default: ``[]`` (Empty list)

List of compiled regular expression objects describing URLs that should be
ignored when reporting HTTP 404 errors via email. Regular expressions are matched against
:meth:`request's full paths <django.http.HttpRequest.get_full_path>` (including
query string, if any). Use this if your site does not provide a commonly
requested file such as ``favicon.ico`` or ``robots.txt``, or if it gets
hammered by script kiddies.

This is only used if
:class:`~django.middleware.common.BrokenLinkEmailsMiddleware` is enabled.

INSTALLED_APPS
--------------

Default: ``[]`` (Empty list)

A list of strings designating all applications that are enabled in this
Django installation. Each string should be a dotted Python path to:

* an application configuration class, or
* a package containing a application.

Learn more about application configurations .

.. admonition:: Use the application registry for introspection

    Your code should never access ``INSTALLED_APPS`` directly. Use
    :attr:`django.apps.apps` instead.

.. admonition:: Application names and labels must be unique in
                ``INSTALLED_APPS``

    Application :attr:`names <django.apps.AppConfig.name>` — the dotted Python
    path to the application package — must be unique. There is no way to
    include the same application twice, short of duplicating its code under
    another name.

    Application :attr:`labels <django.apps.AppConfig.label>` — by default the
    final part of the name — must be unique too. For example, you can't
    include both ``django.contrib.auth`` and ``myproject.auth``. However, you
    can relabel an application with a custom configuration that defines a
    different :attr:`~django.apps.AppConfig.label`.

    These rules apply regardless of whether ``INSTALLED_APPS``
    references application configuration classes on application packages.

When several applications provide different versions of the same resource
(template, static file, management command, translation), the application
listed first in ``INSTALLED_APPS`` has precedence.

INTERNAL_IPS
------------

Default: ``[]`` (Empty list)

A list of IP addresses, as strings, that:

* See debug comments, when ``DEBUG`` is ``True``
* Receive X headers in admindocs if the ``XViewMiddleware`` is installed.

LANGUAGE_CODE
-------------

Default: ``'en-us'``

A string representing the language code for this installation. This should be in
standard :term:`language ID format <language code>`. For example, U.S. English
is ``"en-us"``. 

``USE_I18N`` must be active for this setting to have any effect.

It serves two purposes:

* If the locale middleware isn't in use, it decides which translation is served
  to all users.
* If the locale middleware is active, it provides the fallback translation when
  no translation exist for a given literal to the user's preferred language.

.. _list of language identifiers: http://www.i18nguy.com/unicode/language-identifiers.html

LANGUAGE_COOKIE_AGE
-------------------

Default: ``None`` (expires at browser close)

The age of the language cookie, in seconds.

LANGUAGE_COOKIE_DOMAIN
----------------------

Default: ``None``

The domain to use for the language cookie. Set this to a string such as
``".example.com"`` (note the leading dot!) for cross-domain cookies, or use
``None`` for a standard domain cookie.

Be cautious when updating this setting on a production site. If you update
this setting to enable cross-domain cookies on a site that previously used
standard domain cookies, existing user cookies that have the old domain
will not be updated. This will result in site users being unable to switch
the language as long as these cookies persist. The only safe and reliable
option to perform the switch is to change the language cookie name
permanently (via the ``LANGUAGE_COOKIE_NAME`` setting) and to add
a middleware that copies the value from the old cookie to a new one and then
deletes the old one.

LANGUAGE_COOKIE_NAME
--------------------

Default: ``'django_language'``

The name of the cookie to use for the language cookie. This can be whatever
you want (but should be different from ``SESSION_COOKIE_NAME``). 
LANGUAGE_COOKIE_PATH
--------------------

Default: ``/``

The path set on the language cookie. This should either match the URL path of your
Django installation or be a parent of that path.

This is useful if you have multiple Django instances running under the same
hostname. They can use different cookie paths and each instance will only see
its own language cookie.

Be cautious when updating this setting on a production site. If you update this
setting to use a deeper path than it previously used, existing user cookies that
have the old path will not be updated. This will result in site users being
unable to switch the language as long as these cookies persist. The only safe
and reliable option to perform the switch is to change the language cookie name
permanently (via the ``LANGUAGE_COOKIE_NAME`` setting), and to add
a middleware that copies the value from the old cookie to a new one and then
deletes the one.

LANGUAGES
---------

Default: A list of all available languages. This list is continually growing
and including a copy here would inevitably become rapidly out of date. You can
see the current list of translated languages by looking in
``django/conf/global_settings.py`` (or view the `online source`_).

.. _online source: https://github.com/django/django/blob/master/django/conf/global_settings.py

The list is a list of two-tuples in the format
(``language code``, ``language name``) -- for example,
``('ja', 'Japanese')``.
This specifies which languages are available for language selection. 
Generally, the default value should suffice. Only set this setting if you want
to restrict language selection to a subset of the Django-provided languages.

If you define a custom ``LANGUAGES`` setting, you can mark the
language names as translation strings using the
:func:`~django.utils.translation.ugettext_lazy` function.

Here's a sample settings file::

    from django.utils.translation import ugettext_lazy as _

    LANGUAGES = [
        ('de', _('German')),
        ('en', _('English')),
    ]

LOCALE_PATHS
------------

Default: ``[]`` (Empty list)

A list of directories where Django looks for translation files.

Example::

    LOCALE_PATHS = [
        '/home/www/project/common_files/locale',
        '/var/local/translations/locale',
    ]

Django will look within each of these paths for the ``<locale_code>/LC_MESSAGES``
directories containing the actual translation files.

LOGGING
-------

Default: A logging configuration dictionary.

A data structure containing configuration information. The contents of
this data structure will be passed as the argument to the
configuration method described in ``LOGGING_CONFIG``.

Among other things, the default logging configuration passes HTTP 500 server
errors to an email log handler when ``DEBUG`` is ``False``. 
You can see the default logging configuration by looking in
``django/utils/log.py`` (or view the `online source`__).

__ https://github.com/django/django/blob/master/django/utils/log.py

LOGGING_CONFIG
--------------

Default: ``'logging.config.dictConfig'``

A path to a callable that will be used to configure logging in the
Django project. Points at a instance of Python's `dictConfig`_
configuration method by default.

If you set ``LOGGING_CONFIG`` to ``None``, the logging
configuration process will be skipped.

.. _dictConfig: https://docs.python.org/library/logging.config.html#configuration-dictionary-schema

MANAGERS
--------

Default: ``[]`` (Empty list)

A list in the same format as ``ADMINS`` that specifies who should get
broken link notifications when
:class:`~django.middleware.common.BrokenLinkEmailsMiddleware` is enabled.

MEDIA_ROOT
----------

Default: ``''`` (Empty string)

Absolute filesystem path to the directory that will hold user-uploaded
files.

Example: ``"/var/www/example.com/media/"``

See also ``MEDIA_URL``.

.. warning::

    ``MEDIA_ROOT`` and ``STATIC_ROOT`` must have different
    values. Before ``STATIC_ROOT`` was introduced, it was common to
    rely or fallback on ``MEDIA_ROOT`` to also serve static files;
    however, since this can have serious security implications, there is a
    validation check to prevent it.

MEDIA_URL
---------

Default: ``''`` (Empty string)

URL that handles the media served from ``MEDIA_ROOT``, used for managing
stored files . It must end in a slash if set to a non-empty value. You will
need to configure these files to be served in both development and production.

If you want to use ``{{ MEDIA_URL }}`` in your templates, add
``'django.template.context_processors.media'`` in the ``'context_processors'``
option of ``TEMPLATES``.

Example: ``"http://media.example.com/"``

.. warning::

    There are security risks if you are accepting uploaded content from
    untrusted users! See Chapter 21 for mitigation details.

.. warning::

    ``MEDIA_URL`` and ``STATIC_URL`` must have different
    values. See ``MEDIA_ROOT`` for more details.

MIDDLEWARE_CLASSES
------------------

Default::

    ['django.middleware.common.CommonMiddleware',
     'django.middleware.csrf.CsrfViewMiddleware']

A list of middleware classes to use. See Chapter 19.

MIGRATION_MODULES
-----------------

Default::

    {}  # empty dictionary

A dictionary specifying the package where migration modules can be found on a per-app basis. The default value
of this setting is an empty dictionary, but the default package name for migration modules is ``migrations``.

Example::

    {'blog': 'blog.db_migrations'}

In this case, migrations pertaining to the ``blog`` app will be contained in the ``blog.db_migrations`` package.

If you provide the ``app_label`` argument, ``makemigrations`` will
automatically create the package if it doesn't already exist.

MONTH_DAY_FORMAT
----------------

Default: ``'F j'``

The default formatting to use for date fields on Django admin change-list
pages -- and, possibly, by other parts of the system -- in cases when only the
month and day are displayed.

For example, when a Django admin change-list page is being filtered by a date
drilldown, the header for a given day displays the day and month. Different
locales have different formats. For example, U.S. English would say
"January 1," whereas Spanish might say "1 Enero."

Note that if ``USE_L10N`` is set to ``True``, then the corresponding
locale-dictated format has higher precedence and will be applied.

See also ``DATE_FORMAT``, ``DATETIME_FORMAT``,
``TIME_FORMAT`` and ``YEAR_MONTH_FORMAT``.

NUMBER_GROUPING
----------------

Default: ``0``

Number of digits grouped together on the integer part of a number.

Common use is to display a thousand separator. If this setting is ``0``, then
no grouping will be applied to the number. If this setting is greater than
``0``, then ``THOUSAND_SEPARATOR`` will be used as the separator between
those groups.

Note that if ``USE_L10N`` is set to ``True``, then the locale-dictated
format has higher precedence and will be applied instead.

See also ``DECIMAL_SEPARATOR``, ``THOUSAND_SEPARATOR`` and
``USE_THOUSAND_SEPARATOR``.

PREPEND_WWW
-----------

Default: ``False``

Whether to prepend the "www." subdomain to URLs that don't have it. This is only
used if :class:`~django.middleware.common.CommonMiddleware` is installed. See also ``APPEND_SLASH``.

ROOT_URLCONF
------------

Default: Not defined

A string representing the full Python import path to your root URLconf. For example:
``"mydjangoapps.urls"``. Can be overridden on a per-request basis by
setting the attribute ``urlconf`` on the incoming ``HttpRequest``
object. 

SECRET_KEY
----------

Default: ``''`` (Empty string)

A secret key for a particular Django installation. This is used to provide
cryptographic signing , and should be set to a unique,
unpredictable value.

django-admin startproject  automatically adds a
randomly-generated ``SECRET_KEY`` to each new project.

Django will refuse to start if ``SECRET_KEY`` is not set.

.. warning::

    **Keep this value secret.**

    Running Django with a known ``SECRET_KEY`` defeats many of Django's
    security protections, and can lead to privilege escalation and remote code
    execution vulnerabilities.

The secret key is used for:

* All sessions  if you are using
  any other session backend than ``django.contrib.sessions.backends.cache``,
  or if you use
  :class:`~django.contrib.auth.middleware.SessionAuthenticationMiddleware`
  and are using the default
  :meth:`~django.contrib.auth.models.AbstractBaseUser.get_session_auth_hash()`.
* All messages  if you are using
  :class:`~django.contrib.messages.storage.cookie.CookieStorage` or
  :class:`~django.contrib.messages.storage.fallback.FallbackStorage`.
* :mod:`Form wizard <formtools.wizard.views>` progress when using
  cookie storage with
  :class:`formtools.wizard.views.CookieWizardView`.
* All :func:`~django.contrib.auth.views.password_reset` tokens.
* All in progress :mod:`form previews <formtools.preview>`.
* Any usage of cryptographic signing , unless a
  different key is provided.

If you rotate your secret key, all of the above will be invalidated.
Secret keys are not used for passwords of users and key rotation will not
affect them.

SECURE_BROWSER_XSS_FILTER
-------------------------

Default: ``False``

If ``True``, the :class:`~django.middleware.security.SecurityMiddleware` sets
the xss protection header on all responses that do not already have it.

SECURE_CONTENT_TYPE_NOSNIFF
---------------------------

Default: ``False``

If ``True``, the :class:`~django.middleware.security.SecurityMiddleware`
sets the x content type options header on all responses that do not
already have it.

SECURE_HSTS_INCLUDE_SUBDOMAINS
------------------------------

Default: ``False``

If ``True``, the :class:`~django.middleware.security.SecurityMiddleware` adds
the ``includeSubDomains`` tag to the http-strict-transport-security
header. It has no effect unless ``SECURE_HSTS_SECONDS`` is set to a
non-zero value.

.. warning::
    Setting this incorrectly can irreversibly (for some time) break your site.
    Read the http-strict-transport-security documentation first.

SECURE_HSTS_SECONDS
-------------------

Default: ``0``

If set to a non-zero integer value, the
:class:`~django.middleware.security.SecurityMiddleware` sets the
http-strict-transport-security header on all responses that do not
already have it.

.. warning::
    Setting this incorrectly can irreversibly (for some time) break your site.
    Read the http-strict-transport-security documentation first.

SECURE_PROXY_SSL_HEADER
-----------------------

Default: ``None``

A tuple representing a HTTP header/value combination that signifies a request
is secure. This controls the behavior of the request object's ``is_secure()``
method.

This takes some explanation. By default, ``is_secure()`` is able to determine
whether a request is secure by looking at whether the requested URL uses
"https://". This is important for Django's CSRF protection, and may be used
by your own code or third-party apps.

If your Django app is behind a proxy, though, the proxy may be "swallowing" the
fact that a request is HTTPS, using a non-HTTPS connection between the proxy
and Django. In this case, ``is_secure()`` would always return ``False`` -- even
for requests that were made via HTTPS by the end user.

In this situation, you'll want to configure your proxy to set a custom HTTP
header that tells Django whether the request came in via HTTPS, and you'll want
to set ``SECURE_PROXY_SSL_HEADER`` so that Django knows what header to look
for.

You'll need to set a tuple with two elements -- the name of the header to look
for and the required value. For example::

    SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

Here, we're telling Django that we trust the ``X-Forwarded-Proto`` header
that comes from our proxy, and any time its value is ``'https'``, then the
request is guaranteed to be secure (i.e., it originally came in via HTTPS).
Obviously, you should *only* set this setting if you control your proxy or
have some other guarantee that it sets/strips this header appropriately.

Note that the header needs to be in the format as used by ``request.META`` --
all caps and likely starting with ``HTTP_``. (Remember, Django automatically
adds ``'HTTP_'`` to the start of x-header names before making the header
available in ``request.META``.)

.. warning::

    **You will probably open security holes in your site if you set this
    without knowing what you're doing. And if you fail to set it when you
    should. Seriously.**

    Make sure ALL of the following are true before setting this (assuming the
    values from the example above):

    * Your Django app is behind a proxy.
    * Your proxy strips the ``X-Forwarded-Proto`` header from all incoming
      requests. In other words, if end users include that header in their
      requests, the proxy will discard it.
    * Your proxy sets the ``X-Forwarded-Proto`` header and sends it to Django,
      but only for requests that originally come in via HTTPS.

    If any of those are not true, you should keep this setting set to ``None``
    and find another way of determining HTTPS, perhaps via custom middleware.

SECURE_REDIRECT_EXEMPT
----------------------

Default: ``[]``

If a URL path matches a regular expression in this list, the request will not be
redirected to HTTPS. If ``SECURE_SSL_REDIRECT`` is ``False``, this
setting has no effect.

SECURE_SSL_HOST
---------------

Default: ``None``

If a string (e.g. ``secure.example.com``), all SSL redirects will be directed
to this host rather than the originally-requested host
(e.g. ``www.example.com``). If ``SECURE_SSL_REDIRECT`` is ``False``, this
setting has no effect.

SECURE_SSL_REDIRECT
-------------------

Default: ``False``.

If ``True``, the :class:`~django.middleware.security.SecurityMiddleware`
redirects all non-HTTPS requests to HTTPS (except for
those URLs matching a regular expression listed in
``SECURE_REDIRECT_EXEMPT``).

.. note::

   If turning this to ``True`` causes infinite redirects, it probably means
   your site is running behind a proxy and can't tell which requests are secure
   and which are not. Your proxy likely sets a header to indicate secure
   requests; you can correct the problem by finding out what that header is and
   configuring the ``SECURE_PROXY_SSL_HEADER`` setting accordingly.

SERIALIZATION_MODULES
---------------------

Default: Not defined.

A dictionary of modules containing serializer definitions (provided as
strings), keyed by a string identifier for that serialization type. For
example, to define a YAML serializer, use::

    SERIALIZATION_MODULES = {'yaml': 'path.to.yaml_serializer'}

SERVER_EMAIL
------------

Default: ``'root@localhost'``

The email address that error messages come from, such as those sent to
``ADMINS`` and ``MANAGERS``.

.. admonition:: Why are my emails sent from a different address?

    This address is used only for error messages. It is *not* the address that
    regular email messages sent with :meth:`~django.core.mail.send_mail()`
    come from; for that, see ``DEFAULT_FROM_EMAIL``.

SHORT_DATE_FORMAT
-----------------

Default: ``m/d/Y`` (e.g. ``12/31/2003``)

An available formatting that can be used for displaying date fields on
templates. Note that if ``USE_L10N`` is set to ``True``, then the
corresponding locale-dictated format has higher precedence and will be applied.

See also ``DATE_FORMAT`` and ``SHORT_DATETIME_FORMAT``.

SHORT_DATETIME_FORMAT
---------------------

Default: ``m/d/Y P`` (e.g. ``12/31/2003 4 p.m.``)

An available formatting that can be used for displaying datetime fields on
templates. Note that if ``USE_L10N`` is set to ``True``, then the
corresponding locale-dictated format has higher precedence and will be applied.

See also ``DATE_FORMAT`` and ``SHORT_DATE_FORMAT``.

SIGNING_BACKEND
---------------

Default: ``'django.core.signing.TimestampSigner'``

The backend used for signing cookies and other data.

SILENCED_SYSTEM_CHECKS
----------------------

Default: ``[]``

A list of identifiers of messages generated by the system check framework
(i.e. ``["models.W001"]``) that you wish to permanently acknowledge and ignore.
Silenced warnings will no longer be output to the console; silenced errors
will still be printed, but will not prevent management commands from running.

TEMPLATES
---------

Default:: ``[]`` (Empty list)

A list containing the settings for all template engines to be used with
Django. Each item of the list is a dictionary containing the options for an
individual engine.

Here's a simple setup that tells the Django template engine to load templates
from the ``templates`` subdirectories inside installed applications::

    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'APP_DIRS': True,
        },
    ]

The following options are available for all backends.

BACKEND
~~~~~~~

Default: not defined

The template backend to use. The built-in template backends are:

* ``'django.template.backends.django.DjangoTemplates'``
* ``'django.template.backends.jinja2.Jinja2'``

You can use a template backend that doesn't ship with Django by setting
``BACKEND`` to a fully-qualified path (i.e. ``'mypackage.whatever.Backend'``).

NAME
~~~~

Default: see below

The alias for this particular template engine. It's an identifier that allows
selecting an engine for rendering. Aliases must be unique across all
configured template engines.

It defaults to the name of the module defining the engine class, i.e. the
next to last piece of ``BACKEND <TEMPLATES-BACKEND>``, when it isn't
provided. For example if the backend is ``'mypackage.whatever.Backend'`` then
its default name is ``'whatever'``.

DIRS
~~~~

Default:: ``[]`` (Empty list)

Directories where the engine should look for template source files, in search
order.

APP_DIRS
~~~~~~~~

Default:: ``False``

Whether the engine should look for template source files inside installed
applications.

OPTIONS
~~~~~~~

Default:: ``{}`` (Empty dict)

Extra parameters to pass to the template backend. Available parameters vary
depending on the template backend.

TEMPLATE_DEBUG
--------------

Default: ``False``

A boolean that turns on/off template debug mode. If this is ``True``, the fancy
error page will display a detailed report for any exception raised during
template rendering. This report contains the relevant snippet of the template,
with the appropriate line highlighted.

Note that Django only displays fancy error pages if ``DEBUG`` is ``True``, so
you'll want to set that to take advantage of this setting.

See also ``DEBUG``.

TEST_RUNNER
-----------

Default: ``'django.test.runner.DiscoverRunner'``

The name of the class to use for starting the test suite. 

TEST_NON_SERIALIZED_APPS
------------------------

Default: ``[]``

In order to restore the database state between tests for
``TransactionTestCase``\s and database backends without transactions, Django
will serialize the contents of all apps when it starts the test run so it can then reload from that copy before tests
that need it.

This slows down the startup time of the test runner; if you have apps that
you know don't need this feature, you can add their full names in here (e.g.
``'django.contrib.contenttypes'``) to exclude them from this serialization
process.

THOUSAND_SEPARATOR
------------------

Default: ``,`` (Comma)

Default thousand separator used when formatting numbers. This setting is
used only when ``USE_THOUSAND_SEPARATOR`` is ``True`` and
``NUMBER_GROUPING`` is greater than ``0``.

Note that if ``USE_L10N`` is set to ``True``, then the locale-dictated
format has higher precedence and will be applied instead.

See also ``NUMBER_GROUPING``, ``DECIMAL_SEPARATOR`` and
``USE_THOUSAND_SEPARATOR``.

TIME_FORMAT
-----------

Default: ``'P'`` (e.g. ``4 p.m.``)

The default formatting to use for displaying time fields in any part of the
system. Note that if ``USE_L10N`` is set to ``True``, then the
locale-dictated format has higher precedence and will be applied instead.

See also ``DATE_FORMAT`` and ``DATETIME_FORMAT``.

TIME_INPUT_FORMATS
------------------

Default::

    [
        '%H:%M:%S',     # '14:30:59'
        '%H:%M:%S.%f',  # '14:30:59.000200'
        '%H:%M',        # '14:30'
    ]

A list of formats that will be accepted when inputting data on a time field.
Formats will be tried in order, using the first valid one. Note that these
format strings use Python's datetime_ module syntax, not the format strings
from the ``date`` Django template tag.

When ``USE_L10N`` is ``True``, the locale-dictated format has higher
precedence and will be applied instead.

See also ``DATE_INPUT_FORMATS`` and ``DATETIME_INPUT_FORMATS``.

.. _datetime: https://docs.python.org/library/datetime.html#strftime-strptime-behavior

TIME_ZONE
---------

Default: ``'America/Chicago'``

A string representing the time zone for this installation, or ``None``. See
the `list of time zones`_.

.. note::
    Since Django was first released with the ``TIME_ZONE`` set to
    ``'America/Chicago'``, the global setting (used if nothing is defined in
    your project's ``settings.py``) remains ``'America/Chicago'`` for backwards
    compatibility. New project templates default to ``'UTC'``.

Note that this isn't necessarily the time zone of the server. For example, one
server may serve multiple Django-powered sites, each with a separate time zone
setting.

When ``USE_TZ`` is ``False``, this is the time zone in which Django
will store all datetimes. When ``USE_TZ`` is ``True``, this is the
default time zone that Django will use to display datetimes in templates and
to interpret datetimes entered in forms.

Django sets the ``os.environ['TZ']`` variable to the time zone you specify in
the ``TIME_ZONE`` setting. Thus, all your views and models will
automatically operate in this time zone. However, Django won't set the ``TZ``
environment variable under the following conditions:

* If you're using the manual configuration option as described in
  manually configuring settings,or

* If you specify ``TIME_ZONE = None``. This will cause Django to fall back to
  using the system timezone. However, this is discouraged when ``USE_TZ
  = True``, because it makes conversions between local time and UTC
  less reliable.

If Django doesn't set the ``TZ`` environment variable, it's up to you
to ensure your processes are running in the correct environment.

.. note::
    Django cannot reliably use alternate time zones in a Windows environment.
    If you're running Django on Windows, ``TIME_ZONE`` must be set to
    match the system time zone.

.. _list of time zones: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones

.. _pytz: http://pytz.sourceforge.net/

USE_ETAGS
---------

Default: ``False``

A boolean that specifies whether to output the "Etag" header. This saves
bandwidth but slows down performance. This is used by the ``CommonMiddleware``
and in the``Cache Framework``.

USE_I18N
--------

Default: ``True``

A boolean that specifies whether Django's translation system should be enabled.
This provides an easy way to turn it off, for performance. If this is set to
``False``, Django will make some optimizations so as not to load the
translation machinery.

See also ``LANGUAGE_CODE``, ``USE_L10N`` and ``USE_TZ``.

USE_L10N
--------

Default: ``False``

A boolean that specifies if localized formatting of data will be enabled by
default or not. If this is set to ``True``, e.g. Django will display numbers and
dates using the format of the current locale.

See also ``LANGUAGE_CODE``, ``USE_I18N`` and ``USE_TZ``.

.. note::

    The default :file:`settings.py` file created by ``django-admin
    startproject`` includes ``USE_L10N = True`` for convenience.

USE_THOUSAND_SEPARATOR
----------------------

Default: ``False``

A boolean that specifies whether to display numbers using a thousand separator.
When ``USE_L10N`` is set to ``True`` and if this is also set to
``True``, Django will use the values of ``THOUSAND_SEPARATOR`` and
``NUMBER_GROUPING`` to format numbers.

See also ``DECIMAL_SEPARATOR``, ``NUMBER_GROUPING`` and
``THOUSAND_SEPARATOR``.

USE_TZ
------

Default: ``False``

A boolean that specifies if datetimes will be timezone-aware by default or not.
If this is set to ``True``, Django will use timezone-aware datetimes internally.
Otherwise, Django will use naive datetimes in local time.

See also ``TIME_ZONE``, ``USE_I18N`` and ``USE_L10N``.

.. note::

    The default :file:`settings.py` file created by
    django-admin startproject  includes
    ``USE_TZ = True`` for convenience.

USE_X_FORWARDED_HOST
--------------------

Default: ``False``

A boolean that specifies whether to use the X-Forwarded-Host header in
preference to the Host header. This should only be enabled if a proxy
which sets this header is in use.

WSGI_APPLICATION
----------------

Default: ``None``

The full Python path of the WSGI application object that Django's built-in
servers (e.g. ``runserver``) will use. The ``django-admin
startproject`` management command will create a simple
``wsgi.py`` file with an ``application`` callable in it, and point this setting
to that ``application``.

If not set, the return value of ``django.core.wsgi.get_wsgi_application()``
will be used. In this case, the behavior of ``runserver`` will be
identical to previous Django versions.

YEAR_MONTH_FORMAT
-----------------

Default: ``'F Y'``

The default formatting to use for date fields on Django admin change-list
pages -- and, possibly, by other parts of the system -- in cases when only the
year and month are displayed.

For example, when a Django admin change-list page is being filtered by a date
drilldown, the header for a given month displays the month and the year.
Different locales have different formats. For example, U.S. English would say
"January 2006," whereas another locale might say "2006/January."

Note that if ``USE_L10N`` is set to ``True``, then the corresponding
locale-dictated format has higher precedence and will be applied.

See also ``DATE_FORMAT``, ``DATETIME_FORMAT``, ``TIME_FORMAT``
and ``MONTH_DAY_FORMAT``.

X_FRAME_OPTIONS
---------------

Default: ``'SAMEORIGIN'``

The default value for the X-Frame-Options header used by
:class:`~django.middleware.clickjacking.XFrameOptionsMiddleware`. See the
clickjacking protection  documentation.


Auth
====

Settings for :mod:`django.contrib.auth`.

AUTHENTICATION_BACKENDS
-----------------------

Default: ``['django.contrib.auth.backends.ModelBackend']``

A list of authentication backend classes (as strings) to use when attempting to
authenticate a user. See the authentication backends documentation for details.

AUTH_USER_MODEL
---------------

Default: 'auth.User'

The model to use to represent a User.

.. warning::
    You cannot change the AUTH_USER_MODEL setting during the lifetime of
    a project (i.e. once you have made and migrated models that depend on it)
    without serious effort. It is intended to be set at the project start,
    and the model it refers to must be available in the first migration of
    the app that it lives in.

LOGIN_REDIRECT_URL
------------------

Default: ``'/accounts/profile/'``

The URL where requests are redirected after login when the
``contrib.auth.login`` view gets no ``next`` parameter.

This is used by the :func:`~django.contrib.auth.decorators.login_required`
decorator, for example.

This setting also accepts view function names and named URL patterns which can
be used to reduce configuration duplication since you don't have to define the
URL in two places (``settings`` and URLconf).

LOGIN_URL
---------

Default: ``'/accounts/login/'``

The URL where requests are redirected for login, especially when using the
:func:`~django.contrib.auth.decorators.login_required` decorator.

This setting also accepts view function names and named URL patterns which can
be used to reduce configuration duplication since you don't have to define the
URL in two places (``settings`` and URLconf).

LOGOUT_URL
----------

Default: ``'/accounts/logout/'``

LOGIN_URL counterpart.

PASSWORD_RESET_TIMEOUT_DAYS
---------------------------

Default: ``3``

The number of days a password reset link is valid for. Used by the
:mod:`django.contrib.auth` password reset mechanism.

PASSWORD_HASHERS
----------------

Default::

    ['django.contrib.auth.hashers.PBKDF2PasswordHasher',
     'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
     'django.contrib.auth.hashers.BCryptPasswordHasher',
     'django.contrib.auth.hashers.SHA1PasswordHasher',
     'django.contrib.auth.hashers.MD5PasswordHasher',
     'django.contrib.auth.hashers.UnsaltedMD5PasswordHasher',
     'django.contrib.auth.hashers.CryptPasswordHasher']

.. _settings-messages:

Messages
========

Settings for :mod:`django.contrib.messages`.

MESSAGE_LEVEL
-------------

Default: ``messages.INFO``

Sets the minimum message level that will be recorded by the messages
framework.

.. admonition:: Important

   If you override ``MESSAGE_LEVEL`` in your settings file and rely on any of
   the built-in constants, you must import the constants module directly to
   avoid the potential for circular imports, e.g.::

       from django.contrib.messages import constants as message_constants
       MESSAGE_LEVEL = message_constants.DEBUG

   If desired, you may specify the numeric values for the constants directly
   according to the values in the above constants table.

MESSAGE_STORAGE
---------------

Default: ``'django.contrib.messages.storage.fallback.FallbackStorage'``

Controls where Django stores message data. Valid values are:

* ``'django.contrib.messages.storage.fallback.FallbackStorage'``
* ``'django.contrib.messages.storage.session.SessionStorage'``
* ``'django.contrib.messages.storage.cookie.CookieStorage'``

The backends that use cookies --
:class:`~django.contrib.messages.storage.cookie.CookieStorage` and
:class:`~django.contrib.messages.storage.fallback.FallbackStorage` --
use the value of ``SESSION_COOKIE_DOMAIN``, ``SESSION_COOKIE_SECURE``
and ``SESSION_COOKIE_HTTPONLY`` when setting their cookies.

MESSAGE_TAGS
------------

Default::

    {messages.DEBUG: 'debug',
    messages.INFO: 'info',
    messages.SUCCESS: 'success',
    messages.WARNING: 'warning',
    messages.ERROR: 'error'}

This sets the mapping of message level to message tag, which is typically
rendered as a CSS class in HTML. If you specify a value, it will extend
the default. This means you only have to specify those values which you need
to override.

.. admonition:: Important

   If you override ``MESSAGE_TAGS`` in your settings file and rely on any of
   the built-in constants, you must import the ``constants`` module directly to
   avoid the potential for circular imports, e.g.::

       from django.contrib.messages import constants as message_constants
       MESSAGE_TAGS = {message_constants.INFO: ''}

   If desired, you may specify the numeric values for the constants directly
   according to the values in the above constants table.

.. _settings-sessions:

Sessions
========

Settings for :mod:`django.contrib.sessions`.

SESSION_CACHE_ALIAS
-------------------

Default: ``default``

If you're using cache-based session storage,
this selects the cache to use.

SESSION_COOKIE_AGE
------------------

Default: ``1209600`` (2 weeks, in seconds)

The age of session cookies, in seconds.

SESSION_COOKIE_DOMAIN
---------------------

Default: ``None``

The domain to use for session cookies. Set this to a string such as
``".example.com"`` (note the leading dot!) for cross-domain cookies, or use
``None`` for a standard domain cookie.

Be cautious when updating this setting on a production site. If you update
this setting to enable cross-domain cookies on a site that previously used
standard domain cookies, existing user cookies will be set to the old
domain. This may result in them being unable to log in as long as these cookies
persist.

This setting also affects cookies set by :mod:`django.contrib.messages`.

SESSION_COOKIE_HTTPONLY
-----------------------

Default: ``True``

Whether to use ``HTTPOnly`` flag on the session cookie. If this is set to
``True``, client-side JavaScript will not to be able to access the
session cookie.

HTTPOnly_ is a flag included in a Set-Cookie HTTP response header. It
is not part of the :rfc:`2109` standard for cookies, and it isn't honored
consistently by all browsers. However, when it is honored, it can be a
useful way to mitigate the risk of client side script accessing the
protected cookie data.

Turning it on makes it less trivial for an attacker to escalate a cross-site
scripting vulnerability into full hijacking of a user's session. There's not
much excuse for leaving this off, either: if your code depends on reading
session cookies from Javascript, you're probably doing it wrong.

.. _HTTPOnly: https://www.owasp.org/index.php/HTTPOnly

SESSION_COOKIE_NAME
-------------------

Default: ``'sessionid'``

The name of the cookie to use for sessions. This can be whatever you want (but
should be different from ``LANGUAGE_COOKIE_NAME``).

SESSION_COOKIE_PATH
-------------------

Default: ``'/'``

The path set on the session cookie. This should either match the URL path of your
Django installation or be parent of that path.

This is useful if you have multiple Django instances running under the same
hostname. They can use different cookie paths, and each instance will only see
its own session cookie.

SESSION_COOKIE_SECURE
---------------------

Default: ``False``

Whether to use a secure cookie for the session cookie. If this is set to
``True``, the cookie will be marked as "secure," which means browsers may
ensure that the cookie is only sent under an HTTPS connection.

Since it's trivial for a packet sniffer (e.g. `Firesheep`_) to hijack a user's
session if the session cookie is sent unencrypted, there's really no good
excuse to leave this off. It will prevent you from using sessions on insecure
requests and that's a good thing.

.. _Firesheep: http://codebutler.com/firesheep

SESSION_ENGINE
--------------

Default: ``django.contrib.sessions.backends.db``

Controls where Django stores session data. Included engines are:

* ``'django.contrib.sessions.backends.db'``
* ``'django.contrib.sessions.backends.file'``
* ``'django.contrib.sessions.backends.cache'``
* ``'django.contrib.sessions.backends.cached_db'``
* ``'django.contrib.sessions.backends.signed_cookies'``


SESSION_EXPIRE_AT_BROWSER_CLOSE
-------------------------------

Default: ``False``

Whether to expire the session when the user closes their browser.

SESSION_FILE_PATH
-----------------

Default: ``None``

If you're using file-based session storage, this sets the directory in
which Django will store session data. When the default value (``None``) is
used, Django will use the standard temporary directory for the system.


SESSION_SAVE_EVERY_REQUEST
--------------------------

Default: ``False``

Whether to save the session data on every request. If this is ``False``
(default), then the session data will only be saved if it has been modified --
that is, if any of its dictionary values have been assigned or deleted.

SESSION_SERIALIZER
------------------

Default: ``'django.contrib.sessions.serializers.JSONSerializer'``

Full import path of a serializer class to use for serializing session data.
Included serializers are:

* ``'django.contrib.sessions.serializers.PickleSerializer'``
* ``'django.contrib.sessions.serializers.JSONSerializer'``

Sites
=====

Settings for :mod:`django.contrib.sites`.

SITE_ID
-------

Default: Not defined

The ID, as an integer, of the current site in the ``django_site`` database
table. This is used so that application data can hook into specific sites
and a single database can manage content for multiple sites.


.. _settings-staticfiles:

Static files
============

Settings for :mod:`django.contrib.staticfiles`.

STATIC_ROOT
-----------

Default: ``None``

The absolute path to the directory where ``collectstatic`` will collect
static files for deployment.

Example: ``"/var/www/example.com/static/"``

If the staticfiles contrib app is enabled
(default) the ``collectstatic`` management command will collect static
files into this directory. See the howto on managing static
files for more details about usage.

.. warning::

    This should be an (initially empty) destination directory for collecting
    your static files from their permanent locations into one directory for
    ease of deployment; it is **not** a place to store your static files
    permanently. You should do that in directories that will be found by
    staticfiles’
    ``finders<STATICFILES_FINDERS>``, which by default, are
    ``'static/'`` app sub-directories and any directories you include in
    ``STATICFILES_DIRS``).

STATIC_URL
----------

Default: ``None``

URL to use when referring to static files located in ``STATIC_ROOT``.

Example: ``"/static/"`` or ``"http://static.example.com/"``

If not ``None``, this will be used as the base path for
asset definitions (the ``Media`` class) and the
staticfiles app.

It must end in a slash if set to a non-empty value.

You may need to configure these files to be served in development and will
definitely need to do so in production .

STATICFILES_DIRS
----------------

Default: ``[]`` (Empty list)

This setting defines the additional locations the staticfiles app will traverse
if the ``FileSystemFinder`` finder is enabled, e.g. if you use the
``collectstatic`` or ``findstatic`` management command or use the
static file serving view.

This should be set to a list of strings that contain full paths to
your additional files directory(ies) e.g.::

    STATICFILES_DIRS = [
        "/home/special.polls.com/polls/static",
        "/home/polls.com/polls/static",
        "/opt/webfiles/common",
    ]

Note that these paths should use Unix-style forward slashes, even on Windows
(e.g. ``"C:/Users/user/mysite/extra_static_content"``).

Prefixes (optional)
~~~~~~~~~~~~~~~~~~~

In case you want to refer to files in one of the locations with an additional
namespace, you can **optionally** provide a prefix as ``(prefix, path)``
tuples, e.g.::

    STATICFILES_DIRS = [
        # ...
        ("downloads", "/opt/webfiles/stats"),
    ]

For example, assuming you have ``STATIC_URL`` set to ``'/static/'``, the
``collectstatic`` management command would collect the "stats" files
in a ``'downloads'`` subdirectory of ``STATIC_ROOT``.

This would allow you to refer to the local file
``'/opt/webfiles/stats/polls_20101022.tar.gz'`` with
``'/static/downloads/polls_20101022.tar.gz'`` in your templates, e.g.:

.. code-block:: html+django

    <a href="{% static "downloads/polls_20101022.tar.gz" %}">

STATICFILES_STORAGE
-------------------

Default: ``'django.contrib.staticfiles.storage.StaticFilesStorage'``

The file storage engine to use when collecting static files with the
``collectstatic`` management command.

A ready-to-use instance of the storage backend defined in this setting
can be found at ``django.contrib.staticfiles.storage.staticfiles_storage``.

STATICFILES_FINDERS
-------------------

Default::

    ["django.contrib.staticfiles.finders.FileSystemFinder",
     "django.contrib.staticfiles.finders.AppDirectoriesFinder"]

The list of finder backends that know how to find static files in
various locations.

The default will find files stored in the ``STATICFILES_DIRS`` setting
(using ``django.contrib.staticfiles.finders.FileSystemFinder``) and in a
``static`` subdirectory of each app (using
``django.contrib.staticfiles.finders.AppDirectoriesFinder``). If multiple
files with the same name are present, the first file that is found will be
used.

One finder is disabled by default:
``django.contrib.staticfiles.finders.DefaultStorageFinder``. If added to
your ``STATICFILES_FINDERS`` setting, it will look for static files in
the default file storage as defined by the ``DEFAULT_FILE_STORAGE``
setting.

.. note::

    When using the ``AppDirectoriesFinder`` finder, make sure your apps
    can be found by staticfiles. Simply add the app to the
    ``INSTALLED_APPS`` setting of your site.

Static file finders are currently considered a private interface, and this
interface is thus undocumented.

