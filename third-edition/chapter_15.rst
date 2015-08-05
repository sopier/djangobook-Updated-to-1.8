=======================================
Chapter 15: Generating Non-HTML Content
=======================================

Usually when we talk about developing Web sites, we're talking about producing
HTML. Of course, there's a lot more to the Web than HTML; we use the Web
to distribute data in all sorts of formats: RSS, PDFs, images, and so forth.

So far, we've focused on the common case of HTML production, but in this chapter
we'll take a detour and look at using Django to produce other types of content.

Django has convenient built-in tools that you can use to produce some common
non-HTML content:

* Comma-delimited (CSV) files for importing into spreadsheet applications.

* PDF files.

*  RSS/Atom syndication feeds

* Sitemaps (an XML format originally developed by Google that gives hints to
  search engines)

We'll examine each of those tools a little later, but first we'll cover the
basic principles.

The basics: views and MIME types
================================

Recall from Chapter 2 that a view function is simply a Python function that
takes a Web request and returns a Web response. This response can be the HTML
contents of a Web page, or a redirect, or a 404 error, or an XML document,
or an image...or anything, really.

More formally, a Django view function *must*

* Accept an ``HttpRequest`` instance as its first argument

* Return an ``HttpResponse`` instance

The key to returning non-HTML content from a view lies in the ``HttpResponse``
class, specifically the ``content_type`` argument. By default, Django sets
``content_type`` to **'text/html'**. You can however, set ``content_type`` to
any of the official Internet media types (MIME types) managed by IANA_.

By tweaking the MIME type, we can indicate to the browser that we've returned
a response of a different format.

.. _IANA: http://www.iana.org/assignments/media-types/media-types.xhtml

For example, let's look at a view that returns a PNG image. To
keep things simple, we'll just read the file off the disk::

    from django.http import HttpResponse

    def my_image(request):
        image_data = open("/path/to/my/image.png", "rb").read()
        return HttpResponse(image_data, content_type="image/png")

That's it! If you replace the image path in the ``open()`` call with a path to
a real image, you can use this very simple view to serve an image, and the
browser will display it correctly.

The other important thing to keep in mind is that ``HttpResponse`` objects
implement Python's standard "file-like object" API. This means that you can use
an ``HttpResponse`` instance in any place Python (or a third-party library)
expects a file.

For an example of how that works, let's take a look at producing CSV with
Django.

Producing CSV
=============
Python comes with a CSV library, :mod:`csv`. The key to using it with Django is
that the :mod:`csv` module's CSV-creation capability acts on file-like objects,
and Django's :class:`~django.http.HttpResponse` objects are file-like objects.

Here's an example::

    import csv
    from django.http import HttpResponse

    def some_view(request):
        # Create the HttpResponse object with the appropriate CSV header.
        response = HttpResponse(content_type='text/csv')
        response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'

        writer = csv.writer(response)
        writer.writerow(['First row', 'Foo', 'Bar', 'Baz'])
        writer.writerow(['Second row', 'A', 'B', 'C', '"Testing"', "Here's a quote"])

        return response

The code and comments should be self-explanatory, but a few things deserve a
mention:

* The response gets a special MIME type, ``text/csv``. This tells
  browsers that the document is a CSV file, rather than an HTML file. If
  you leave this off, browsers will probably interpret the output as HTML,
  which will result in ugly, scary gobbledygook in the browser window.

* The response gets an additional ``Content-Disposition`` header, which
  contains the name of the CSV file. This filename is arbitrary; call it
  whatever you want. It'll be used by browsers in the "Save as..."
  dialogue, etc.

* Hooking into the CSV-generation API is easy: Just pass ``response`` as the
  first argument to ``csv.writer``. The ``csv.writer`` function expects a
  file-like object, and :class:`~django.http.HttpResponse` objects fit the
  bill.

* For each row in your CSV file, call ``writer.writerow``, passing it an
  iterable object such as a list or tuple.

* The CSV module takes care of quoting for you, so you don't have to worry
  about escaping strings with quotes or commas in them. Just pass
  ``writerow()`` your raw strings, and it'll do the right thing.

.. _streaming-csv-files:

Streaming large CSV files
-------------------------

When dealing with views that generate very large responses, you might want to
consider using Django's :class:`~django.http.StreamingHttpResponse` instead.
For example, by streaming a file that takes a long time to generate you can
avoid a load balancer dropping a connection that might have otherwise timed out
while the server was generating the response.

In this example, we make full use of Python generators to efficiently handle
the assembly and transmission of a large CSV file::

    import csv

    from django.utils.six.moves import range
    from django.http import StreamingHttpResponse

    class Echo(object):
        """An object that implements just the write method of the file-like
        interface.
        """
        def write(self, value):
            """Write the value by returning it, instead of storing in a buffer."""
            return value

    def some_streaming_csv_view(request):
        """A view that streams a large CSV file."""
        # Generate a sequence of rows. The range is based on the maximum number of
        # rows that can be handled by a single sheet in most spreadsheet
        # applications.
        rows = (["Row {}".format(idx), str(idx)] for idx in range(65536))
        pseudo_buffer = Echo()
        writer = csv.writer(pseudo_buffer)
        response = StreamingHttpResponse((writer.writerow(row) for row in rows),
                                         content_type="text/csv")
        response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'
        return response

Using the template system
=========================

Alternatively, you can use the Django template system 
to generate CSV. This is lower-level than using the convenient Python :mod:`csv`
module, but the solution is presented here for completeness.

The idea here is to pass a list of items to your template, and have the
template output the commas in a ``for`` loop.

Here's an example, which generates the same CSV file as above::

    from django.http import HttpResponse
    from django.template import loader, Context

    def some_view(request):
        # Create the HttpResponse object with the appropriate CSV header.
        response = HttpResponse(content_type='text/csv')
        response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'

        # The data is hard-coded here, but you could load it from a database or
        # some other source.
        csv_data = (
            ('First row', 'Foo', 'Bar', 'Baz'),
            ('Second row', 'A', 'B', 'C', '"Testing"', "Here's a quote"),
        )

        t = loader.get_template('my_template_name.txt')
        c = Context({
            'data': csv_data,
        })
        response.write(t.render(c))
        return response

The only difference between this example and the previous example is that this
one uses template loading instead of the CSV module. The rest of the code --
such as the ``content_type='text/csv'`` -- is the same.

Then, create the template ``my_template_name.txt``, with this template code:

.. code-block:: html+django

    {% for row in data %}
		"{{ row.0|addslashes }}", 
		"{{ row.1|addslashes }}", 
		"{{ row.2|addslashes }}", 
		"{{ row.3|addslashes }}", 
		"{{ row.4|addslashes }}"
    {% endfor %}

This template is quite basic. It just iterates over the given data and displays
a line of CSV for each row. It uses the ``addslashes`` template filter to
ensure there aren't any problems with quotes.

Other text-based formats
========================

Notice that there isn't very much specific to CSV here -- just the specific
output format. You can use either of these techniques to output any text-based
format you can dream of. You can also use a similar technique to generate
arbitrary binary data; For example, generating PDFs.

Generating PDFs
===============

Django is able to output PDF files dynamically using views. This is made
possible by the excellent, open-source ReportLab_ Python PDF library.

The advantage of generating PDF files dynamically is that you can create
customized PDFs for different purposes -- say, for different users or
different pieces of content.

.. _ReportLab: http://www.reportlab.com/opensource/

Install ReportLab
=================

The ReportLab library is `available on PyPI`_. A `user guide`_ (not
coincidentally, a PDF file) is also available for download.
You can install ReportLab with ``pip``:

.. code-block:: bash

    $ pip install reportlab

Test your installation by importing it in the Python interactive interpreter::

    >>> import reportlab

If that command doesn't raise any errors, the installation worked.

