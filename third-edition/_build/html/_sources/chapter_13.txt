============================
Chapter 13: Deploying Django
============================

This chapter covers the last essential step of building a Django application:
deploying it to a production server.

If you've been following along with our ongoing examples, you've likely been
using the ``runserver``, which makes things very easy -- with ``runserver``,
you don't have to worry about Web server setup. But ``runserver`` is intended
only for development on your local machine, not for exposure on the public Web.
To deploy your Django application, you'll need to hook it into an
industrial-strength Web server such as Apache. In this chapter, we'll show you
how to do that -- but, first, we'll give you a checklist of things to do in
your codebase before you go live.

Preparing Your Codebase for Production
======================================

Deployment checklist
--------------------

The Internet is a hostile environment. Before deploying your Django project,
you should take some time to review your settings, with security, performance,
and operations in mind.

Django includes many security features . Some are
built-in and always enabled. Others are optional because they aren't always
appropriate, or because they're inconvenient for development. For example,
forcing HTTPS may not be suitable for all websites, and it's impractical for
local development.

Performance optimizations are another category of trade-offs with convenience.
For instance, caching is useful in production, less so for local development.
Error reporting needs are also widely different.

The following checklist includes settings that:

- must be set properly for Django to provide the expected level of security;
- are expected to be different in each environment;
- enable optional security features;
- enable performance optimizations;
- provide error reporting.

Many of these settings are sensitive and should be treated as confidential. If
you're releasing the source code for your project, a common practice is to
publish suitable settings for development, and to use a private settings
module for production.

Some of the checks described below can be automated using the
``--deploy`` option of the ``check`` command. Be sure to run it
against your production settings file as described in the option's
documentation.

Critical settings
=================

SECRET_KEY
----------

**The secret key must be a large random value and it must be kept secret.**

Make sure that the key used in production isn't used anywhere else and avoid
committing it to source control. This reduces the number of vectors from which
an attacker may acquire the key.

Instead of hardcoding the secret key in your settings module, consider loading
it from an environment variable::

    import os
    SECRET_KEY = os.environ['SECRET_KEY']

or from a file::

    with open('/etc/secret_key.txt') as f:
        SECRET_KEY = f.read().strip()

DEBUG
-----

**You must never enable debug in production.**

When we created a project in Chapter 1, the command
``django-admin startproject`` created a ``settings.py`` file with ``DEBUG``
set to ``True``. Many internal parts of Django check this setting and change
their behavior if ``DEBUG`` mode is on. For example, if ``DEBUG`` is set to
``True``, then:

* All database queries will be saved in memory as the object
  ``django.db.connection.queries``. As you can imagine, this eats up
  memory!

* Any 404 error will be rendered by Django's special 404 error page
  (covered in Chapter 3) rather than returning a proper 404 response. This
  page contains potentially sensitive information and should *not* be
  exposed to the public Internet.

* Any uncaught exception in your Django application -- from basic Python
  syntax errors to database errors to template syntax errors -- will be
  rendered by the Django pretty error page that you've likely come to know
  and love. This page contains even *more* sensitive information than the
  404 page and should *never* be exposed to the public.

In short, setting ``DEBUG`` to ``True`` tells Django to assume only trusted
developers are using your site. The Internet is full of untrustworthy
hooligans, and the first thing you should do when you're preparing your
application for deployment is set ``DEBUG`` to ``False``.

Environment-specific settings
=============================

ALLOWED_HOSTS
-------------

When ``DEBUG = False <DEBUG>``, Django doesn't work at all without a
suitable value for ``ALLOWED_HOSTS``.

This setting is required to protect your site against some CSRF attacks. If
you use a wildcard, you must perform your own validation of the ``Host`` HTTP
header, or otherwise ensure that you aren't vulnerable to this category of
attacks.

CACHES
------

If you're using a cache, connection parameters may be different in development
and in production.

Cache servers often have weak authentication. Make sure they only accept
connections from your application servers.

If you're using Memcached, consider using cached sessions
to improve performance.

DATABASES
---------

Database connection parameters are probably different in development and in
production.

Database passwords are very sensitive. You should protect them exactly like
``SECRET_KEY``.

