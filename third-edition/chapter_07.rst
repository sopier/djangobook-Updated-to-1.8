======================================
Chapter 7: Advanced Views and URLconfs
======================================

In Chapter 2, we explained the basics of Django view functions and URLconfs.
This chapter goes into more detail about advanced functionality in those two
pieces of the framework.

URLconf Tricks
==============

There's nothing "special" about URLconfs -- like anything else in Django,
they're just Python code. You can take advantage of this in several ways, as
described in the sections that follow.

Streamlining Function Imports
-----------------------------

Consider this URLconf, which builds on the example in Chapter 2::

  from django.conf.urls import include, url
        from django.contrib import admin
        from mysite.views import hello, current_datetime, hours_ahead

        urlpatterns = [
        url(r'^admin/', include(admin.site.urls)),
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
        url(r'^time/plus/(\d{1,2})/$', hours_ahead),
        ]

As explained in Chapter 2, each entry in the URLconf includes its associated
view function, passed directly as a function object. This means it's necessary
to import the view functions at the top of the module.

But as a Django application grows in complexity, its URLconf grows, too, and
keeping those imports can be tedious to manage. (For each new view function,
you have to remember to import it, and the import statement tends to get
overly long if you use this approach.) It's possible to avoid that tedium by
importing the ``views`` module itself. This example URLconf is equivalent to
the previous one:

.. parsed-literal::

    from django.conf.urls import include, url
    **from . import views**

    urlpatterns = [
       url(r'^hello/$', **views.hello**),
       url(r'^time/$', **views.current_datetime**),
       url(r'^time/plus/(\d{1,2})/$', **views.hours_ahead**),
    ]

Special-Casing URLs in Debug Mode
---------------------------------

Speaking of constructing ``urlpatterns`` dynamically, you might want to take
advantage of this technique to alter your URLconf's behavior while in Django's
debug mode. To do this, just check the value of the ``DEBUG`` setting at
runtime, like so::

    from django.conf import settings
    from django.conf.urls import url
    from . import views

    urlpatterns = [
        url(r'^$', views.homepage),
        url(r'^(\d{4})/([a-z]{3})/$', views.archive_month),
    ]

    if settings.DEBUG:
        urlpatterns += [url(r'^debuginfo/$', views.debug),] 

In this example, the URL ``/debuginfo/`` will only be available if your
``DEBUG`` setting is set to ``True``.

Named groups
============

The above example used simple, *non-named* regular-expression groups (via
parenthesis) to capture bits of the URL and pass them as *positional* arguments
to a view. In more advanced usage, it's possible to use *named*
regular-expression groups to capture URL bits and pass them as *keyword*
arguments to a view.

.. admonition:: Keyword Arguments vs. Positional Arguments

    A Python function can be called using keyword arguments or positional
    arguments -- and, in some cases, both at the same time. In a keyword
    argument call, you specify the names of the arguments along with the values
    you're passing. In a positional argument call, you simply pass the
    arguments without explicitly specifying which argument matches which value;
    the association is implicit in the arguments' order.

    For example, consider this simple function::

        def sell(item, price, quantity):
            print "Selling %s unit(s) of %s at %s" % (quantity, item, price)

    To call it with positional arguments, you specify the arguments in the
    order in which they're listed in the function definition::

        sell('Socks', '$2.50', 6)

    To call it with keyword arguments, you specify the names of the arguments
    along with the values. The following statements are equivalent::

        sell(item='Socks', price='$2.50', quantity=6)
        sell(item='Socks', quantity=6, price='$2.50')
        sell(price='$2.50', item='Socks', quantity=6)
        sell(price='$2.50', quantity=6, item='Socks')
        sell(quantity=6, item='Socks', price='$2.50')
        sell(quantity=6, price='$2.50', item='Socks')

    Finally, you can mix keyword and positional arguments, as long as all
    positional arguments are listed before keyword arguments. The following
    statements are equivalent to the previous examples::

        sell('Socks', '$2.50', quantity=6)
        sell('Socks', price='$2.50', quantity=6)
        sell('Socks', quantity=6, price='$2.50')