.. _available on PyPI: https://pypi.python.org/pypi/reportlab
.. _user guide: http://www.reportlab.com/docs/reportlab-userguide.pdf

Write your view
===============

The key to generating PDFs dynamically with Django is that the ReportLab API
acts on file-like objects, and Django's :class:`~django.http.HttpResponse`
objects are file-like objects.

Here's a "Hello World" example::

    from reportlab.pdfgen import canvas
    from django.http import HttpResponse

    def some_view(request):
        # Create the HttpResponse object with the appropriate PDF headers.
        response = HttpResponse(content_type='application/pdf')
        response['Content-Disposition'] = 'attachment; filename="somefilename.pdf"'

        # Create the PDF object, using the response object as its "file."
        p = canvas.Canvas(response)

        # Draw things on the PDF. Here's where the PDF generation happens.
        # See the ReportLab documentation for the full list of functionality.
        p.drawString(100, 100, "Hello world.")

        # Close the PDF object cleanly, and we're done.
        p.showPage()
        p.save()
        return response

The code and comments should be self-explanatory, but a few things deserve a
mention:

* The response gets a special MIME type, ``application/pdf``. This
  tells browsers that the document is a PDF file, rather than an HTML file.
  If you leave this off, browsers will probably interpret the output as
  HTML, which would result in ugly, scary gobbledygook in the browser
  window.

* The response gets an additional ``Content-Disposition`` header, which
  contains the name of the PDF file. This filename is arbitrary: Call it
  whatever you want. It'll be used by browsers in the "Save as..."
  dialogue, etc.

* The ``Content-Disposition`` header starts with ``'attachment; '`` in this
  example. This forces Web browsers to pop-up a dialog box
  prompting/confirming how to handle the document even if a default is set
  on the machine. If you leave off ``'attachment;'``, browsers will handle
  the PDF using whatever program/plugin they've been configured to use for
  PDFs. Here's what that code would look like::

      response['Content-Disposition'] = 'filename="somefilename.pdf"'

* Hooking into the ReportLab API is easy: Just pass ``response`` as the
  first argument to ``canvas.Canvas``. The ``Canvas`` class expects a
  file-like object, and :class:`~django.http.HttpResponse` objects fit the
  bill.

* Note that all subsequent PDF-generation methods are called on the PDF
  object (in this case, ``p``) -- not on ``response``.

* Finally, it's important to call ``showPage()`` and ``save()`` on the PDF
  file.

.. note::

    ReportLab is not thread-safe. Some of our users have reported odd issues
    with building PDF-generating Django views that are accessed by many people
    at the same time.

Complex PDFs
============

If you're creating a complex PDF document with ReportLab, consider using the
:mod:`io` library as a temporary holding place for your PDF file. This
library provides a file-like object interface that is particularly efficient.
Here's the above "Hello World" example rewritten to use :mod:`io`::

    from io import BytesIO
    from reportlab.pdfgen import canvas
    from django.http import HttpResponse

    def some_view(request):
        # Create the HttpResponse object with the appropriate PDF headers.
        response = HttpResponse(content_type='application/pdf')
        response['Content-Disposition'] = 'attachment; filename="somefilename.pdf"'

        buffer = BytesIO()

        # Create the PDF object, using the BytesIO object as its "file."
        p = canvas.Canvas(buffer)

        # Draw things on the PDF. Here's where the PDF generation happens.
        # See the ReportLab documentation for the full list of functionality.
        p.drawString(100, 100, "Hello world.")

        # Close the PDF object cleanly.
        p.showPage()
        p.save()

        # Get the value of the BytesIO buffer and write it to the response.
        pdf = buffer.getvalue()
        buffer.close()
        response.write(pdf)
        return response

Further resources
=================

* PDFlib_ is another PDF-generation library that has Python bindings. To
  use it with Django, just use the same concepts explained in this article.
* `Pisa XHTML2PDF`_ is yet another PDF-generation library. Pisa ships with
  an example of how to integrate Pisa with Django.
* HTMLdoc_ is a command-line script that can convert HTML to PDF. It
  doesn't have a Python interface, but you can escape out to the shell
  using ``system`` or ``popen`` and retrieve the output in Python.

.. _PDFlib: http://www.pdflib.org/
.. _`Pisa XHTML2PDF`: http://www.xhtml2pdf.com/
.. _HTMLdoc: http://www.htmldoc.org/

Other Possibilities
===================

There's a whole host of other types of content you can generate in Python.
Here are a few more ideas and some pointers to libraries you could use to
implement them:

* *ZIP files*: Python's standard library ships with the
  ``zipfile`` module, which can both read and write compressed ZIP files.
  You could use it to provide on-demand archives of a bunch of files, or
  perhaps compress large documents when requested. You could similarly
  produce TAR files using the standard library's ``tarfile`` module.