For maximum security, make sure database servers only accept connections from
your application servers.

If you haven't set up backups for your database, do it right now!

EMAIL_BACKEND and related settings
----------------------------------

If your site sends emails, these values need to be set correctly.

STATIC_ROOT and STATIC_URL
--------------------------

Static files are automatically served by the development server. In
production, you must define a ``STATIC_ROOT`` directory where
``collectstatic`` will copy them.


MEDIA_ROOT and MEDIA_URL
------------------------

Media files are uploaded by your users. They're untrusted! Make sure your web
server never attempt to interpret them. For instance, if a user uploads a
``.php`` file , the web server shouldn't execute it.

Now is a good time to check your backup strategy for these files.

HTTPS
=====

Any website which allows users to log in should enforce site-wide HTTPS to
avoid transmitting access tokens in clear. In Django, access tokens include
the login/password, the session cookie, and password reset tokens. (You can't
do much to protect password reset tokens if you're sending them by email.)

Protecting sensitive areas such as the user account or the admin isn't
sufficient, because the same session cookie is used for HTTP and HTTPS. Your
web server must redirect all HTTP traffic to HTTPS, and only transmit HTTPS
requests to Django.

Once you've set up HTTPS, enable the following settings.

``CSRF_COOKIE_SECURE``
-----------------------------

Set this to ``True`` to avoid transmitting the CSRF cookie over HTTP
accidentally.

``SESSION_COOKIE_SECURE``
--------------------------------

Set this to ``True`` to avoid transmitting the session cookie over HTTP
accidentally.

Performance optimizations
=========================

Setting ``DEBUG = False <DEBUG>`` disables several features that are
only useful in development. In addition, you can tune the following settings.

``CONN_MAX_AGE``
-----------------------

Enabling persistent database connections
can result in a nice speed-up when
connecting to the database accounts for a significant part of the request
processing time.

This helps a lot on virtualized hosts with limited network performance.

``TEMPLATES``
--------------------

Enabling the cached template loader often improves performance drastically, as
it avoids compiling each template every time it needs to be rendered. See the
template loaders docs  for more information.

Error reporting
===============

By the time you push your code to production, it's hopefully robust, but you
can't rule out unexpected errors. Thankfully, Django can capture errors and
notify you accordingly.

``LOGGING``
------------------

Review your logging configuration before putting your website in production,
and check that it works as expected as soon as you have received some traffic.

``ADMINS`` and ``MANAGERS``
-----------------------------------------

``ADMINS`` will be notified of 500 errors by email.

``MANAGERS`` will be notified of 404 errors.
``IGNORABLE_404_URLS`` can help filter out spurious reports.

.. admonition:: Error reporting by email doesn't scale very well

    Consider using an error monitoring system such as Sentry_ before your
    inbox is flooded by reports. Sentry can also aggregate logs.

    .. _Sentry: http://sentry.readthedocs.org/en/latest/

Customize the default error views
---------------------------------

Django includes default views and templates for several HTTP error codes. You
may want to override the default templates by creating the following templates
in your root template directory: ``404.html``, ``500.html``, ``403.html``, and
``400.html``. The default views should suffice for 99% of Web applications, but
if you desire to customize them, see these instructions which also contain
details about the default templates:

* ``http_not_found_view``
* ``http_internal_server_error_view``
* ``http_forbidden_view``
* ``http_bad_request_view``

Python Options
==============

It's strongly recommended that you invoke the Python process running your
Django application using the `-R`_ option or with the
:envvar:`PYTHONHASHSEED` environment variable set to ``random``.

These options help protect your site from denial-of-service (DoS)
attacks triggered by carefully crafted inputs. Such an attack can
drastically increase CPU usage by causing worst-case performance when
creating ``dict`` instances. See `oCERT advisory #2011-003
<http://www.ocert.org/advisories/ocert-2011-003.html>`_ for more information.

.. _-r: https://docs.python.org/using/cmdline.html#cmdoption-R

Using Different Settings for Production
=======================================

So far in this book, we've dealt with only a single settings file: the
``settings.py`` generated by ``django-admin.py startproject``. But as you get
ready to deploy, you'll likely find yourself needing multiple settings files to
keep your development environment isolated from your production environment.
(For example, you probably won't want to change ``DEBUG`` from ``False`` to
``True`` whenever you want to test code changes on your local machine.) Django
makes this very easy by allowing you to use multiple settings files.

If you'd like to organize your settings files into "production" and
"development" settings, you can accomplish this in one of three ways:

* Set up two full-blown, independent settings files.

* Set up a "base" settings file (say, for development) and a second (say,
  production) settings file that merely imports from the first one and
  defines whatever overrides it needs to define.

* Use only a single settings file that has Python logic to change the
  settings based on context.

We'll take these one at a time.

First, the most basic approach is to define two separate settings files. If
you're following along, you've already got ``settings.py``. Now, just make a
copy of it called ``settings_production.py``. (We made this name up; you can
call it whatever you want.) In this new file, change ``DEBUG``, etc.

The second approach is similar but cuts down on redundancy. Instead of having
two settings files whose contents are mostly similar, you can treat one as the
"base" file and create another file that imports from it. For example::

    # settings.py

    DEBUG = True
    TEMPLATE_DEBUG = DEBUG

    DATABASE_ENGINE = 'postgresql_psycopg2'
    DATABASE_NAME = 'devdb'
    DATABASE_USER = ''
    DATABASE_PASSWORD = ''
    DATABASE_PORT = ''

    # ...

    # settings_production.py

    from settings import *

    DEBUG = TEMPLATE_DEBUG = False
    DATABASE_NAME = 'production'
    DATABASE_USER = 'app'
    DATABASE_PASSWORD = 'letmein'

Here, ``settings_production.py`` imports everything from ``settings.py`` and
just redefines the settings that are particular to production. In this case,
``DEBUG`` is set to ``False``, but we've also set different database access
parameters for the production setting. (The latter goes to show that you can
redefine *any* setting, not just the basic ones like ``DEBUG``.)

Finally, the most concise way of accomplishing two settings environments is to
use a single settings file that branches based on the environment. One way to
do this is to check the current hostname. For example::

    # settings.py

    import socket

    if socket.gethostname() == 'my-laptop':
        DEBUG = TEMPLATE_DEBUG = True
    else:
        DEBUG = TEMPLATE_DEBUG = False

    # ...

Here, we import the ``socket`` module from Python's standard library and use it
to check the current system's hostname. We can check the hostname to determine
whether the code is being run on the production server.

A core lesson here is that settings files are *just Python code*. They can
import from other files, they can execute arbitrary logic, etc. Just make sure
that, if you go down this road, the Python code in your settings files is
bulletproof. If it raises any exceptions, Django will likely crash badly.

.. admonition:: Renaming settings.py

    Feel free to rename your ``settings.py`` to ``settings_dev.py`` or
    ``settings/dev.py`` or ``foobar.py`` -- Django doesn't care, as long as
    you tell it what settings file you're using.

    But if you *do* rename the ``settings.py`` file that is generated by
    ``django-admin.py startproject``, you'll find that ``manage.py`` will give
    you an error message saying that it can't find the settings. That's because
    it tries to import a module called ``settings``. You can fix this either by
    editing ``manage.py`` to change ``settings`` to the name of your module, or
    by using ``django-admin.py`` instead of ``manage.py``. In the latter case,
    you'll need to set the ``DJANGO_SETTINGS_MODULE`` environment variable to
    the Python path to your settings file (e.g., ``'mysite.settings'``).


Deploying Django with Apache and mod_wsgi
==========================================

Deploying Django with Apache_ and `mod_wsgi`_ is a tried and tested way to get
Django into production.

.. _Apache: http://httpd.apache.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/

mod_wsgi is an Apache module which can host any Python WSGI_ application,
including Django. Django will work with any version of Apache which supports
mod_wsgi.

.. _WSGI: http://www.wsgi.org

The `official mod_wsgi documentation`_ is fantastic; it's your source for all
the details about how to use mod_wsgi. You'll probably want to start with the
`installation and configuration documentation`_.

.. _official mod_wsgi documentation: http://code.google.com/p/modwsgi/
.. _installation and configuration documentation: http://code.google.com/p/modwsgi/wiki/InstallationInstructions

Basic configuration
===================

Once you've got mod_wsgi installed and activated, edit your Apache server's
``httpd.conf`` file and add the following. If you are using a version of Apache
older than 2.4, replace ``Require all granted`` with ``Allow from all`` and
also add the line ``Order deny,allow`` above it.

.. code-block:: apache

    WSGIScriptAlias / /path/to/mysite.com/mysite/wsgi.py
    WSGIPythonPath /path/to/mysite.com

    <Directory /path/to/mysite.com/mysite>
    <Files wsgi.py>
    Require all granted
    </Files>
    </Directory>

The first bit in the ``WSGIScriptAlias`` line is the base URL path you want to
serve your application at (``/`` indicates the root url), and the second is the
location of a "WSGI file" -- see below -- on your system, usually inside of
your project package (``mysite`` in this example). This tells Apache to serve
any request below the given URL using the WSGI application defined in that
file.

The ``WSGIPythonPath`` line ensures that your project package is available for
import on the Python path; in other words, that ``import mysite`` works.

The ``<Directory>`` piece just ensures that Apache can access your
:file:`wsgi.py` file.

Next we'll need to ensure this :file:`wsgi.py` with a WSGI application object
exists. As of Django version 1.4, ``startproject`` will have created one
for you; otherwise, you'll need to create it. See the WSGI overview
for the default contents you
should put in this file, and what else you can add to it.

.. warning::

    If multiple Django sites are run in a single mod_wsgi process, all of them
    will use the settings of whichever one happens to run first. This can be
    solved by changing::

        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "{{ project_name }}.settings")

    in ``wsgi.py``, to::

        os.environ["DJANGO_SETTINGS_MODULE"] = "{{ project_name }}.settings"

    or by using mod_wsgi daemon mode and ensuring that each
    site runs in its own daemon process.