In Python regular expressions, the syntax for named regular-expression groups
is ``(?P<name>pattern)``, where ``name`` is the name of the group and
``pattern`` is some pattern to match.

Here's a sample URLconf::

    from django.conf.urls import url

    from . import views

    urlpatterns = [
        url(r'^articles/2003/$', views.special_case_2003),
        url(r'^articles/([0-9]{4})/$', views.year_archive),
        url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive),
        url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),
    ]

Notes:

* To capture a value from the URL, just put parenthesis around it.

* There's no need to add a leading slash, because every URL has that. For
  example, it's ``^articles``, not ``^/articles``.

* The ``'r'`` in front of each regular expression string is optional but
  recommended. It tells Python that a string is "raw" -- that nothing in
  the string should be escaped.

Example requests:

* A request to ``/articles/2005/03/`` would match the third entry in the
  list. Django would call the function
  ``views.month_archive(request, '2005', '03')``.

* ``/articles/2005/3/`` would not match any URL patterns, because the
  third entry in the list requires two digits for the month.

* ``/articles/2003/`` would match the first pattern in the list, not the
  second one, because the patterns are tested in order, and the first one
  is the first test to pass. Feel free to exploit the ordering to insert
  special cases like this.

* ``/articles/2003`` would not match any of these patterns, because each
  pattern requires that the URL end with a slash.

* ``/articles/2003/03/03/`` would match the final pattern. Django would call
  the function ``views.article_detail(request, '2003', '03', '03')``.

Here's the above example URLconf, rewritten to use named groups::

    from django.conf.urls import url

    from . import views

    urlpatterns = [
        url(r'^articles/2003/$', views.special_case_2003),
        url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
        url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),
        url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<day>[0-9]{2})/$', views.article_detail),
    ]

This accomplishes exactly the same thing as the previous example, with one
subtle difference: The captured values are passed to view functions as keyword
arguments rather than positional arguments. For example:

* A request to ``/articles/2005/03/`` would call the function
  ``views.month_archive(request, year='2005', month='03')``, instead
  of ``views.month_archive(request, '2005', '03')``.

* A request to ``/articles/2003/03/03/`` would call the function
  ``views.article_detail(request, year='2003', month='03', day='03')``.

In practice, this means your URLconfs are slightly more explicit and less prone
to argument-order bugs -- and you can reorder the arguments in your views'
function definitions. Of course, these benefits come at the cost of brevity;
some developers find the named-group syntax ugly and too verbose.

The matching/grouping algorithm
-------------------------------

Here's the algorithm the URLconf parser follows, with respect to named groups
vs. non-named groups in a regular expression:

1. If there are any named arguments, it will use those, ignoring non-named
   arguments.

2. Otherwise, it will pass all non-named arguments as positional arguments.

In both cases, any extra keyword arguments that have been given will also be passed to the view.

What the URLconf searches against
=================================

The URLconf searches against the requested URL, as a normal Python string. This
does not include GET or POST parameters, or the domain name.

For example, in a request to ``http://www.example.com/myapp/``, the URLconf
will look for ``myapp/``.

In a request to ``http://www.example.com/myapp/?page=3``, the URLconf will look
for ``myapp/``.

The URLconf doesn't look at the request method. In other words, all request
methods -- ``POST``, ``GET``, ``HEAD``, etc. -- will be routed to the same
function for the same URL.

Captured arguments are always strings
=====================================

Each captured argument is sent to the view as a plain Python string, regardless
of what sort of match the regular expression makes. For example, in this
URLconf line::

    url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),

...the ``year`` argument to ``views.year_archive()`` will be a string, not
an integer, even though the ``[0-9]{4}`` will only match integer strings.

Specifying defaults for view arguments
======================================