* *Dynamic images*: The Python Imaging Library
  (PIL; http://www.pythonware.com/products/pil/) is a fantastic toolkit for
  producing images (PNG, JPEG, GIF, and a whole lot more). You could use
  it to automatically scale down images into thumbnails, composite
  multiple images into a single frame, or even do Web-based image
  processing.

* *Plots and charts*: There are a number of powerful Python plotting and
  charting libraries you could use to produce on-demand maps, charts,
  plots, and graphs. We can't possibly list them all, so here are
  a couple of the highlights:

* ``matplotlib`` (http://matplotlib.sourceforge.net/) can be
  used to produce the type of high-quality plots usually generated
  with MatLab or Mathematica.

* ``pygraphviz`` (http://networkx.lanl.gov/pygraphviz/), an
  interface to the Graphviz graph layout toolkit
  (http://graphviz.org/), can be used for generating structured diagrams of
  graphs and networks.

In general, any Python library capable of writing to a file can be hooked into
Django. The possibilities are immense.

Now that we've looked at the basics of generating non-HTML content, let's step
up a level of abstraction. Django ships with some pretty nifty built-in tools
for generating some common types of non-HTML content.

The Syndication Feed Framework
==============================

Django comes with a high-level syndication-feed-generating framework that
makes creating RSS and Atom feeds easy.

.. admonition:: What's RSS? What's Atom?

    RSS and Atom are both XML-based formats you can use to provide
    automatically updating "feeds" of your site's content. Read more about RSS
    at http://www.whatisrss.com/, and get information on Atom at
    http://www.atomenabled.org/.

To create any syndication feed, all you have to do is write a short Python
class. You can create as many feeds as you want.

Django also comes with a lower-level feed-generating API. Use this if
you want to generate feeds outside of a Web context, or in some other
lower-level way.

.. _RSS: http://www.whatisrss.com/
.. _Atom: http://tools.ietf.org/html/rfc4287

The high-level framework
========================

Overview
--------

The high-level feed-generating framework is supplied by the
:class:`~django.contrib.syndication.views.Feed` class. To create a
feed, write a :class:`~django.contrib.syndication.views.Feed` class
and point to an instance of it in your URLconf.

Feed classes
------------

A :class:`~django.contrib.syndication.views.Feed` class is a Python
class that represents a syndication feed. A feed can be simple (e.g.,
a "site news" feed, or a basic feed displaying the latest entries of a
blog) or more complex (e.g., a feed displaying all the blog entries in
a particular category, where the category is variable).

Feed classes subclass :class:`django.contrib.syndication.views.Feed`.
They can live anywhere in your codebase.

Instances of :class:`~django.contrib.syndication.views.Feed` classes
are views which can be used in your URLconf .

A simple example
----------------

This simple example, taken from a hypothetical police beat news site describes
a feed of the latest five news items::

    from django.contrib.syndication.views import Feed
    from django.core.urlresolvers import reverse
    from policebeat.models import NewsItem

    class LatestEntriesFeed(Feed):
        title = "Police beat site news"
        link = "/sitenews/"
        description = "Updates on changes and additions to police beat central."

        def items(self):
            return NewsItem.objects.order_by('-pub_date')[:5]

        def item_title(self, item):
            return item.title

        def item_description(self, item):
            return item.description

        # item_link is only needed if NewsItem has no get_absolute_url method.
        def item_link(self, item):
            return reverse('news-item', args=[item.pk])

To connect a URL to this feed, put an instance of the Feed object in
your URLconf . For example::

    from django.conf.urls import url
    from myproject.feeds import LatestEntriesFeed

    urlpatterns = [
        # ...
        url(r'^latest/feed/$', LatestEntriesFeed()),
        # ...
    ]

Note:

* The Feed class subclasses :class:`django.contrib.syndication.views.Feed`.

* ``title``, ``link`` and ``description`` correspond to the
  standard RSS ``<title>``, ``<link>`` and ``<description>`` elements,
  respectively.

* ``items()`` is, simply, a method that returns a list of objects that
  should be included in the feed as ``<item>`` elements. Although this
  example returns ``NewsItem`` objects using Django's
  object-relational mapper 
  doesn't have to return model instances. Although you get a few bits of
  functionality "for free" by using Django models, ``items()`` can
  return any type of object you want.

* If you're creating an Atom feed, rather than an RSS feed, set the
  ``subtitle`` attribute instead of the ``description`` attribute.
  See `Publishing Atom and RSS feeds in tandem`_, later, for an example.

One thing is left to do. In an RSS feed, each ``<item>`` has a ``<title>``,
``<link>`` and ``<description>``. We need to tell the framework what data to put
into those elements.

* For the contents of ``<title>`` and ``<description>``, Django tries
  calling the methods ``item_title()`` and ``item_description()`` on
  the :class:`~django.contrib.syndication.views.Feed` class. They are passed
  a single parameter, ``item``, which is the object itself. These are
  optional; by default, the unicode representation of the object is used for
  both.

  If you want to do any special formatting for either the title or
  description, Django templates  can be used
  instead. Their paths can be specified with the ``title_template`` and
  ``description_template`` attributes on the
  :class:`~django.contrib.syndication.views.Feed` class. The templates are
  rendered for each item and are passed two template context variables:

  * ``{{ obj }}`` -- The current object (one of whichever objects you
    returned in ``items()``).

  * ``{{ site }}`` -- A :class:`django.contrib.sites.models.Site` object
    representing the current site. This is useful for ``{{ site.domain
    }}`` or ``{{ site.name }}``. If you do *not* have the Django sites
    framework installed, this will be set to a
    :class:`~django.contrib.sites.requests.RequestSite` object. See the
    RequestSite section of the sites framework documentation
    for more.

  See `a complex example`_ below that uses a description template.

  .. method:: Feed.get_context_data(\*\*kwargs)

      There is also a way to pass additional information to title and description
      templates, if you need to supply more than the two variables mentioned
      before. You can provide your implementation of ``get_context_data`` method
      in your ``Feed`` subclass. For example::

        from mysite.models import Article
        from django.contrib.syndication.views import Feed

        class ArticlesFeed(Feed):
            title = "My articles"
            description_template = "feeds/articles.html"

            def items(self):
                return Article.objects.order_by('-pub_date')[:5]

            def get_context_data(self, **kwargs):
                context = super(ArticlesFeed, self).get_context_data(**kwargs)
                context['foo'] = 'bar'
                return context

  And the template:

  .. code-block:: html+django

    Something about {{ foo }}: {{ obj.description }}

  This method will be called once per each item in the list returned by
  ``items()`` with the following keyword arguments:

  * ``item``: the current item. For backward compatibility reasons, the name
    of this context variable is ``{{ obj }}``.

  * ``obj``: the object returned by ``get_object()``. By default this is not
    exposed to the templates to avoid confusion with ``{{ obj }}`` (see above),
    but you can use it in your implementation of ``get_context_data()``.

  * ``site``: current site as described above.

  * ``request``: current request.

  The behavior of ``get_context_data()`` mimics that of
  generic views <adding-extra-context> - you're supposed to call
  ``super()`` to retrieve context data from parent class, add your data
  and return the modified dictionary.

* To specify the contents of ``<link>``, you have two options. For each item
  in ``items()``, Django first tries calling the
  ``item_link()`` method on the
  :class:`~django.contrib.syndication.views.Feed` class. In a similar way to
  the title and description, it is passed it a single parameter,
  ``item``. If that method doesn't exist, Django tries executing a
  ``get_absolute_url()`` method on that object. Both
  ``get_absolute_url()`` and ``item_link()`` should return the
  item's URL as a normal Python string. As with ``get_absolute_url()``, the
  result of ``item_link()`` will be included directly in the URL, so you
  are responsible for doing all necessary URL quoting and conversion to
  ASCII inside the method itself.

A complex example
-----------------

The framework also supports more complex feeds, via arguments.

For example, a website could offer an RSS feed of recent crimes for every
police beat in a city. It'd be silly to create a separate
:class:`~django.contrib.syndication.views.Feed` class for each police beat; that
would violate the DRY principle and would couple data to
programming logic. Instead, the syndication framework lets you access the
arguments passed from your URLconf  so feeds can output
items based on information in the feed's URL.

The police beat feeds could be accessible via URLs like this:

* :file:`/beats/613/rss/` -- Returns recent crimes for beat 613.
* :file:`/beats/1424/rss/` -- Returns recent crimes for beat 1424.

These can be matched with a URLconf  line such as::

    url(r'^beats/(?P<beat_id>[0-9]+)/rss/$', BeatFeed()),

Like a view, the arguments in the URL are passed to the ``get_object()``
method along with the request object.

Here's the code for these beat-specific feeds::

    from django.contrib.syndication.views import FeedDoesNotExist
    from django.shortcuts import get_object_or_404

    class BeatFeed(Feed):
        description_template = 'feeds/beat_description.html'

        def get_object(self, request, beat_id):
            return get_object_or_404(Beat, pk=beat_id)

        def title(self, obj):
            return "Police beat central: Crimes for beat %s" % obj.beat

        def link(self, obj):
            return obj.get_absolute_url()

        def description(self, obj):
            return "Crimes recently reported in police beat %s" % obj.beat

        def items(self, obj):
            return Crime.objects.filter(beat=obj).order_by('-crime_date')[:30]

To generate the feed's ``<title>``, ``<link>`` and ``<description>``, Django
uses the ``title()``, ``link()`` and ``description()`` methods. In
the previous example, they were simple string class attributes, but this example
illustrates that they can be either strings *or* methods. For each of
``title``, ``link`` and ``description``, Django follows this
algorithm:

* First, it tries to call a method, passing the ``obj`` argument, where
  ``obj`` is the object returned by ``get_object()``.

* Failing that, it tries to call a method with no arguments.

* Failing that, it uses the class attribute.

Also note that ``items()`` also follows the same algorithm -- first, it
tries ``items(obj)``, then ``items()``, then finally an ``items``
class attribute (which should be a list).

We are using a template for the item descriptions. It can be very simple:

.. code-block:: html+django

    {{ obj.description }}

However, you are free to add formatting as desired.

The ``ExampleFeed`` class below gives full documentation on methods and
attributes of :class:`~django.contrib.syndication.views.Feed` classes.

Specifying the type of feed
---------------------------

By default, feeds produced in this framework use RSS 2.0.

To change that, add a ``feed_type`` attribute to your
:class:`~django.contrib.syndication.views.Feed` class, like so::

    from django.utils.feedgenerator import Atom1Feed

    class MyFeed(Feed):
        feed_type = Atom1Feed

Note that you set ``feed_type`` to a class object, not an instance.

Currently available feed types are:

* :class:`django.utils.feedgenerator.Rss201rev2Feed` (RSS 2.01. Default.)
* :class:`django.utils.feedgenerator.RssUserland091Feed` (RSS 0.91.)
* :class:`django.utils.feedgenerator.Atom1Feed` (Atom 1.0.)

Enclosures
----------

To specify enclosures, such as those used in creating podcast feeds, use the
``item_enclosure_url``, ``item_enclosure_length`` and
``item_enclosure_mime_type`` hooks. See the ``ExampleFeed`` class below for
usage examples.

Language
--------

Feeds created by the syndication framework automatically include the
appropriate ``<language>`` tag (RSS 2.0) or ``xml:lang`` attribute (Atom). This
comes directly from your ``LANGUAGE_CODE`` setting.

URLs
----

The ``link`` method/attribute can return either an absolute path (e.g.
:file:`"/blog/"`) or a URL with the fully-qualified domain and protocol (e.g.
``"http://www.example.com/blog/"``). If ``link`` doesn't return the domain,
the syndication framework will insert the domain of the current site, according
to your ``SITE_ID setting <SITE_ID>``.

Atom feeds require a ``<link rel="self">`` that defines the feed's current
location. The syndication framework populates this automatically, using the
domain of the current site according to the ``SITE_ID`` setting.

Publishing Atom and RSS feeds in tandem
---------------------------------------

Some developers like to make available both Atom *and* RSS versions of their
feeds. That's easy to do with Django: Just create a subclass of your
:class:`~django.contrib.syndication.views.Feed`
class and set the ``feed_type`` to something different. Then update your
URLconf to add the extra versions.

Here's a full example::

    from django.contrib.syndication.views import Feed
    from policebeat.models import NewsItem
    from django.utils.feedgenerator import Atom1Feed

    class RssSiteNewsFeed(Feed):
        title = "Police beat site news"
        link = "/sitenews/"
        description = "Updates on changes and additions to police beat central."

        def items(self):
            return NewsItem.objects.order_by('-pub_date')[:5]

    class AtomSiteNewsFeed(RssSiteNewsFeed):
        feed_type = Atom1Feed
        subtitle = RssSiteNewsFeed.description

.. Note::
    In this example, the RSS feed uses a ``description`` while the Atom
    feed uses a ``subtitle``. That's because Atom feeds don't provide for
    a feed-level "description," but they *do* provide for a "subtitle."

    If you provide a ``description`` in your
    :class:`~django.contrib.syndication.views.Feed` class, Django will *not*
    automatically put that into the ``subtitle`` element, because a
    subtitle and description are not necessarily the same thing. Instead, you
    should define a ``subtitle`` attribute.

    In the above example, we simply set the Atom feed's ``subtitle`` to the
    RSS feed's ``description``, because it's quite short already.

And the accompanying URLconf::

    from django.conf.urls import url
    from myproject.feeds import RssSiteNewsFeed, AtomSiteNewsFeed

    urlpatterns = [
        # ...
        url(r'^sitenews/rss/$', RssSiteNewsFeed()),
        url(r'^sitenews/atom/$', AtomSiteNewsFeed()),
        # ...
    ]

Feed class reference
--------------------

.. class:: views.Feed

This example illustrates all possible attributes and methods for a
:class:`~django.contrib.syndication.views.Feed` class::

    from django.contrib.syndication.views import Feed
    from django.utils import feedgenerator

    class ExampleFeed(Feed):

        # FEED TYPE -- Optional. This should be a class that subclasses
        # django.utils.feedgenerator.SyndicationFeed. This designates
        # which type of feed this should be: RSS 2.0, Atom 1.0, etc. If
        # you don't specify feed_type, your feed will be RSS 2.0. This
        # should be a class, not an instance of the class.

        feed_type = feedgenerator.Rss201rev2Feed

        # TEMPLATE NAMES -- Optional. These should be strings
        # representing names of Django templates that the system should
        # use in rendering the title and description of your feed items.
        # Both are optional. If a template is not specified, the
        # item_title() or item_description() methods are used instead.

        title_template = None
        description_template = None

        # TITLE -- One of the following three is required. The framework
        # looks for them in this order.

        def title(self, obj):
            """
            Takes the object returned by get_object() and returns the
            feed's title as a normal Python string.
            """

        def title(self):
            """
            Returns the feed's title as a normal Python string.
            """

        title = 'foo' # Hard-coded title.

        # LINK -- One of the following three is required. The framework
        # looks for them in this order.

        def link(self, obj):
            """
            # Takes the object returned by get_object() and returns the URL
            # of the HTML version of the feed as a normal Python string.
            """

        def link(self):
            """
            Returns the URL of the HTML version of the feed as a normal Python
            string.
            """

        link = '/blog/' # Hard-coded URL.

        # FEED_URL -- One of the following three is optional. The framework
        # looks for them in this order.

        def feed_url(self, obj):
            """
            # Takes the object returned by get_object() and returns the feed's
            # own URL as a normal Python string.
            """

        def feed_url(self):
            """
            Returns the feed's own URL as a normal Python string.
            """

        feed_url = '/blog/rss/' # Hard-coded URL.

        # GUID -- One of the following three is optional. The framework looks
        # for them in this order. This property is only used for Atom feeds
        # (where it is the feed-level ID element). If not provided, the feed
        # link is used as the ID.

        def feed_guid(self, obj):
            """
            Takes the object returned by get_object() and returns the globally
            unique ID for the feed as a normal Python string.
            """

        def feed_guid(self):
            """
            Returns the feed's globally unique ID as a normal Python string.
            """

        feed_guid = '/foo/bar/1234' # Hard-coded guid.

        # DESCRIPTION -- One of the following three is required. The framework
        # looks for them in this order.

        def description(self, obj):
            """
            Takes the object returned by get_object() and returns the feed's
            description as a normal Python string.
            """

        def description(self):
            """
            Returns the feed's description as a normal Python string.
            """

        description = 'Foo bar baz.' # Hard-coded description.

        # AUTHOR NAME --One of the following three is optional. The framework
        # looks for them in this order.

        def author_name(self, obj):
            """
            Takes the object returned by get_object() and returns the feed's
            author's name as a normal Python string.
            """

        def author_name(self):
            """
            Returns the feed's author's name as a normal Python string.
            """

        author_name = 'Sally Smith' # Hard-coded author name.

        # AUTHOR EMAIL --One of the following three is optional. The framework
        # looks for them in this order.

        def author_email(self, obj):
            """
            Takes the object returned by get_object() and returns the feed's
            author's email as a normal Python string.
            """

        def author_email(self):
            """
            Returns the feed's author's email as a normal Python string.
            """

        author_email = 'test@example.com' # Hard-coded author email.

        # AUTHOR LINK --One of the following three is optional. The framework
        # looks for them in this order. In each case, the URL should include
        # the "http://" and domain name.

        def author_link(self, obj):
            """
            Takes the object returned by get_object() and returns the feed's
            author's URL as a normal Python string.
            """

        def author_link(self):
            """
            Returns the feed's author's URL as a normal Python string.
            """

        author_link = 'http://www.example.com/' # Hard-coded author URL.

        # CATEGORIES -- One of the following three is optional. The framework
        # looks for them in this order. In each case, the method/attribute
        # should return an iterable object that returns strings.

        def categories(self, obj):
            """
            Takes the object returned by get_object() and returns the feed's
            categories as iterable over strings.
            """

        def categories(self):
            """
            Returns the feed's categories as iterable over strings.
            """

        categories = ("python", "django") # Hard-coded list of categories.

        # COPYRIGHT NOTICE -- One of the following three is optional. The
        # framework looks for them in this order.

        def feed_copyright(self, obj):
            """
            Takes the object returned by get_object() and returns the feed's
            copyright notice as a normal Python string.
            """

        def feed_copyright(self):
            """
            Returns the feed's copyright notice as a normal Python string.
            """

        feed_copyright = 'Copyright (c) 2007, Sally Smith' # Hard-coded copyright notice.

        # TTL -- One of the following three is optional. The framework looks
        # for them in this order. Ignored for Atom feeds.

        def ttl(self, obj):
            """
            Takes the object returned by get_object() and returns the feed's
            TTL (Time To Live) as a normal Python string.
            """

        def ttl(self):
            """
            Returns the feed's TTL as a normal Python string.
            """

        ttl = 600 # Hard-coded Time To Live.

        # ITEMS -- One of the following three is required. The framework looks
        # for them in this order.

        def items(self, obj):
            """
            Takes the object returned by get_object() and returns a list of
            items to publish in this feed.
            """

        def items(self):
            """
            Returns a list of items to publish in this feed.
            """

        items = ('Item 1', 'Item 2') # Hard-coded items.

        # GET_OBJECT -- This is required for feeds that publish different data
        # for different URL parameters. (See "A complex example" above.)

        def get_object(self, request, *args, **kwargs):
            """
            Takes the current request and the arguments from the URL, and
            returns an object represented by this feed. Raises
            django.core.exceptions.ObjectDoesNotExist on error.
            """

        # ITEM TITLE AND DESCRIPTION -- If title_template or
        # description_template are not defined, these are used instead. Both are
        # optional, by default they will use the unicode representation of the
        # item.

        def item_title(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            title as a normal Python string.
            """

        def item_title(self):
            """
            Returns the title for every item in the feed.
            """

        item_title = 'Breaking News: Nothing Happening' # Hard-coded title.

        def item_description(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            description as a normal Python string.
            """

        def item_description(self):
            """
            Returns the description for every item in the feed.
            """

        item_description = 'A description of the item.' # Hard-coded description.

        def get_context_data(self, **kwargs):
            """
            Returns a dictionary to use as extra context if either
            description_template or item_template are used.

            Default implementation preserves the old behavior
            of using {'obj': item, 'site': current_site} as the context.
            """

        # ITEM LINK -- One of these three is required. The framework looks for
        # them in this order.

        # First, the framework tries the two methods below, in
        # order. Failing that, it falls back to the get_absolute_url()
        # method on each item returned by items().

        def item_link(self, item):
            """
            Takes an item, as returned by items(), and returns the item's URL.
            """

        def item_link(self):
            """
            Returns the URL for every item in the feed.
            """

        # ITEM_GUID -- The following method is optional. If not provided, the
        # item's link is used by default.

        def item_guid(self, obj):
            """
            Takes an item, as return by items(), and returns the item's ID.
            """

        # ITEM_GUID_IS_PERMALINK -- The following method is optional. If
        # provided, it sets the 'isPermaLink' attribute of an item's
        # GUID element. This method is used only when 'item_guid' is
        # specified.

        def item_guid_is_permalink(self, obj):
            """
            Takes an item, as returned by items(), and returns a boolean.
            """

        item_guid_is_permalink = False  # Hard coded value

        # ITEM AUTHOR NAME -- One of the following three is optional. The
        # framework looks for them in this order.

        def item_author_name(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            author's name as a normal Python string.
            """

        def item_author_name(self):
            """
            Returns the author name for every item in the feed.
            """

        item_author_name = 'Sally Smith' # Hard-coded author name.

        # ITEM AUTHOR EMAIL --One of the following three is optional. The
        # framework looks for them in this order.
        #
        # If you specify this, you must specify item_author_name.

        def item_author_email(self, obj):
            """
            Takes an item, as returned by items(), and returns the item's
            author's email as a normal Python string.
            """

        def item_author_email(self):
            """
            Returns the author email for every item in the feed.
            """

        item_author_email = 'test@example.com' # Hard-coded author email.

        # ITEM AUTHOR LINK -- One of the following three is optional. The
        # framework looks for them in this order. In each case, the URL should
        # include the "http://" and domain name.
        #
        # If you specify this, you must specify item_author_name.

        def item_author_link(self, obj):
            """
            Takes an item, as returned by items(), and returns the item's
            author's URL as a normal Python string.
            """

        def item_author_link(self):
            """
            Returns the author URL for every item in the feed.
            """

        item_author_link = 'http://www.example.com/' # Hard-coded author URL.

        # ITEM ENCLOSURE URL -- One of these three is required if you're
        # publishing enclosures. The framework looks for them in this order.

        def item_enclosure_url(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            enclosure URL.
            """

        def item_enclosure_url(self):
            """
            Returns the enclosure URL for every item in the feed.
            """

        item_enclosure_url = "/foo/bar.mp3" # Hard-coded enclosure link.

        # ITEM ENCLOSURE LENGTH -- One of these three is required if you're
        # publishing enclosures. The framework looks for them in this order.
        # In each case, the returned value should be either an integer, or a
        # string representation of the integer, in bytes.

        def item_enclosure_length(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            enclosure length.
            """

        def item_enclosure_length(self):
            """
            Returns the enclosure length for every item in the feed.
            """

        item_enclosure_length = 32000 # Hard-coded enclosure length.

        # ITEM ENCLOSURE MIME TYPE -- One of these three is required if you're
        # publishing enclosures. The framework looks for them in this order.

        def item_enclosure_mime_type(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            enclosure MIME type.
            """

        def item_enclosure_mime_type(self):
            """
            Returns the enclosure MIME type for every item in the feed.
            """

        item_enclosure_mime_type = "audio/mpeg" # Hard-coded enclosure MIME type.

        # ITEM PUBDATE -- It's optional to use one of these three. This is a
        # hook that specifies how to get the pubdate for a given item.
        # In each case, the method/attribute should return a Python
        # datetime.datetime object.

        def item_pubdate(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            pubdate.
            """

        def item_pubdate(self):
            """
            Returns the pubdate for every item in the feed.
            """

        item_pubdate = datetime.datetime(2005, 5, 3) # Hard-coded pubdate.

        # ITEM UPDATED -- It's optional to use one of these three. This is a
        # hook that specifies how to get the updateddate for a given item.
        # In each case, the method/attribute should return a Python
        # datetime.datetime object.

        def item_updateddate(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            updateddate.
            """

        def item_updateddate(self):
            """
            Returns the updateddated for every item in the feed.
            """

        item_updateddate = datetime.datetime(2005, 5, 3) # Hard-coded updateddate.

        # ITEM CATEGORIES -- It's optional to use one of these three. This is
        # a hook that specifies how to get the list of categories for a given
        # item. In each case, the method/attribute should return an iterable
        # object that returns strings.

        def item_categories(self, item):
            """
            Takes an item, as returned by items(), and returns the item's
            categories.
            """

        def item_categories(self):
            """
            Returns the categories for every item in the feed.
            """

        item_categories = ("python", "django") # Hard-coded categories.

        # ITEM COPYRIGHT NOTICE (only applicable to Atom feeds) -- One of the
        # following three is optional. The framework looks for them in this
        # order.

        def item_copyright(self, obj):
            """
            Takes an item, as returned by items(), and returns the item's
            copyright notice as a normal Python string.
            """

        def item_copyright(self):
            """
            Returns the copyright notice for every item in the feed.
            """

        item_copyright = 'Copyright (c) 2007, Sally Smith' # Hard-coded copyright notice.


The low-level framework
=======================

Behind the scenes, the high-level RSS framework uses a lower-level framework
for generating feeds' XML. This framework lives in a single module:
`django/utils/feedgenerator.py`_.

You use this framework on your own, for lower-level feed generation. You can
also create custom feed generator subclasses for use with the ``feed_type``
``Feed`` option.

.. currentmodule:: django.utils.feedgenerator

``SyndicationFeed`` classes
---------------------------

The :mod:`~django.utils.feedgenerator` module contains a base class:

* :class:`django.utils.feedgenerator.SyndicationFeed`

and several subclasses:

* :class:`django.utils.feedgenerator.RssUserland091Feed`
* :class:`django.utils.feedgenerator.Rss201rev2Feed`
* :class:`django.utils.feedgenerator.Atom1Feed`

Each of these three classes knows how to render a certain type of feed as XML.
They share this interface:

:meth:`.SyndicationFeed.__init__`
    Initialize the feed with the given dictionary of metadata, which applies to
    the entire feed. Required keyword arguments are:

    * ``title``
    * ``link``
    * ``description``

    There's also a bunch of other optional keywords:

    * ``language``
    * ``author_email``
    * ``author_name``
    * ``author_link``
    * ``subtitle``
    * ``categories``
    * ``feed_url``
    * ``feed_copyright``
    * ``feed_guid``
    * ``ttl``

    Any extra keyword arguments you pass to ``__init__`` will be stored in
    ``self.feed`` for use with `custom feed generators`_.

    All parameters should be Unicode objects, except ``categories``, which
    should be a sequence of Unicode objects.

:meth:`.SyndicationFeed.add_item`
    Add an item to the feed with the given parameters.

    Required keyword arguments are:

    * ``title``
    * ``link``
    * ``description``

    Optional keyword arguments are:

    * ``author_email``
    * ``author_name``
    * ``author_link``
    * ``pubdate``
    * ``comments``
    * ``unique_id``
    * ``enclosure``
    * ``categories``
    * ``item_copyright``
    * ``ttl``
    * ``updateddate``

    Extra keyword arguments will be stored for `custom feed generators`_.

    All parameters, if given, should be Unicode objects, except:

    * ``pubdate`` should be a Python  :class:`~datetime.datetime` object.
    * ``updateddate`` should be a Python  :class:`~datetime.datetime` object.
    * ``enclosure`` should be an instance of
      :class:`django.utils.feedgenerator.Enclosure`.
    * ``categories`` should be a sequence of Unicode objects.

:meth:`.SyndicationFeed.write`
    Outputs the feed in the given encoding to outfile, which is a file-like object.

:meth:`.SyndicationFeed.writeString`
    Returns the feed as a string in the given encoding.

For example, to create an Atom 1.0 feed and print it to standard output::

    >>> from django.utils import feedgenerator
    >>> from datetime import datetime
    >>> f = feedgenerator.Atom1Feed(
    ...     title="My Weblog",
    ...     link="http://www.example.com/",
    ...     description="In which I write about what I ate today.",
    ...     language="en",
    ...     author_name="Myself",
    ...     feed_url="http://example.com/atom.xml")
    >>> f.add_item(title="Hot dog today",
    ...     link="http://www.example.com/entries/1/",
    ...     pubdate=datetime.now(),
    ...     description="<p>Today I had a Vienna Beef hot dog. It was pink, plump and perfect.</p>")
    >>> print(f.writeString('UTF-8'))
    <?xml version="1.0" encoding="UTF-8"?>
    <feed xmlns="http://www.w3.org/2005/Atom" xml:lang="en">
    ...
    </feed>

.. _django/utils/feedgenerator.py: https://github.com/django/django/blob/master/django/utils/feedgenerator.py

.. currentmodule:: django.contrib.syndication

Custom feed generators
----------------------

If you need to produce a custom feed format, you've got a couple of options.

If the feed format is totally custom, you'll want to subclass
``SyndicationFeed`` and completely replace the ``write()`` and
``writeString()`` methods.

However, if the feed format is a spin-off of RSS or Atom (i.e. GeoRSS_, Apple's
`iTunes podcast format`_, etc.), you've got a better choice. These types of
feeds typically add extra elements and/or attributes to the underlying format,
and there are a set of methods that ``SyndicationFeed`` calls to get these extra
attributes. Thus, you can subclass the appropriate feed generator class
(``Atom1Feed`` or ``Rss201rev2Feed``) and extend these callbacks. They are:

.. _georss: http://georss.org/
.. _itunes podcast format: http://www.apple.com/itunes/podcasts/specs.html

``SyndicationFeed.root_attributes(self, )``
    Return a ``dict`` of attributes to add to the root feed element
    (``feed``/``channel``).

``SyndicationFeed.add_root_elements(self, handler)``
    Callback to add elements inside the root feed element
    (``feed``/``channel``). ``handler`` is an
    :class:`~xml.sax.saxutils.XMLGenerator` from Python's built-in SAX library;
    you'll call methods on it to add to the XML document in process.

``SyndicationFeed.item_attributes(self, item)``
    Return a ``dict`` of attributes to add to each item (``item``/``entry``)
    element. The argument, ``item``, is a dictionary of all the data passed to
    ``SyndicationFeed.add_item()``.

``SyndicationFeed.add_item_elements(self, handler, item)``
    Callback to add elements to each item (``item``/``entry``) element.
    ``handler`` and ``item`` are as above.

.. warning::

    If you override any of these methods, be sure to call the superclass methods
    since they add the required elements for each feed format.

For example, you might start implementing an iTunes RSS feed generator like so::

    class iTunesFeed(Rss201rev2Feed):
        def root_attributes(self):
            attrs = super(iTunesFeed, self).root_attributes()
            attrs['xmlns:itunes'] = 'http://www.itunes.com/dtds/podcast-1.0.dtd'
            return attrs

        def add_root_elements(self, handler):
            super(iTunesFeed, self).add_root_elements(handler)
            handler.addQuickElement('itunes:explicit', 'clean')

Obviously there's a lot more work to be done for a complete custom feed class,
but the above example should demonstrate the basic idea.

The Sitemap Framework
=====================

A *sitemap* is an XML file on your Web site that tells search engine indexers
how frequently your pages change and how "important" certain pages are in
relation to other pages on your site. This information helps search engines
index your site.

For more on sitemaps, see http://www.sitemaps.org/.

The Django sitemap framework automates the creation of this XML file by letting
you express this information in Python code.

It works much like Django's syndication framework
. To create a sitemap, just write a
:class:`~django.contrib.sitemaps.Sitemap` class and point to it in your
URLconf .

Installation
------------

To install the sitemap app, follow these steps:

1. Add ``'django.contrib.sitemaps'`` to your ``INSTALLED_APPS``
   setting.

2. Make sure your ``TEMPLATES`` setting contains a ``DjangoTemplates``
   backend whose ``APP_DIRS`` options is set to ``True``. It's in there by
   default, so you'll only need to change this if you've changed that setting.

3. Make sure you've installed the
   :mod:`sites framework <django.contrib.sites>`.

(Note: The sitemap application doesn't install any database tables. The only
reason it needs to go into ``INSTALLED_APPS`` is so that the
:func:`~django.template.loaders.app_directories.Loader` template
loader can find the default templates.)

Initialization
--------------

.. function:: views.sitemap(request, sitemaps, section=None, template_name='sitemap.xml', content_type='application/xml')

To activate sitemap generation on your Django site, add this line to your
URLconf ::

    from django.contrib.sitemaps.views import sitemap

    url(r'^sitemap\.xml$', sitemap, {'sitemaps': sitemaps},
        name='django.contrib.sitemaps.views.sitemap')

This tells Django to build a sitemap when a client accesses :file:`/sitemap.xml`.

The name of the sitemap file is not important, but the location is. Search
engines will only index links in your sitemap for the current URL level and
below. For instance, if :file:`sitemap.xml` lives in your root directory, it may
reference any URL in your site. However, if your sitemap lives at
:file:`/content/sitemap.xml`, it may only reference URLs that begin with
:file:`/content/`.

The sitemap view takes an extra, required argument: ``{'sitemaps': sitemaps}``.
``sitemaps`` should be a dictionary that maps a short section label (e.g.,
``blog`` or ``news``) to its :class:`~django.contrib.sitemaps.Sitemap` class
(e.g., ``BlogSitemap`` or ``NewsSitemap``). It may also map to an *instance* of
a :class:`~django.contrib.sitemaps.Sitemap` class (e.g.,
``BlogSitemap(some_var)``).

Sitemap classes
---------------

A :class:`~django.contrib.sitemaps.Sitemap` class is a simple Python
class that represents a "section" of entries in your sitemap. For example,
one :class:`~django.contrib.sitemaps.Sitemap` class could represent
all the entries of your Weblog, while another could represent all of the
events in your events calendar.

In the simplest case, all these sections get lumped together into one
:file:`sitemap.xml`, but it's also possible to use the framework to generate a
sitemap index that references individual sitemap files, one per section. (See
`Creating a sitemap index`_ below.)

:class:`~django.contrib.sitemaps.Sitemap` classes must subclass
``django.contrib.sitemaps.Sitemap``. They can live anywhere in your codebase.

A simple example
----------------

Let's assume you have a blog system, with an ``Entry`` model, and you want your
sitemap to include all the links to your individual blog entries. Here's how
your sitemap class might look::

    from django.contrib.sitemaps import Sitemap
    from blog.models import Entry

    class BlogSitemap(Sitemap):
        changefreq = "never"
        priority = 0.5

        def items(self):
            return Entry.objects.filter(is_draft=False)

        def lastmod(self, obj):
            return obj.pub_date

Note:

* :attr:`~Sitemap.changefreq` and :attr:`~Sitemap.priority` are class
  attributes corresponding to ``<changefreq>`` and ``<priority>`` elements,
  respectively. They can be made callable as functions, as
  :attr:`~Sitemap.lastmod` was in the example.
* :attr:`~Sitemap.items()` is simply a method that returns a list of
  objects. The objects returned will get passed to any callable methods
  corresponding to a sitemap property (:attr:`~Sitemap.location`,
  :attr:`~Sitemap.lastmod`, :attr:`~Sitemap.changefreq`, and
  :attr:`~Sitemap.priority`).
* :attr:`~Sitemap.lastmod` should return a Python ``datetime`` object.
* There is no :attr:`~Sitemap.location` method in this example, but you
  can provide it in order to specify the URL for your object. By default,
  :attr:`~Sitemap.location()` calls ``get_absolute_url()`` on each object
  and returns the result.

Sitemap class reference
-----------------------

.. class:: Sitemap

    A ``Sitemap`` class can define the following methods/attributes:

    .. attribute:: Sitemap.items

        **Required.** A method that returns a list of objects. The framework
        doesn't care what *type* of objects they are; all that matters is that
        these objects get passed to the :attr:`~Sitemap.location()`,
        :attr:`~Sitemap.lastmod()`, :attr:`~Sitemap.changefreq()` and
        :attr:`~Sitemap.priority()` methods.

    .. attribute:: Sitemap.location

        **Optional.** Either a method or attribute.

        If it's a method, it should return the absolute path for a given object
        as returned by :attr:`~Sitemap.items()`.

        If it's an attribute, its value should be a string representing an
        absolute path to use for *every* object returned by
        :attr:`~Sitemap.items()`.

        In both cases, "absolute path" means a URL that doesn't include the
        protocol or domain. Examples:

        * Good: :file:`'/foo/bar/'`
        * Bad: :file:`'example.com/foo/bar/'`
        * Bad: :file:`'http://example.com/foo/bar/'`

        If :attr:`~Sitemap.location` isn't provided, the framework will call
        the ``get_absolute_url()`` method on each object as returned by
        :attr:`~Sitemap.items()`.

        To specify a protocol other than ``'http'``, use
        :attr:`~Sitemap.protocol`.

    .. attribute:: Sitemap.lastmod

        **Optional.** Either a method or attribute.

        If it's a method, it should take one argument -- an object as returned by
        :attr:`~Sitemap.items()` -- and return that object's last-modified date/time, as a Python
        ``datetime.datetime`` object.

        If it's an attribute, its value should be a Python ``datetime.datetime`` object
        representing the last-modified date/time for *every* object returned by
        :attr:`~Sitemap.items()`.

        If all items in a sitemap have a :attr:`~Sitemap.lastmod`, the sitemap
        generated by :func:`views.sitemap` will have a ``Last-Modified``
        header equal to the latest ``lastmod``. You can activate the
        :class:`~django.middleware.http.ConditionalGetMiddleware` to make
        Django respond appropriately to requests with an ``If-Modified-Since``
        header which will prevent sending the sitemap if it hasn't changed.

    .. attribute:: Sitemap.changefreq

        **Optional.** Either a method or attribute.

        If it's a method, it should take one argument -- an object as returned by
        :attr:`~Sitemap.items()` -- and return that object's change frequency, as a Python string.

        If it's an attribute, its value should be a string representing the change
        frequency of *every* object returned by :attr:`~Sitemap.items()`.

        Possible values for :attr:`~Sitemap.changefreq`, whether you use a method or attribute, are:

        * ``'always'``
        * ``'hourly'``
        * ``'daily'``
        * ``'weekly'``
        * ``'monthly'``
        * ``'yearly'``
        * ``'never'``

    .. attribute:: Sitemap.priority

        **Optional.** Either a method or attribute.

        If it's a method, it should take one argument -- an object as returned by
        :attr:`~Sitemap.items()` -- and return that object's priority, as either a string or float.

        If it's an attribute, its value should be either a string or float representing
        the priority of *every* object returned by :attr:`~Sitemap.items()`.

        Example values for :attr:`~Sitemap.priority`: ``0.4``, ``1.0``. The default priority of a
        page is ``0.5``. See the `sitemaps.org documentation`_ for more.

        .. _sitemaps.org documentation: http://www.sitemaps.org/protocol.html#prioritydef

    .. attribute:: Sitemap.protocol

        **Optional.**

        This attribute defines the protocol (``'http'`` or ``'https'``) of the
        URLs in the sitemap. If it isn't set, the protocol with which the
        sitemap was requested is used. If the sitemap is built outside the
        context of a request, the default is ``'http'``.

    .. attribute:: Sitemap.i18n

        **Optional.**

        A boolean attribute that defines if the URLs of this sitemap should
        be generated using all of your ``LANGUAGES``. The default is
        ``False``.

Shortcuts
---------

The sitemap framework provides a convenience class for a common case:

.. class:: GenericSitemap

    The :class:`django.contrib.sitemaps.GenericSitemap` class allows you to
    create a sitemap by passing it a dictionary which has to contain at least
    a ``queryset`` entry. This queryset will be used to generate the items
    of the sitemap. It may also have a ``date_field`` entry that
    specifies a date field for objects retrieved from the ``queryset``.
    This will be used for the :attr:`~Sitemap.lastmod` attribute in the
    generated sitemap. You may also pass :attr:`~Sitemap.priority` and
    :attr:`~Sitemap.changefreq` keyword arguments to the
    :class:`~django.contrib.sitemaps.GenericSitemap`  constructor to specify
    these attributes for all URLs.

Example
~~~~~~~

Here's an example of a URLconf  using
:class:`GenericSitemap`::

    from django.conf.urls import url
    from django.contrib.sitemaps import GenericSitemap
    from django.contrib.sitemaps.views import sitemap
    from blog.models import Entry

    info_dict = {
        'queryset': Entry.objects.all(),
        'date_field': 'pub_date',
    }

    urlpatterns = [
        # some generic view using info_dict
        # ...

        # the sitemap
        url(r'^sitemap\.xml$', sitemap,
            {'sitemaps': {'blog': GenericSitemap(info_dict, priority=0.6)}},
            name='django.contrib.sitemaps.views.sitemap'),
    ]

.. _URLconf: ../url_dispatch/

Sitemap for static views
------------------------

Often you want the search engine crawlers to index views which are neither
object detail pages nor flatpages. The solution is to explicitly list URL
names for these views in ``items`` and call
:func:`~django.core.urlresolvers.reverse` in the ``location`` method of
the sitemap. For example::

    # sitemaps.py
    from django.contrib import sitemaps
    from django.core.urlresolvers import reverse

    class StaticViewSitemap(sitemaps.Sitemap):
        priority = 0.5
        changefreq = 'daily'

        def items(self):
            return ['main', 'about', 'license']

        def location(self, item):
            return reverse(item)

    # urls.py
    from django.conf.urls import url
    from django.contrib.sitemaps.views import sitemap

    from .sitemaps import StaticViewSitemap
    from . import views

    sitemaps = {
        'static': StaticViewSitemap,
    }

    urlpatterns = [
        url(r'^$', views.main, name='main'),
        url(r'^about/$', views.about, name='about'),
        url(r'^license/$', views.license, name='license'),
        # ...
        url(r'^sitemap\.xml$', sitemap, {'sitemaps': sitemaps},
            name='django.contrib.sitemaps.views.sitemap')
    ]


Creating a sitemap index
------------------------

.. function:: views.index(request, sitemaps, template_name='sitemap_index.xml', content_type='application/xml', sitemap_url_name='django.contrib.sitemaps.views.sitemap')

The sitemap framework also has the ability to create a sitemap index that
references individual sitemap files, one per each section defined in your
``sitemaps`` dictionary. The only differences in usage are:

* You use two views in your URLconf: :func:`django.contrib.sitemaps.views.index`
  and :func:`django.contrib.sitemaps.views.sitemap`.
* The :func:`django.contrib.sitemaps.views.sitemap` view should take a
  ``section`` keyword argument.

Here's what the relevant URLconf lines would look like for the example above::

    from django.contrib.sitemaps import views

    urlpatterns = [
        url(r'^sitemap\.xml$', views.index, {'sitemaps': sitemaps}),
        url(r'^sitemap-(?P<section>.+)\.xml$', views.sitemap, {'sitemaps': sitemaps}),
    ]

This will automatically generate a :file:`sitemap.xml` file that references
both :file:`sitemap-flatpages.xml` and :file:`sitemap-blog.xml`. The
:class:`~django.contrib.sitemaps.Sitemap` classes and the ``sitemaps``
dict don't change at all.

You should create an index file if one of your sitemaps has more than 50,000
URLs. In this case, Django will automatically paginate the sitemap, and the
index will reflect that.

If you're not using the vanilla sitemap view -- for example, if it's wrapped
with a caching decorator -- you must name your sitemap view and pass
``sitemap_url_name`` to the index view::

    from django.contrib.sitemaps import views as sitemaps_views
    from django.views.decorators.cache import cache_page

    urlpatterns = [
        url(r'^sitemap\.xml$',
            cache_page(86400)(sitemaps_views.index),
            {'sitemaps': sitemaps, 'sitemap_url_name': 'sitemaps'}),
        url(r'^sitemap-(?P<section>.+)\.xml$',
            cache_page(86400)(sitemaps_views.sitemap),
            {'sitemaps': sitemaps}, name='sitemaps'),
    ]


Template customization
----------------------

If you wish to use a different template for each sitemap or sitemap index
available on your site, you may specify it by passing a ``template_name``
parameter to the ``sitemap`` and ``index`` views via the URLconf::

    from django.contrib.sitemaps import views

    urlpatterns = [
        url(r'^custom-sitemap\.xml$', views.index, {
            'sitemaps': sitemaps,
            'template_name': 'custom_sitemap.html'
        }),
        url(r'^custom-sitemap-(?P<section>.+)\.xml$', views.sitemap, {
            'sitemaps': sitemaps,
            'template_name': 'custom_sitemap.html'
        }),
    ]


These views return :class:`~django.template.response.TemplateResponse`
instances which allow you to easily customize the response data before
rendering.

Context variables
------------------

When customizing the templates for the
:func:`~django.contrib.sitemaps.views.index` and
:func:`~django.contrib.sitemaps.views.sitemap` views, you can rely on the
following context variables.

Index
~~~~~

The variable ``sitemaps`` is a list of absolute URLs to each of the sitemaps.

Sitemap
~~~~~~~

The variable ``urlset`` is a list of URLs that should appear in the
sitemap. Each URL exposes attributes as defined in the
:class:`~django.contrib.sitemaps.Sitemap` class:

- ``changefreq``
- ``item``
- ``lastmod``
- ``location``
- ``priority``

The ``item`` attribute has been added for each URL to allow more flexible
customization of the templates, such as `Google news sitemaps`_. Assuming
Sitemap's :attr:`~Sitemap.items()` would return a list of items with
``publication_data`` and a ``tags`` field something like this would
generate a Google News compatible sitemap:

.. code-block:: xml+django

    <?xml version="1.0" encoding="UTF-8"?>
    <urlset
      xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
      xmlns:news="http://www.google.com/schemas/sitemap-news/0.9">
    {% spaceless %}
    {% for url in urlset %}
      <url>
        <loc>{{ url.location }}</loc>
        {% if url.lastmod %}<lastmod>{{ url.lastmod|date:"Y-m-d" }}</lastmod>{% endif %}
        {% if url.changefreq %}<changefreq>{{ url.changefreq }}</changefreq>{% endif %}
        {% if url.priority %}<priority>{{ url.priority }}</priority>{% endif %}
        <news:news>
          {% if url.item.publication_date %}<news:publication_date>{{ url.item.publication_date|date:"Y-m-d" }}</news:publication_date>{% endif %}
          {% if url.item.tags %}<news:keywords>{{ url.item.tags }}</news:keywords>{% endif %}
        </news:news>
       </url>
    {% endfor %}
    {% endspaceless %}
    </urlset>

.. _`Google news sitemaps`: https://support.google.com/news/publisher/answer/74288?hl=en

Pinging Google
--------------

You may want to "ping" Google when your sitemap changes, to let it know to
reindex your site. The sitemaps framework provides a function to do just
that: :func:`django.contrib.sitemaps.ping_google()`.

.. function:: ping_google

    :func:`ping_google` takes an optional argument, ``sitemap_url``,
    which should be the absolute path to your site's sitemap (e.g.,
    :file:`'/sitemap.xml'`). If this argument isn't provided,
    :func:`ping_google` will attempt to figure out your
    sitemap by performing a reverse looking in your URLconf.

    :func:`ping_google` raises the exception
    ``django.contrib.sitemaps.SitemapNotFound`` if it cannot determine your
    sitemap URL.

.. admonition:: Register with Google first!

    The :func:`ping_google` command only works if you have registered your
    site with `Google Webmaster Tools`_.

.. _`Google Webmaster Tools`: http://www.google.com/webmasters/tools/

One useful way to call :func:`ping_google` is from a model's ``save()``
method::

    from django.contrib.sitemaps import ping_google

    class Entry(models.Model):
        # ...
        def save(self, force_insert=False, force_update=False):
            super(Entry, self).save(force_insert, force_update)
            try:
                ping_google()
            except Exception:
                # Bare 'except' because we could get a variety
                # of HTTP-related exceptions.
                pass

A more efficient solution, however, would be to call :func:`ping_google` from a
cron script, or some other scheduled task. The function makes an HTTP request
to Google's servers, so you may not want to introduce that network overhead
each time you call ``save()``.

Pinging Google via ``manage.py``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the sitemaps application is added to your project, you may also
ping Google using the ``ping_google`` management command::

    python manage.py ping_google [/sitemap.xml]

What's Next?
============

Next, we'll continue to dig deeper into the built-in tools Django gives you by
taking a closer look at the Django session framework.