Using a virtualenv
==================

If you install your project's Python dependencies inside a `virtualenv`_,
you'll need to add the path to this virtualenv's ``site-packages`` directory to
your Python path as well. To do this, add an additional path to your
``WSGIPythonPath`` directive, with multiple paths separated by a colon (``:``)
if using a UNIX-like system, or a semicolon (``;``) if using Windows. If any
part of a directory path contains a space character, the complete argument
string to ``WSGIPythonPath`` must be quoted::

    WSGIPythonPath /path/to/mysite.com:/path/to/your/venv/lib/python3.X/site-packages

Make sure you give the correct path to your virtualenv, and replace
``python3.X`` with the correct Python version (e.g. ``python3.4``).

.. _virtualenv: http://www.virtualenv.org

.. _daemon-mode:

Using mod_wsgi daemon mode
==========================

"Daemon mode" is the recommended mode for running mod_wsgi (on non-Windows
platforms). To create the required daemon process group and delegate the
Django instance to run in it, you will need to add appropriate
``WSGIDaemonProcess`` and ``WSGIProcessGroup`` directives. A further change
required to the above configuration if you use daemon mode is that you can't
use ``WSGIPythonPath``; instead you should use the ``python-path`` option to
``WSGIDaemonProcess``, for example::

    WSGIDaemonProcess example.com python-path=/path/to/mysite.com:/path/to/venv/lib/python2.7/site-packages
    WSGIProcessGroup example.com