A convenient trick is to specify default parameters for your views' arguments.
Here's an example URLconf and view::

    # URLconf
    from django.conf.urls import url

    from . import views

    urlpatterns = [
        url(r'^blog/$', views.page),
        url(r'^blog/page(?P<num>[0-9]+)/$', views.page),
    ]

    # View (in blog/views.py)
    def page(request, num="1"):
        # Output the appropriate page of blog entries, according to num.
        ...

In the above example, both URL patterns point to the same view --
``views.page`` -- but the first pattern doesn't capture anything from the
URL. If the first pattern matches, the ``page()`` function will use its
default argument for ``num``, ``"1"``. If the second pattern matches,
``page()`` will use whatever ``num`` value was captured by the regex.

Performance
===========

Each regular expression in a ``urlpatterns`` is compiled the first time it's
accessed. This makes the system blazingly fast.

Error handling
==============

When Django can't find a regex matching the requested URL, or when an
exception is raised, Django will invoke an error-handling view.

The views to use for these cases are specified by four variables. Their
default values should suffice for most projects, but further customization is
possible by assigning values to them.

Such values can be set in your root URLconf. Setting these variables in any
other URLconf will have no effect.

Values must be callables, or strings representing the full Python import path
to the view that should be called to handle the error condition at hand.

The variables are:

* ``handler404`` -- See :data:`django.conf.urls.handler404`.
* ``handler500`` -- See :data:`django.conf.urls.handler500`.
* ``handler403`` -- See :data:`django.conf.urls.handler403`.
* ``handler400`` -- See :data:`django.conf.urls.handler400`.

.. _including-other-urlconfs:

Including other URLconfs
========================

At any point, your ``urlpatterns`` can "include" other URLconf modules. This
essentially "roots" a set of URLs below other ones.

For example, here's an excerpt of the URLconf for the Django Web site
itself. It includes a number of other URLconfs::

    from django.conf.urls import include, url

    urlpatterns = [
        # ... snip ...
        url(r'^community/', include('django_website.aggregator.urls')),
        url(r'^contact/', include('django_website.contact.urls')),
        # ... snip ...
    ]

Note that the regular expressions in this example don't have a ``$``
(end-of-string match character) but do include a trailing slash. Whenever
Django encounters ``include()`` (:func:`django.conf.urls.include()`), it chops
off whatever part of the URL matched up to that point and sends the remaining
string to the included URLconf for further processing.

Another possibility is to include additional URL patterns by using a list of
:func:`~django.conf.urls.url` instances. For example, consider this URLconf::

    from django.conf.urls import include, url

    from apps.main import views as main_views
    from credit import views as credit_views

    extra_patterns = [
        url(r'^reports/(?P<id>[0-9]+)/$', credit_views.report),
        url(r'^charge/$', credit_views.charge),
    ]

    urlpatterns = [
        url(r'^$', main_views.homepage),
        url(r'^help/', include('apps.help.urls')),
        url(r'^credit/', include(extra_patterns)),
    ]

In this example, the ``/credit/reports/`` URL will be handled by the
``credit.views.report()`` Django view.

This can be used to remove redundancy from URLconfs where a single pattern
prefix is used repeatedly. For example, consider this URLconf::

    from django.conf.urls import url
    from . import views

    urlpatterns = [
        url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/history/$', views.history),
        url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/edit/$', views.edit),
        url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/discuss/$', views.discuss),
        url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/permissions/$', views.permissions),
    ]

We can improve this by stating the common path prefix only once and grouping
the suffixes that differ::

    from django.conf.urls import include, url
    from . import views

    urlpatterns = [
        url(r'^(?P<page_slug>\w+)-(?P<page_id>\w+)/', include([
            url(r'^history/$', views.history),
            url(r'^edit/$', views.edit),
            url(r'^discuss/$', views.discuss),
            url(r'^permissions/$', views.permissions),
        ])),
    ]