See the official mod_wsgi documentation for `details on setting up daemon
mode`_.

.. _details on setting up daemon mode: http://code.google.com/p/modwsgi/wiki/QuickConfigurationGuide#Delegation_To_Daemon_Process

.. _serving-files:

Serving files
=============

Django doesn't serve files itself; it leaves that job to whichever Web
server you choose.

We recommend using a separate Web server -- i.e., one that's not also running
Django -- for serving media. Here are some good choices:

* Nginx_
* A stripped-down version of Apache_

If, however, you have no option but to serve media files on the same Apache
``VirtualHost`` as Django, you can set up Apache to serve some URLs as
static media, and others using the mod_wsgi interface to Django.

This example sets up Django at the site root, but explicitly serves
``robots.txt``, ``favicon.ico``, any CSS file, and anything in the
``/static/`` and ``/media/`` URL space as a static file. All other URLs
will be served using mod_wsgi::

    Alias /robots.txt /path/to/mysite.com/static/robots.txt
    Alias /favicon.ico /path/to/mysite.com/static/favicon.ico

    Alias /media/ /path/to/mysite.com/media/
    Alias /static/ /path/to/mysite.com/static/

    <Directory /path/to/mysite.com/static>
    Require all granted
    </Directory>

    <Directory /path/to/mysite.com/media>
    Require all granted
    </Directory>

    WSGIScriptAlias / /path/to/mysite.com/mysite/wsgi.py

    <Directory /path/to/mysite.com/mysite>
    <Files wsgi.py>
    Require all granted
    </Files>
    </Directory>

If you are using a version of Apache older than 2.4, replace
``Require all granted`` with ``Allow from all`` and also add the line
``Order deny,allow`` above it.

.. _Nginx: http://wiki.nginx.org/Main
.. _Apache: http://httpd.apache.org/

.. More details on configuring a mod_wsgi site to serve static files can be found
.. in the mod_wsgi documentation on `hosting static files`_.

.. _hosting static files: http://code.google.com/p/modwsgi/wiki/ConfigurationGuidelines#Hosting_Of_Static_Files

.. _serving-the-admin-files:

Serving the admin files
=======================

When :mod:`django.contrib.staticfiles` is in ``INSTALLED_APPS``, the
Django development server automatically serves the static files of the
admin app (and any other installed apps). This is however not the case when you
use any other server arrangement. You're responsible for setting up Apache, or
whichever Web server you're using, to serve the admin files.

The admin files live in (:file:`django/contrib/admin/static/admin`) of the
Django distribution.

We **strongly** recommend using :mod:`django.contrib.staticfiles` to handle the
admin files (along with a Web server as outlined in the previous section; this
means using the ``collectstatic`` management command to collect the
static files in ``STATIC_ROOT``, and then configuring your Web server to
serve ``STATIC_ROOT`` at ``STATIC_URL``), but here are three
other approaches:

1. Create a symbolic link to the admin static files from within your
   document root (this may require ``+FollowSymLinks`` in your Apache
   configuration).

2. Use an ``Alias`` directive, as demonstrated above, to alias the appropriate
   URL (probably ``STATIC_URL`` + ``admin/``) to the actual location of
   the admin files.

3. Copy the admin static files so that they live within your Apache
   document root.

Authenticating against Django's user database from Apache
=========================================================

Django provides a handler to allow Apache to authenticate users directly
against Django's authentication backends. See the mod_wsgi authentication
documentation.

If you get a UnicodeEncodeError
===============================

If you're taking advantage of the internationalization features of Django and you intend to allow users to upload files, you must
ensure that the environment used to start Apache is configured to accept
non-ASCII file names. If your environment is not correctly configured, you
will trigger ``UnicodeEncodeError`` exceptions when calling functions like
the ones in :mod:`os.path` on filenames that contain non-ASCII characters.

To avoid these problems, the environment used to start Apache should contain
settings analogous to the following::

    export LANG='en_US.UTF-8'
    export LC_ALL='en_US.UTF-8'

Consult the documentation for your operating system for the appropriate syntax
and location to put these configuration items; ``/etc/apache2/envvars`` is a
common location on Unix platforms. Once you have added these statements
to your environment, restart Apache.

.. _staticfiles-production:

Serving static files in production
==================================

The basic outline of putting static files into production is simple: run the
``collectstatic`` command when static files change, then arrange for
the collected static files directory (``STATIC_ROOT``) to be moved to
the static file server and served. Depending on ``STATICFILES_STORAGE``,
files may need to be moved to a new location manually or the :func:`post_process
<django.contrib.staticfiles.storage.StaticFilesStorage.post_process>` method
of the ``Storage`` class might take care of that.

Of course, as with all deployment tasks, the devil's in the details. Every
production setup will be a bit different, so you'll need to adapt the basic
outline to fit your needs. Below are a few common patterns that might help.

Serving the site and your static files from the same server
-----------------------------------------------------------

If you want to serve your static files from the same server that's already
serving your site, the process may look something like:

* Push your code up to the deployment server.
* On the server, run ``collectstatic`` to copy all the static files
  into ``STATIC_ROOT``.