Captured parameters
-------------------

An included URLconf receives any captured parameters from parent URLconfs, so
the following example is valid::

    # In settings/urls/main.py
    from django.conf.urls import include, url

    urlpatterns = [
        url(r'^(?P<username>\w+)/blog/', include('foo.urls.blog')),
    ]

    # In foo/urls/blog.py
    from django.conf.urls import url
    from . import views

    urlpatterns = [
        url(r'^$', views.blog.index),
        url(r'^archive/$', views.blog.archive),
    ]

In the above example, the captured ``"username"`` variable is passed to the
included URLconf, as expected.

.. _views-extra-options:

Passing extra options to view functions
=======================================

URLconfs have a hook that lets you pass extra arguments to your view functions,
as a Python dictionary.

The :func:`django.conf.urls.url` function can take an optional third argument
which should be a dictionary of extra keyword arguments to pass to the view
function.

For example::

    from django.conf.urls import url
    from . import views

    urlpatterns = [
        url(r'^blog/(?P<year>[0-9]{4})/$', views.year_archive, {'foo': 'bar'}),
    ]

In this example, for a request to ``/blog/2005/``, Django will call
``views.year_archive(request, year='2005', foo='bar')``.

This technique is used in the syndication framework to pass metadata and
options to views (see Chapter 15).

.. admonition:: Dealing with conflicts

    It's possible to have a URL pattern which captures named keyword arguments,
    and also passes arguments with the same names in its dictionary of extra
    arguments. When this happens, the arguments in the dictionary will be used
    instead of the arguments captured in the URL.

Passing extra options to ``include()``
--------------------------------------

Similarly, you can pass extra options to :func:`~django.conf.urls.include`.
When you pass extra options to ``include()``, *each* line in the included
URLconf will be passed the extra options.

For example, these two URLconf sets are functionally identical:

Set one::

    # main.py
    from django.conf.urls import include, url

    urlpatterns = [
        url(r'^blog/', include('inner'), {'blogid': 3}),
    ]

    # inner.py
    from django.conf.urls import url
    from mysite import views

    urlpatterns = [
        url(r'^archive/$', views.archive),
        url(r'^about/$', views.about),
    ]

Set two::

    # main.py
    from django.conf.urls import include, url
    from mysite import views

    urlpatterns = [
        url(r'^blog/', include('inner')),
    ]

    # inner.py
    from django.conf.urls import url

    urlpatterns = [
        url(r'^archive/$', views.archive, {'blogid': 3}),
        url(r'^about/$', views.about, {'blogid': 3}),
    ]

Note that extra options will *always* be passed to *every* line in the included
URLconf, regardless of whether the line's view actually accepts those options
as valid. For this reason, this technique is only useful if you're certain that
every view in the included URLconf accepts the extra options you're passing.

Reverse resolution of URLs
==========================

A common need when working on a Django project is the possibility to obtain URLs
in their final forms either for embedding in generated content (views and assets
URLs, URLs shown to the user, etc.) or for handling of the navigation flow on
the server side (redirections, etc.)

It is strongly desirable not having to hard-code these URLs (a laborious,
non-scalable and error-prone strategy) or having to devise ad-hoc mechanisms for
generating URLs that are parallel to the design described by the URLconf and as
such in danger of producing stale URLs at some point.

In other words, what's needed is a DRY mechanism. Among other advantages it
would allow evolution of the URL design without having to go all over the
project source code to search and replace outdated URLs.

The piece of information we have available as a starting point to get a URL is
an identification (e.g. the name) of the view in charge of handling it, other
pieces of information that necessarily must participate in the lookup of the
right URL are the types (positional, keyword) and values of the view arguments.

Django provides a solution such that the URL mapper is the only repository of
the URL design. You feed it with your URLconf and then it can be used in both
directions:

* Starting with a URL requested by the user/browser, it calls the right Django
  view providing any arguments it might need with their values as extracted from
  the URL.