* Configure your web server to serve the files in ``STATIC_ROOT``
  under the URL ``STATIC_URL``. For example, here's
  how to do this with Apache and mod_wsgi .

You'll probably want to automate this process, especially if you've got
multiple web servers. There's any number of ways to do this automation, but
one option that many Django developers enjoy is `Fabric
<http://fabfile.org/>`_.

Below, and in the following sections, we'll show off a few example fabfiles
(i.e. Fabric scripts) that automate these file deployment options. The syntax
of a fabfile is fairly straightforward but won't be covered here; consult
`Fabric's documentation <http://docs.fabfile.org/>`_, for a complete
explanation of the syntax.

So, a fabfile to deploy static files to a couple of web servers might look
something like::

    from fabric.api import *

    # Hosts to deploy onto
    env.hosts = ['www1.example.com', 'www2.example.com']

    # Where your project code lives on the server
    env.project_root = '/home/www/myproject'

    def deploy_static():
        with cd(env.project_root):
            run('./manage.py collectstatic -v0 --noinput')

Serving static files from a dedicated server
--------------------------------------------

Most larger Django sites use a separate Web server -- i.e., one that's not also
running Django -- for serving static files. This server often runs a different
type of web server -- faster but less full-featured. Some common choices are:

* Nginx_
* A stripped-down version of Apache_

.. _Nginx: http://wiki.nginx.org/Main
.. _Apache: http://httpd.apache.org/

Configuring these servers is out of scope of this document; check each
server's respective documentation for instructions.

Since your static file server won't be running Django, you'll need to modify
the deployment strategy to look something like:

* When your static files change, run ``collectstatic`` locally.

* Push your local ``STATIC_ROOT`` up to the static file server into the
  directory that's being served. `rsync <https://rsync.samba.org/>`_ is a
  common choice for this step since it only needs to transfer the bits of
  static files that have changed.

Here's how this might look in a fabfile::

    from fabric.api import *
    from fabric.contrib import project

    # Where the static files get collected locally. Your STATIC_ROOT setting.
    env.local_static_root = '/tmp/static'

    # Where the static files should go remotely
    env.remote_static_root = '/home/www/static.example.com'

    @roles('static')
    def deploy_static():
        local('./manage.py collectstatic')
        project.rsync_project(
            remote_dir = env.remote_static_root,
            local_dir = env.local_static_root,
            delete = True
        )

.. _staticfiles-from-cdn:

Serving static files from a cloud service or CDN
------------------------------------------------

Another common tactic is to serve static files from a cloud storage provider
like Amazon's S3 and/or a CDN (content delivery network). This lets you
ignore the problems of serving static files and can often make for
faster-loading webpages (especially when using a CDN).

When using these services, the basic workflow would look a bit like the above,
except that instead of using ``rsync`` to transfer your static files to the
server you'd need to transfer the static files to the storage provider or CDN.

There's any number of ways you might do this, but if the provider has an API a
custom file storage backend  will make the
process incredibly simple. If you've written or are using a 3rd party custom
storage backend, you can tell ``collectstatic`` to use it by setting
``STATICFILES_STORAGE`` to the storage engine.

For example, if you've written an S3 storage backend in
``myproject.storage.S3Storage`` you could use it with::

    STATICFILES_STORAGE = 'myproject.storage.S3Storage'

Once that's done, all you have to do is run ``collectstatic`` and your
static files would be pushed through your storage package up to S3. If you
later needed to switch to a different storage provider, it could be as simple
as changing your ``STATICFILES_STORAGE`` setting.

There are 3rd party apps available that
provide storage backends for many common file storage APIs. A good starting
point is the `overview at djangopackages.com
<https://www.djangopackages.com/grids/g/storage-backends/>`_.

Scaling
=======

Now that you know how to get Django running on a single server, let's look at
how you can scale out a Django installation. This section walks through how
a site might scale from a single server to a large-scale cluster that could
serve millions of hits an hour.

It's important to note, however, that nearly every large site is large in
different ways, so scaling is anything but a one-size-fits-all operation. The
following coverage should suffice to show the general principle, and whenever
possible we'll try to point out where different choices could be made.

First off, we'll make a pretty big assumption and exclusively talk about
scaling under Apache and mod_python. Though we know of a number of successful
medium- to large-scale FastCGI deployments, we're much more familiar with
Apache.

Running on a Single Server
--------------------------

Most sites start out running on a single server, with an architecture that
looks something like Figure 13-1.

.. figure:: graphics/chapter_13/scaling-1.png

   Figure 13-1: a single server Django setup.

This works just fine for small- to medium-sized sites, and it's relatively cheap -- you
can put together a single-server site designed for Django for well under $3,000.

However, as traffic increases you'll quickly run into *resource contention*
between the different pieces of software. Database servers and Web servers
*love* to have the entire server to themselves, so when run on the same server
they often end up "fighting" over the same resources (RAM, CPU) that they'd
prefer to monopolize.

This is solved easily by moving the database server to a second machine,
as explained in the following section.

Separating Out the Database Server
----------------------------------

As far as Django is concerned, the process of separating out the database server
is extremely easy: you'll simply need to change the ``DATABASE_HOST``
setting to the IP or DNS name of your database server. It's probably a good idea
to use the IP if at all possible, as relying on DNS for the connection between
your Web server and database server isn't recommended.

With a separate database server, our architecture now looks like Figure 13-2.

.. figure:: graphics/chapter_13/scaling-2.png

   Figure 13-2: Moving the database onto a dedicated server.

Here we're starting to move into what's usually called *n-tier*
architecture. Don't be scared by the buzzword -- it just refers to the fact that
different "tiers" of the Web stack get separated out onto different physical
machines.

At this point, if you anticipate ever needing to grow beyond a single database
server, it's probably a good idea to start thinking about connection pooling
and/or database replication. Unfortunately, there's not nearly enough space to do
those topics justice in this book, so you'll need to consult your database's
documentation and/or community for more information.

Running a Separate Media Server
-------------------------------

We still have a big problem left over from the single-server setup:
the serving of media from the same box that handles dynamic content.

Those two activities perform best under different circumstances, and by smashing
them together on the same box you end up with neither performing particularly
well. So the next step is to separate out the media -- that is, anything *not*
generated by a Django view -- onto a dedicated server (see Figure 13-3).

.. figure:: graphics/chapter_13/scaling-3.png

   Figure 13-3: Separating out the media server.