* Starting with the identification of the corresponding Django view plus the
  values of arguments that would be passed to it, obtain the associated URL.

The first one is the usage we've been discussing in the previous sections. The
second one is what is known as *reverse resolution of URLs*, *reverse URL
matching*, *reverse URL lookup*, or simply *URL reversing*.

Django provides tools for performing URL reversing that match the different
layers where URLs are needed:

* In templates: Using the ``url`` template tag.

* In Python code: Using the :func:`django.core.urlresolvers.reverse`
  function.

* In higher level code related to handling of URLs of Django model instances:
  The :meth:`~django.db.models.Model.get_absolute_url` method.

Examples
--------

Consider again this URLconf entry::

    from django.conf.urls import url

    from . import views

    urlpatterns = [
        #...
        url(r'^articles/([0-9]{4})/$', views.year_archive, name='news-year-archive'),
        #...
    ]

According to this design, the URL for the archive corresponding to year *nnnn*
is ``/articles/nnnn/``.

You can obtain these in template code by using:

.. code-block:: html+django

    <a href="{% url 'news-year-archive' 2012 %}">2012 Archive</a>
    {# Or with the year in a template context variable: #}
    <ul>
    {% for yearvar in year_list %}
    <li><a href="{% url 'news-year-archive' yearvar %}">{{ yearvar }} Archive</a></li>
    {% endfor %}
    </ul>

Or in Python code::

    from django.core.urlresolvers import reverse
    from django.http import HttpResponseRedirect

    def redirect_to_year(request):
        # ...
        year = 2006
        # ...
        return HttpResponseRedirect(reverse('news-year-archive', args=(year,)))

If, for some reason, it was decided that the URLs where content for yearly
article archives are published at should be changed then you would only need to
change the entry in the URLconf.

In some scenarios where views are of a generic nature, a many-to-one
relationship might exist between URLs and views. For these cases the view name
isn't a good enough identifier for it when comes the time of reversing
URLs. Read the next section to know about the solution Django provides for this.

.. _naming-url-patterns:

Naming URL patterns
===================

In order to perform URL reversing, you'll need to use **named URL patterns**
as done in the examples above. The string used for the URL name can contain any
characters you like. You are not restricted to valid Python names.

When you name your URL patterns, make sure you use names that are unlikely
to clash with any other application's choice of names. If you call your URL
pattern ``comment``, and another application does the same thing, there's
no guarantee which URL will be inserted into your template when you use
this name.

Putting a prefix on your URL names, perhaps derived from the application
name, will decrease the chances of collision. We recommend something like
``myapp-comment`` instead of ``comment``.

.. _topics-http-defining-url-namespaces:

URL namespaces
==============

Introduction
------------

URL namespaces allow you to uniquely reverse named URL patterns even if different applications use the same URL names.
It's a good practice for third-party apps to always use namespaced URLs (as we
did in the tutorial). Similarly, it also allows you to reverse URLs if multiple
instances of an application are deployed. In other words, since multiple
instances of a single application will share named URLs, namespaces provide a
way to tell these named URLs apart.