Ideally, this media server should run a stripped-down Web server optimized for
static media delivery. lighttpd and tux (http://www.djangoproject.com/r/tux/)
are both excellent choices here, but a heavily stripped down Apache could work,
too.

For sites heavy in static content (photos, videos, etc.), moving to a
separate media server is doubly important and should likely be the *first*
step in scaling up.

This step can be slightly tricky, however. If your application involves file
uploads, Django needs to be able to write uploaded media to the media server.
If media lives on another server, you'll need to arrange a way for that write
to happen across the network.

Implementing Load Balancing and Redundancy
------------------------------------------

At this point, we've broken things down as much as possible. This
three-server setup should handle a very large amount of traffic -- we served
around 10 million hits a day from an architecture of this sort -- so if you
grow further, you'll need to start adding redundancy.

This is a good thing, actually. One glance at Figure 13-3 shows you that
if even a single one of your three servers fails, you'll bring down your
entire site. So as you add redundant servers, not only do you increase capacity,
but you also increase reliability.

For the sake of this example, let's assume that the Web server hits capacity
first. It's relatively easy to get multiple copies of a Django site running on
different hardware -- just copy all the code onto multiple machines, and start
Apache on both of them.

However, you'll need another piece of software to distribute traffic over your
multiple servers: a *load balancer*. You can buy expensive and proprietary
hardware load balancers, but there are a few high-quality open source software
load balancers out there.

Apache's ``mod_proxy`` is one option, but we've found Perlbal
(http://www.djangoproject.com/r/perlbal/) to be fantastic. It's a load
balancer and reverse proxy written by the same folks who wrote ``memcached``
(see `Chapter 15`_).

With the Web servers now clustered, our evolving architecture starts to look
more complex, as shown in Figure 13-4.

.. figure:: graphics/chapter_13/scaling-4.png

   Figure 13-4: A load-balanced, redundant server setup.

Notice that in the diagram the Web servers are referred to as a "cluster" to
indicate that the number of servers is basically variable. Once you have a
load balancer out front, you can easily add and remove back-end Web servers
without a second of downtime.

Going Big
---------

At this point, the next few steps are pretty much derivatives of the last one:

* As you need more database performance, you might want to add replicated
  database servers. MySQL includes built-in replication; PostgreSQL
  users should look into Slony (http://www.djangoproject.com/r/slony/)
  and pgpool (http://www.djangoproject.com/r/pgpool/) for replication and
  connection pooling, respectively.

* If the single load balancer isn't enough, you can add more load
  balancer machines out front and distribute among them using
  round-robin DNS.

* If a single media server doesn't suffice, you can add more media
  servers and distribute the load with your load-balancing cluster.

* If you need more cache storage, you can add dedicated cache servers.

* At any stage, if a cluster isn't performing well, you can add more
  servers to the cluster.

After a few of these iterations, a large-scale architecture might look like Figure 13-5.

.. figure:: graphics/chapter_13/scaling-5.png

   Figure 13-5. An example large-scale Django setup.

Though we've shown only two or three servers at each level, there's no
fundamental limit to how many you can add.

Performance Tuning
==================

If you have huge amount of money, you can just keep throwing hardware at
scaling problems. For the rest of us, though, performance tuning is a must.

.. note::

    Incidentally, if anyone with monstrous gobs of cash is actually reading
    this book, please consider a substantial donation to the Django Foundation.
    We accept uncut diamonds and gold ingots, too.

Unfortunately, performance tuning is much more of an art than a science, and it
is even more difficult to write about than scaling. If you're serious about
deploying a large-scale Django application, you should spend a great deal of
time learning how to tune each piece of your stack.

The following sections, though, present a few Django-specific tuning tips we've
discovered over the years.

There's No Such Thing As Too Much RAM
-------------------------------------

Even the really expensive RAM is relatively affordable these days. Buy as much
RAM as you can possibly afford, and then buy a little bit more.

Faster processors won't improve performance all that much; most Web
servers spend up to 90% of their time waiting on disk I/O. As soon as you start
swapping, performance will just die. Faster disks might help slightly, but
they're much more expensive than RAM, such that it doesn't really matter.

If you have multiple servers, the first place to put your RAM is in the
database server. If you can afford it, get enough RAM to get fit your entire
database into memory. This shouldn't be too hard; we've developed a site
with more than half a million newspaper articles, and it took under 2GB of
space.

Next, max out the RAM on your Web server. The ideal situation is one where
neither server swaps -- ever. If you get to that point, you should be able to
withstand most normal traffic.

Turn Off Keep-Alive
-------------------

``Keep-Alive`` is a feature of HTTP that allows multiple HTTP requests to be
served over a single TCP connection, avoiding the TCP setup/teardown overhead.

This looks good at first glance, but it can kill the performance of a Django
site. If you're properly serving media from a separate server, each user
browsing your site will only request a page from your Django server every ten
seconds or so. This leaves HTTP servers waiting around for the next
keep-alive request, and an idle HTTP server just consumes RAM that an active one
should be using.

Use memcached
-------------

Although Django supports a number of different cache back-ends, none of them
even come *close* to being as fast as memcached. If you have a high-traffic
site, don't even bother with the other backends -- go straight to memcached.

Use memcached Often
-------------------

Of course, selecting memcached does you no good if you don't actually use it.
`Chapter 15`_ is your best friend here: learn how to use Django's cache
framework, and use it everywhere possible. Aggressive, preemptive caching is
usually the only thing that will keep a site up under major traffic.

.. _Chapter 15: chapter15.html

Join the Conversation
---------------------

Each piece of the Django stack -- from Linux to Apache to PostgreSQL or MySQL
-- has an awesome community behind it. If you really want to get that last 1%
out of your servers, join the open source communities behind your software and
ask for help. Most free-software community members will be happy to help.

And also be sure to join the Django community. Your humble authors are only two
members of an incredibly active, growing group of Django developers. Our
community has a huge amount of collective experience to offer.

What's Next?
============

The remaining chapters focus on other Django features that you may or may not
need, depending on your application. Feel free to read them in any order you
choose.