Django applications that make proper use of URL namespacing can be deployed more
than once for a particular site. For example :mod:`django.contrib.admin` has an
:class:`~django.contrib.admin.AdminSite` class which allows you to easily
deploy more than once instance of the admin`.

A URL namespace comes in two parts, both of which are strings:

.. glossary::

  application namespace
    This describes the name of the application that is being deployed. Every
    instance of a single application will have the same application namespace.
    For example, Django's admin application has the somewhat predictable
    application namespace of ``'admin'``.

  instance namespace
    This identifies a specific instance of an application. Instance namespaces
    should be unique across your entire project. However, an instance namespace
    can be the same as the application namespace. This is used to specify a
    default instance of an application. For example, the default Django admin
    instance has an instance namespace of ``'admin'``.

Namespaced URLs are specified using the ``':'`` operator. For example, the main
index page of the admin application is referenced using ``'admin:index'``. This
indicates a namespace of ``'admin'``, and a named URL of ``'index'``.

Namespaces can also be nested. The named URL ``'sports:polls:index'`` would
look for a pattern named ``'index'`` in the namespace ``'polls'`` that is itself
defined within the top-level namespace ``'sports'``.

.. _topics-http-reversing-url-namespaces:

Reversing namespaced URLs
-------------------------

When given a namespaced URL (e.g. ``'polls:index'``) to resolve, Django splits
the fully qualified name into parts and then tries the following lookup:

1. First, Django looks for a matching :term:`application namespace` (in this
   example, ``'polls'``). This will yield a list of instances of that
   application.

2. If there is a *current* application defined, Django finds and returns
   the URL resolver for that instance. The *current* application can be
   specified as an attribute on the request. Applications that expect to
   have multiple deployments should set the ``current_app`` attribute on
   the ``request`` being processed.

   The current application can also be specified manually as an argument
   to the :func:`~django.core.urlresolvers.reverse` function.

3. If there is no current application. Django looks for a default
   application instance. The default application instance is the instance
   that has an :term:`instance namespace` matching the :term:`application
   namespace` (in this example, an instance of ``polls`` called ``'polls'``).

4. If there is no default application instance, Django will pick the last
   deployed instance of the application, whatever its instance name may be.

5. If the provided namespace doesn't match an :term:`application namespace` in
   step 1, Django will attempt a direct lookup of the namespace as an
   :term:`instance namespace`.

If there are nested namespaces, these steps are repeated for each part of the
namespace until only the view name is unresolved. The view name will then be
resolved into a URL in the namespace that has been found.

.. _namespaces-and-include:

URL namespaces and included URLconfs
------------------------------------

URL namespaces of included URLconfs can be specified in two ways.

Firstly, you can provide the :term:`application <application namespace>` and
:term:`instance <instance namespace>` namespaces as arguments to
:func:`~django.conf.urls.include()` when you construct your URL patterns. For
example,::

    url(r'^polls/', include('polls.urls', namespace='author-polls', app_name='polls')),

This will include the URLs defined in ``polls.urls`` into the
:term:`application namespace` ``'polls'``, with the :term:`instance namespace`
``'author-polls'``.

Secondly, you can include an object that contains embedded namespace data. If
you ``include()`` a list of :func:`~django.conf.urls.url` instances,
the URLs contained in that object will be added to the global namespace.
However, you can also ``include()`` a 3-tuple containing::

    (<list of url() instances>, <application namespace>, <instance namespace>)

For example::

    from django.conf.urls import include, url

    from . import views

    polls_patterns = [
        url(r'^$', views.IndexView.as_view(), name='index'),
        url(r'^(?P<pk>\d+)/$', views.DetailView.as_view(), name='detail'),
    ]

    url(r'^polls/', include((polls_patterns, 'polls', 'author-polls'))),

This will include the nominated URL patterns into the given application and
instance namespace.

For example, the Django admin is deployed as instances of
:class:`~django.contrib.admin.AdminSite`.  ``AdminSite`` objects have a ``urls``
attribute: A 3-tuple that contains all the patterns in the corresponding admin
site, plus the application namespace ``'admin'``, and the name of the admin
instance. It is this ``urls`` attribute that you ``include()`` into your
projects ``urlpatterns`` when you deploy an admin instance.

Be sure to pass a tuple to ``include()``. If you simply pass three arguments:
``include(polls_patterns, 'polls', 'author-polls')``, Django won't throw an
error but due to the signature of ``include()``, ``'polls'`` will be the
instance namespace and ``'author-polls'`` will be the application namespace
instead of vice versa.


What's Next?
============

This chapter has provided many advanced tips and tricks for views and URLconfs.
Next, in Chapter 8, we'll give this advanced treatment to Django's template
system.
