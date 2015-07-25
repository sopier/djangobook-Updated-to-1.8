==============================
Chapter 12 - testing in Django
==============================

[TODO this is pretty much the Django docs. Needs simplification in reference
material and more code and screenshots to tie into the books example]

Introducing automated testing
=============================

What are automated tests?
-------------------------

Tests are simple routines that check the operation of your code.

Testing operates at different levels. Some tests might apply to a tiny detail
(*does a particular model method return values as expected?*) while others
examine the overall operation of the software (*does a sequence of user inputs
on the site produce the desired result?*). That's no different from the kind of
testing you did earlier in Tutorial 1 , using the
shell to examine the behavior of a method, or running the
application and entering data to check how it behaves.

What's different in *automated* tests is that the testing work is done for
you by the system. You create a set of tests once, and then as you make changes
to your app, you can check that your code still works as you originally
intended, without having to perform time consuming manual testing.

Why you need to create tests
----------------------------

So why create tests, and why now?

You may feel that you have quite enough on your plate just learning
Python/Django, and having yet another thing to learn and do may seem
overwhelming and perhaps unnecessary. After all, our polls application is
working quite happily now; going through the trouble of creating automated
tests is not going to make it work any better. If creating the polls
application is the last bit of Django programming you will ever do, then true,
you don't need to know how to create automated tests. But, if that's not the
case, now is an excellent time to learn.

Tests will save you time
~~~~~~~~~~~~~~~~~~~~~~~~

Up to a certain point, 'checking that it seems to work' will be a satisfactory
test. In a more sophisticated application, you might have dozens of complex
interactions between components.

A change in any of those components could have unexpected consequences on the
application's behavior. Checking that it still 'seems to work' could mean
running through your code's functionality with twenty different variations of
your test data just to make sure you haven't broken something - not a good use
of your time.

That's especially true when automated tests could do this for you in seconds.
If something's gone wrong, tests will also assist in identifying the code
that's causing the unexpected behavior.

Sometimes it may seem a chore to tear yourself away from your productive,
creative programming work to face the unglamorous and unexciting business
of writing tests, particularly when you know your code is working properly.

However, the task of writing tests is a lot more fulfilling than spending hours
testing your application manually or trying to identify the cause of a
newly-introduced problem.

Tests don't just identify problems, they prevent them
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It's a mistake to think of tests merely as a negative aspect of development.

Without tests, the purpose or intended behavior of an application might be
rather opaque. Even when it's your own code, you will sometimes find yourself
poking around in it trying to find out what exactly it's doing.

Tests change that; they light up your code from the inside, and when something
goes wrong, they focus light on the part that has gone wrong - *even if you
hadn't even realized it had gone wrong*.

Tests make your code more attractive
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You might have created a brilliant piece of software, but you will find that
many other developers will simply refuse to look at it because it lacks tests;
without tests, they won't trust it. Jacob Kaplan-Moss, one of Django's
original developers, says "Code without tests is broken by design."

That other developers want to see tests in your software before they take it
seriously is yet another reason for you to start writing tests.

Tests help teams work together
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The previous points are written from the point of view of a single developer
maintaining an application. Complex applications will be maintained by teams.
Tests guarantee that colleagues don't inadvertently break your code (and that
you don't break theirs without knowing). If you want to make a living as a
Django programmer, you must be good at writing tests!

Basic testing strategies
========================

There are many ways to approach writing tests.

Some programmers follow a discipline called "`test-driven development`_"; they
actually write their tests before they write their code. This might seem
counter-intuitive, but in fact it's similar to what most people will often do
anyway: they describe a problem, then create some code to solve it. Test-driven
development simply formalizes the problem in a Python test case.

More often, a newcomer to testing will create some code and later decide that
it should have some tests. Perhaps it would have been better to write some
tests earlier, but it's never too late to get started.

Sometimes it's difficult to figure out where to get started with writing tests.
If you have written several thousand lines of Python, choosing something to
test might not be easy. In such a case, it's fruitful to write your first test
the next time you make a change, either when you add a new feature or fix a bug.

So let's do that right away.

.. _test-driven development: http://en.wikipedia.org/wiki/Test-driven_development

Writing our first test
======================

We identify a bug
-----------------

Fortunately, there's a little bug in the ``polls`` application for us to fix
right away: the ``Question.was_published_recently()`` method returns ``True`` if
the ``Question`` was published within the last day (which is correct) but also if
the ``Question``’s ``pub_date`` field is in the future (which certainly isn't).

You can see this in the Admin; create a question whose date lies in the future;
you'll see that the ``Question`` change list claims it was published recently.

You can also see this using the shell::

    >>> import datetime
    >>> from django.utils import timezone
    >>> from polls.models import Question
    >>> # create a Question instance with pub_date 30 days in the future
    >>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
    >>> # was it published recently?
    >>> future_question.was_published_recently()
    True

Since things in the future are not 'recent', this is clearly wrong.

Create a test to expose the bug
-------------------------------

What we've just done in the shell to test for the problem is exactly
what we can do in an automated test, so let's turn that into an automated test.

A conventional place for an application's tests is in the application's
``tests.py`` file; the testing system will automatically find tests in any file
whose name begins with ``test``.

Put the following in the ``tests.py`` file in the ``polls`` application:

.. code-block:: python

    # polls/tests.py

    import datetime

    from django.utils import timezone
    from django.test import TestCase

    from polls.models import Question

    class QuestionMethodTests(TestCase):

        def test_was_published_recently_with_future_question(self):
            """
            was_published_recently() should return False for questions whose
            pub_date is in the future
            """
            time = timezone.now() + datetime.timedelta(days=30)
            future_question = Question(pub_date=time)
            self.assertEqual(future_question.was_published_recently(), False)

What we have done here is created a :class:`django.test.TestCase` subclass
with a method that creates a ``Question`` instance with a ``pub_date`` in the
future. We then check the output of ``was_published_recently()`` - which
*ought* to be False.

Running tests
-------------

In the terminal, we can run our test::

    $ python manage.py test polls

and you'll see something like::

    Creating test database for alias 'default'...
    F
    ======================================================================
    FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionMethodTests)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "/path/to/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_question
        self.assertEqual(future_question.was_published_recently(), False)
    AssertionError: True != False

    ----------------------------------------------------------------------
    Ran 1 test in 0.001s

    FAILED (failures=1)
    Destroying test database for alias 'default'...

What happened is this:

* ``python manage.py test polls`` looked for tests in the ``polls`` application

* it found a subclass of the :class:`django.test.TestCase` class

* it created a special database for the purpose of testing

* it looked for test methods - ones whose names begin with ``test``

* in ``test_was_published_recently_with_future_question`` it created a ``Question``
  instance whose ``pub_date`` field is 30 days in the future

* ... and using the ``assertEqual()`` method, it discovered that its
  ``was_published_recently()`` returns ``True``, though we wanted it to return
  ``False``

The test informs us which test failed and even the line on which the failure
occurred.

Fixing the bug
--------------

We already know what the problem is: ``Question.was_published_recently()`` should
return ``False`` if its ``pub_date`` is in the future. Amend the method in
``models.py``, so that it will only return ``True`` if the date is also in the
past:

.. code-block:: python
    
    # polls/models.py

    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now

and run the test again::

    Creating test database for alias 'default'...
    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.001s

    OK
    Destroying test database for alias 'default'...

After identifying a bug, we wrote a test that exposes it and corrected the bug
in the code so our test passes.

Many other things might go wrong with our application in the future, but we can
be sure that we won't inadvertently reintroduce this bug, because simply
running the test will warn us immediately. We can consider this little portion
of the application pinned down safely forever.

More comprehensive tests
------------------------

While we're here, we can further pin down the ``was_published_recently()``
method; in fact, it would be positively embarrassing if in fixing one bug we had
introduced another.

Add two more test methods to the same class, to test the behavior of the method
more comprehensively:

.. code-block:: python
    
    # polls/tests.py

    def test_was_published_recently_with_old_question(self):
        """
        was_published_recently() should return False for questions whose
        pub_date is older than 1 day
        """
        time = timezone.now() - datetime.timedelta(days=30)
        old_question = Question(pub_date=time)
        self.assertEqual(old_question.was_published_recently(), False)

    def test_was_published_recently_with_recent_question(self):
        """
        was_published_recently() should return True for questions whose
        pub_date is within the last day
        """
        time = timezone.now() - datetime.timedelta(hours=1)
        recent_question = Question(pub_date=time)
        self.assertEqual(recent_question.was_published_recently(), True)

And now we have three tests that confirm that ``Question.was_published_recently()``
returns sensible values for past, recent, and future questions.

Again, ``polls`` is a simple application, but however complex it grows in the
future and whatever other code it interacts with, we now have some guarantee
that the method we have written tests for will behave in expected ways.

Test a view
===========

The polls application is fairly undiscriminating: it will publish any question,
including ones whose ``pub_date`` field lies in the future. We should improve
this. Setting a ``pub_date`` in the future should mean that the Question is
published at that moment, but invisible until then.

A test for a view
-----------------

When we fixed the bug above, we wrote the test first and then the code to fix
it. In fact that was a simple example of test-driven development, but it
doesn't really matter in which order we do the work.

In our first test, we focused closely on the internal behavior of the code. For
this test, we want to check its behavior as it would be experienced by a user
through a web browser.

Before we try to fix anything, let's have a look at the tools at our disposal.

The Django test client
----------------------

Django provides a test :class:`~django.test.Client` to simulate a user
interacting with the code at the view level.  We can use it in ``tests.py``
or even in the shell.

We will start again with the shell, where we need to do a couple of
things that won't be necessary in ``tests.py``. The first is to set up the test
environment in the shell::

    >>> from django.test.utils import setup_test_environment
    >>> setup_test_environment()

:meth:`~django.test.utils.setup_test_environment` installs a template renderer
which will allow us to examine some additional attributes on responses such as
``response.context`` that otherwise wouldn't be available. Note that this
method *does not* setup a test database, so the following will be run against
the existing database and the output may differ slightly depending on what
questions you already created.

Next we need to import the test client class (later in ``tests.py`` we will use
the :class:`django.test.TestCase` class, which comes with its own client, so
this won't be required)::

    >>> from django.test import Client
    >>> # create an instance of the client for our use
    >>> client = Client()

With that ready, we can ask the client to do some work for us::

    >>> # get a response from '/'
    >>> response = client.get('/')
    >>> # we should expect a 404 from that address
    >>> response.status_code
    404
    >>> # on the other hand we should expect to find something at '/polls/'
    >>> # we'll use 'reverse()' rather than a hardcoded URL
    >>> from django.core.urlresolvers import reverse
    >>> response = client.get(reverse('polls:index'))
    >>> response.status_code
    200
    >>> response.content
    '\n\n\n    <p>No polls are available.</p>\n\n'
    >>> # note - you might get unexpected results if your ``TIME_ZONE``
    >>> # in ``settings.py`` is not correct. If you need to change it,
    >>> # you will also need to restart your shell session
    >>> from polls.models import Question
    >>> from django.utils import timezone
    >>> # create a Question and save it
    >>> q = Question(question_text="Who is your favorite Beatle?", pub_date=timezone.now())
    >>> q.save()
    >>> # check the response once again
    >>> response = client.get('/polls/')
    >>> response.content
    '\n\n\n    <ul>\n    \n        <li><a href="/polls/1/">Who is your favorite Beatle?</a></li>\n    \n    </ul>\n\n'
    >>> # If the following doesn't work, you probably omitted the call to
    >>> # setup_test_environment() described above
    >>> response.context['latest_question_list']
    [<Question: Who is your favorite Beatle?>]

Improving our view
------------------

The list of polls shows polls that aren't published yet (i.e. those that have a
``pub_date`` in the future). Let's fix that.

In Tutorial 4 we introduced a class-based view,
based on :class:`~django.views.generic.list.ListView`:

.. code-block:: python
    
   # polls/views.py

    class IndexView(generic.ListView):
        template_name = 'polls/index.html'
        context_object_name = 'latest_question_list'

        def get_queryset(self):
            """Return the last five published questions."""
            return Question.objects.order_by('-pub_date')[:5]

``response.context_data['latest_question_list']`` extracts the data this view
places into the context.

We need to amend the ``get_queryset`` method and change it so that it also
checks the date by comparing it with ``timezone.now()``. First we need to add
an import:

.. code-block:: python
    
    # polls/views.py

    from django.utils import timezone

and then we must amend the ``get_queryset`` method like so:

.. code-block:: python
    
    # polls/views.py

    def get_queryset(self):
        """
        Return the last five published questions (not including those set to be
        published in the future).
        """
        return Question.objects.filter(
            pub_date__lte=timezone.now()
        ).order_by('-pub_date')[:5]

``Question.objects.filter(pub_date__lte=timezone.now())`` returns a queryset
containing ``Question``\s whose ``pub_date`` is less than or equal to - that
is, earlier than or equal to - ``timezone.now``.

Testing our new view
--------------------

Now you can satisfy yourself that this behaves as expected by firing up the
runserver, loading the site in your browser, creating ``Questions`` with dates
in the past and future, and checking that only those that have been published
are listed.  You don't want to have to do that *every single time you make any
change that might affect this* - so let's also create a test, based on our
shell session above.

Add the following to ``polls/tests.py``:

.. code-block:: python
    
    # polls/tests.py

    from django.core.urlresolvers import reverse

and we'll create a shortcut function to create questions as well as a new test
class:

.. code-block:: python
    
    # polls/tests.py

    def create_question(question_text, days):
        """
        Creates a question with the given `question_text` published the given
        number of `days` offset to now (negative for questions published
        in the past, positive for questions that have yet to be published).
        """
        time = timezone.now() + datetime.timedelta(days=days)
        return Question.objects.create(question_text=question_text,
                                       pub_date=time)


    class QuestionViewTests(TestCase):
        def test_index_view_with_no_questions(self):
            """
            If no questions exist, an appropriate message should be displayed.
            """
            response = self.client.get(reverse('polls:index'))
            self.assertEqual(response.status_code, 200)
            self.assertContains(response, "No polls are available.")
            self.assertQuerysetEqual(response.context['latest_question_list'], [])

        def test_index_view_with_a_past_question(self):
            """
            Questions with a pub_date in the past should be displayed on the
            index page
            """
            create_question(question_text="Past question.", days=-30)
            response = self.client.get(reverse('polls:index'))
            self.assertQuerysetEqual(
                response.context['latest_question_list'],
                ['<Question: Past question.>']
            )

        def test_index_view_with_a_future_question(self):
            """
            Questions with a pub_date in the future should not be displayed on
            the index page.
            """
            create_question(question_text="Future question.", days=30)
            response = self.client.get(reverse('polls:index'))
            self.assertContains(response, "No polls are available.",
                                status_code=200)
            self.assertQuerysetEqual(response.context['latest_question_list'], [])

        def test_index_view_with_future_question_and_past_question(self):
            """
            Even if both past and future questions exist, only past questions
            should be displayed.
            """
            create_question(question_text="Past question.", days=-30)
            create_question(question_text="Future question.", days=30)
            response = self.client.get(reverse('polls:index'))
            self.assertQuerysetEqual(
                response.context['latest_question_list'],
                ['<Question: Past question.>']
            )

        def test_index_view_with_two_past_questions(self):
            """
            The questions index page may display multiple questions.
            """
            create_question(question_text="Past question 1.", days=-30)
            create_question(question_text="Past question 2.", days=-5)
            response = self.client.get(reverse('polls:index'))
            self.assertQuerysetEqual(
                response.context['latest_question_list'],
                ['<Question: Past question 2.>', '<Question: Past question 1.>']
            )


Let's look at some of these more closely.

First is a question shortcut function, ``create_question``, to take some
repetition out of the process of creating questions.

``test_index_view_with_no_questions`` doesn't create any questions, but checks
the message: "No polls are available." and verifies the ``latest_question_list``
is empty. Note that the :class:`django.test.TestCase` class provides some
additional assertion methods. In these examples, we use
:meth:`~django.test.SimpleTestCase.assertContains()` and
:meth:`~django.test.TransactionTestCase.assertQuerysetEqual()`.

In ``test_index_view_with_a_past_question``, we create a question and verify that it
appears in the list.

In ``test_index_view_with_a_future_question``, we create a question with a
``pub_date`` in the future. The database is reset for each test method, so the
first question is no longer there, and so again the index shouldn't have any
questions in it.

And so on. In effect, we are using the tests to tell a story of admin input
and user experience on the site, and checking that at every state and for every
new change in the state of the system, the expected results are published.

Testing the ``DetailView``
--------------------------

What we have works well; however, even though future questions don't appear in
the *index*, users can still reach them if they know or guess the right URL. So
we need to add a similar  constraint to ``DetailView``:

.. code-block:: python
    
    # polls/views.py

    class DetailView(generic.DetailView):
        ...
        def get_queryset(self):
            """
            Excludes any questions that aren't published yet.
            """
            return Question.objects.filter(pub_date__lte=timezone.now())

And of course, we will add some tests, to check that a ``Question`` whose
``pub_date`` is in the past can be displayed, and that one with a ``pub_date``
in the future is not:

.. code-block:: python
    
    # polls/tests.py

    class QuestionIndexDetailTests(TestCase):
        def test_detail_view_with_a_future_question(self):
            """
            The detail view of a question with a pub_date in the future should
            return a 404 not found.
            """
            future_question = create_question(question_text='Future question.',
                                              days=5)
            response = self.client.get(reverse('polls:detail',
                                       args=(future_question.id,)))
            self.assertEqual(response.status_code, 404)

        def test_detail_view_with_a_past_question(self):
            """
            The detail view of a question with a pub_date in the past should
            display the question's text.
            """
            past_question = create_question(question_text='Past Question.',
                                            days=-5)
            response = self.client.get(reverse('polls:detail',
                                       args=(past_question.id,)))
            self.assertContains(response, past_question.question_text,
                                status_code=200)


Ideas for more tests
--------------------

We ought to add a similar ``get_queryset`` method to ``ResultsView`` and
create a new test class for that view. It'll be very similar to what we have
just created; in fact there will be a lot of repetition.

We could also improve our application in other ways, adding tests along the
way. For example, it's silly that ``Questions`` can be published on the site
that have no ``Choices``. So, our views could check for this, and exclude such
``Questions``. Our tests would create a ``Question`` without ``Choices`` and
then test that it's not published, as well as create a similar ``Question``
*with* ``Choices``, and test that it *is* published.

Perhaps logged-in admin users should be allowed to see unpublished
``Questions``, but not ordinary visitors. Again: whatever needs to be added to
the software to accomplish this should be accompanied by a test, whether you
write the test first and then make the code pass the test, or work out the
logic in your code first and then write a test to prove it.

At a certain point you are bound to look at your tests and wonder whether your
code is suffering from test bloat, which brings us to:

When testing, more is better
============================

It might seem that our tests are growing out of control. At this rate there will
soon be more code in our tests than in our application, and the repetition
is unaesthetic, compared to the elegant conciseness of the rest of our code.

**It doesn't matter**. Let them grow. For the most part, you can write a test
once and then forget about it. It will continue performing its useful function
as you continue to develop your program.

Sometimes tests will need to be updated. Suppose that we amend our views so that
only ``Questions`` with ``Choices`` are published. In that case, many of our
existing tests will fail - *telling us exactly which tests need to be amended to
bring them up to date*, so to that extent tests help look after themselves.

At worst, as you continue developing, you might find that you have some tests
that are now redundant. Even that's not a problem; in testing redundancy is
a *good* thing.

As long as your tests are sensibly arranged, they won't become unmanageable.
Good rules-of-thumb include having:

* a separate ``TestClass`` for each model or view
* a separate test method for each set of conditions you want to test
* test method names that describe their function

Further testing
===============

This tutorial only introduces some of the basics of testing. There's a great
deal more you can do, and a number of very useful tools at your disposal to
achieve some very clever things.

For example, while our tests here have covered some of the internal logic of a
model and the way our views publish information, you can use an "in-browser"
framework such as Selenium_ to test the way your HTML actually renders in a
browser. These tools allow you to check not just the behavior of your Django
code, but also, for example, of your JavaScript. It's quite something to see
the tests launch a browser, and start interacting with your site, as if a human
being were driving it! Django includes :class:`~django.test.LiveServerTestCase`
to facilitate integration with tools like Selenium.

If you have a complex application, you may want to run tests automatically
with every commit for the purposes of `continuous integration`_, so that
quality control is itself - at least partially - automated.

A good way to spot untested parts of your application is to check code
coverage. This also helps identify fragile or even dead code. If you can't test
a piece of code, it usually means that code should be refactored or removed.
Coverage will help to identify dead code. 

.. _Selenium: http://seleniumhq.org/
.. _continuous integration: http://en.wikipedia.org/wiki/Continuous_integration

Testing a Web application is a complex task, because a Web application is made
of several layers of logic -- from HTTP-level request handling, to form
validation and processing, to template rendering. With Django's test-execution
framework and assorted utilities, you can simulate requests, insert test data,
inspect your application's output and generally verify your code is doing what
it should be doing.

The best part is, it's really easy.

The preferred way to write tests in Django is using the :mod:`unittest` module
built in to the Python standard library. 

You can also use any *other* Python test framework; Django provides an API and
tools for that kind of integration. They are described in the
other-testing-frameworks section of advanced.

Writing and running tests
=========================

.. module:: django.test
   :synopsis: Testing tools for Django applications.

This document is split into two primary sections. First, we explain how to write
tests with Django. Then, we explain how to run them.

Writing tests
-------------

Django's unit tests use a Python standard library module: :mod:`unittest`. This
module defines tests using a class-based approach.

Here is an example which subclasses from :class:`django.test.TestCase`,
which is a subclass of :class:`unittest.TestCase` that runs each test inside a
transaction to provide isolation::

    from django.test import TestCase
    from myapp.models import Animal

    class AnimalTestCase(TestCase):
        def setUp(self):
            Animal.objects.create(name="lion", sound="roar")
            Animal.objects.create(name="cat", sound="meow")

        def test_animals_can_speak(self):
            """Animals that can speak are correctly identified"""
            lion = Animal.objects.get(name="lion")
            cat = Animal.objects.get(name="cat")
            self.assertEqual(lion.speak(), 'The lion says "roar"')
            self.assertEqual(cat.speak(), 'The cat says "meow"')

When you run your tests, the default behavior of the
test utility is to find all the test cases (that is, subclasses of
:class:`unittest.TestCase`) in any file whose name begins with ``test``,
automatically build a test suite out of those test cases, and run that suite.

For more details about :mod:`unittest`, see the Python documentation.

.. warning::

    If your tests rely on database access such as creating or querying models,
    be sure to create your test classes as subclasses of
    :class:`django.test.TestCase` rather than :class:`unittest.TestCase`.

    Using :class:`unittest.TestCase` avoids the cost of running each test in a
    transaction and flushing the database, but if your tests interact with
    the database their behavior will vary based on the order that the test
    runner executes them. This can lead to unit tests that pass when run in
    isolation but fail when run in a suite.

.. _running-tests:


Running tests
-------------

Once you've written tests, run them using the test command of
your project's ``manage.py`` utility::

    $ ./manage.py test

Test discovery is based on the unittest module's built-in test
discovery.  By default, this will discover tests in
any file named "test*.py" under the current working directory.

You can specify particular tests to run by supplying any number of "test
labels" to ``./manage.py test``. Each test label can be a full Python dotted
path to a package, module, ``TestCase`` subclass, or test method. For instance::

    # Run all the tests in the animals.tests module
    $ ./manage.py test animals.tests

    # Run all the tests found within the 'animals' package
    $ ./manage.py test animals

    # Run just one test case
    $ ./manage.py test animals.tests.AnimalTestCase

    # Run just one test method
    $ ./manage.py test animals.tests.AnimalTestCase.test_animals_can_speak

You can also provide a path to a directory to discover tests below that
directory::

    $ ./manage.py test animals/

You can specify a custom filename pattern match using the ``-p`` (or
``--pattern``) option, if your test files are named differently from the
``test*.py`` pattern::

    $ ./manage.py test --pattern="tests_*.py"

If you press ``Ctrl-C`` while the tests are running, the test runner will
wait for the currently running test to complete and then exit gracefully.
During a graceful exit the test runner will output details of any test
failures, report on how many tests were run and how many errors and failures
were encountered, and destroy any test databases as usual. Thus pressing
``Ctrl-C`` can be very useful if you forget to pass the ``--failfast``
option, notice that some tests are unexpectedly failing, and want to get details
on the failures without waiting for the full test run to complete.

If you do not want to wait for the currently running test to finish, you
can press ``Ctrl-C`` a second time and the test run will halt immediately,
but not gracefully. No details of the tests run before the interruption will
be reported, and any test databases created by the run will not be destroyed.

.. admonition:: Test with warnings enabled

    It's a good idea to run your tests with Python warnings enabled:
    ``python -Wall manage.py test``. The ``-Wall`` flag tells Python to
    display deprecation warnings. Django, like many other Python libraries,
    uses these warnings to flag when features are going away. It also might
    flag areas in your code that aren't strictly wrong but could benefit
    from a better implementation.


.. _the-test-database:

The test database
-----------------

Tests that require a database (namely, model tests) will not use your "real"
(production) database. Separate, blank databases are created for the tests.

Regardless of whether the tests pass or fail, the test databases are destroyed
when all the tests have been executed.

You can prevent the test databases from being destroyed by adding the
``--keepdb`` flag to the test command. This will preserve the test
database between runs. If the database does not exist, it will first
be created. Any migrations will also be applied in order to keep it
up to date.

By default the test databases get their names by prepending ``test_``
to the value of the ``NAME`` settings for the databases
defined in ``DATABASES``. When using the SQLite database engine
the tests will by default use an in-memory database (i.e., the
database will be created in memory, bypassing the filesystem
entirely!). If you want to use a different database name, specify
``NAME <TEST_NAME>`` in the ``TEST <DATABASE-TEST>``
dictionary for any given database in ``DATABASES``.

On PostgreSQL, ``USER`` will also need read access to the built-in
``postgres`` database.

Aside from using a separate database, the test runner will otherwise
use all of the same database settings you have in your settings file:
``ENGINE <DATABASE-ENGINE>``, ``USER``, ``HOST``, etc. The
test database is created by the user specified by ``USER``, so you'll
need to make sure that the given user account has sufficient privileges to
create a new database on the system.

For fine-grained control over the character encoding of your test
database, use the ``CHARSET <TEST_CHARSET>`` TEST option. If you're using
MySQL, you can also use the ``COLLATION <TEST_COLLATION>`` option to
control the particular collation used by the test database. See the
settings documentation  for details of these
and other advanced settings.

If using a SQLite in-memory database with Python 3.4+ and SQLite 3.7.13+,
`shared cache <https://www.sqlite.org/sharedcache.html>`_ will be enabled, so
you can write tests with ability to share the database between threads.

.. admonition:: Finding data from your production database when running tests?

    If your code attempts to access the database when its modules are compiled,
    this will occur *before* the test database is set up, with potentially
    unexpected results. For example, if you have a database query in
    module-level code and a real database exists, production data could pollute
    your tests. *It is a bad idea to have such import-time database queries in
    your code* anyway - rewrite your code so that it doesn't do this.

    This also applies to customized implementations of
    :meth:`~django.apps.AppConfig.ready()`.

.. _order-of-tests:

Order in which tests are executed
---------------------------------

In order to guarantee that all ``TestCase`` code starts with a clean database,
the Django test runner reorders tests in the following way:

* All :class:`~django.test.TestCase` subclasses are run first.

* Then, all other Django-based tests (test cases based on
  :class:`~django.test.SimpleTestCase`, including
  :class:`~django.test.TransactionTestCase`) are run with no particular
  ordering guaranteed nor enforced among them.

* Then any other :class:`unittest.TestCase` tests (including doctests) that may
  alter the database without restoring it to its original state are run.

.. note::

    The new ordering of tests may reveal unexpected dependencies on test case
    ordering. This is the case with doctests that relied on state left in the
    database by a given :class:`~django.test.TransactionTestCase` test, they
    must be updated to be able to run independently.

    You may reverse the execution order inside groups by passing
    ``--reverse`` to the test command. This can help with ensuring
    your tests are independent from each other.

.. _test-case-serialized-rollback:

Rollback emulation
------------------

Any initial data loaded in migrations will only be available in ``TestCase``
tests and not in ``TransactionTestCase`` tests, and additionally only on
backends where transactions are supported (the most important exception being
MyISAM). This is also true for tests which rely on ``TransactionTestCase``
such as :class:`LiveServerTestCase` and
:class:`~django.contrib.staticfiles.testing.StaticLiveServerTestCase`.

Django can reload that data for you on a per-testcase basis by
setting the ``serialized_rollback`` option to ``True`` in the body of the
``TestCase`` or ``TransactionTestCase``, but note that this will slow down
that test suite by approximately 3x.

Third-party apps or those developing against MyISAM will need to set this;
in general, however, you should be developing your own projects against a
transactional database and be using ``TestCase`` for most tests, and thus
not need this setting.

The initial serialization is usually very quick, but if you wish to exclude
some apps from this process (and speed up test runs slightly), you may add
those apps to ``TEST_NON_SERIALIZED_APPS``.

Other test conditions
---------------------

Regardless of the value of the ``DEBUG`` setting in your configuration
file, all Django tests run with ``DEBUG``\=False. This is to ensure that
the observed output of your code matches what will be seen in a production
setting.

Caches are not cleared after each test, and running "manage.py test fooapp" can
insert data from the tests into the cache of a live system if you run your
tests in production because, unlike databases, a separate "test cache" is not
used. This behavior `may change`_ in the future.

.. _may change: https://code.djangoproject.com/ticket/11505

Understanding the test output
-----------------------------

When you run your tests, you'll see a number of messages as the test runner
prepares itself. You can control the level of detail of these messages with the
``verbosity`` option on the command line::

    Creating test database...
    Creating table myapp_animal
    Creating table myapp_mineral

This tells you that the test runner is creating a test database, as described
in the previous section.

Once the test database has been created, Django will run your tests.
If everything goes well, you'll see something like this::

    ----------------------------------------------------------------------
    Ran 22 tests in 0.221s

    OK

If there are test failures, however, you'll see full details about which tests
failed::

    ======================================================================
    FAIL: test_was_published_recently_with_future_poll (polls.tests.PollMethodTests)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "/dev/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_poll
        self.assertEqual(future_poll.was_published_recently(), False)
    AssertionError: True != False

    ----------------------------------------------------------------------
    Ran 1 test in 0.003s

    FAILED (failures=1)

A full explanation of this error output is beyond the scope of this document,
but it's pretty intuitive. You can consult the documentation of Python's
:mod:`unittest` library for details.

Note that the return code for the test-runner script is 1 for any number of
failed and erroneous tests. If all the tests pass, the return code is 0. This
feature is useful if you're using the test-runner script in a shell script and
need to test for success or failure at that level.

Speeding up the tests
---------------------

In recent versions of Django, the default password hasher is rather slow by
design. If during your tests you are authenticating many users, you may want
to use a custom settings file and set the ``PASSWORD_HASHERS`` setting
to a faster hashing algorithm::

    PASSWORD_HASHERS = [
        'django.contrib.auth.hashers.MD5PasswordHasher',
    ]

Don't forget to also include in ``PASSWORD_HASHERS`` any hashing
algorithm used in fixtures, if any.

Testing tools
=============

.. currentmodule:: django.test

Django provides a small set of tools that come in handy when writing tests.

.. _test-client:

The test client
---------------

The test client is a Python class that acts as a dummy Web browser, allowing
you to test your views and interact with your Django-powered application
programmatically.

Some of the things you can do with the test client are:

* Simulate GET and POST requests on a URL and observe the response --
  everything from low-level HTTP (result headers and status codes) to
  page content.

* See the chain of redirects (if any) and check the URL and status code at
  each step.

* Test that a given request is rendered by a given Django template, with
  a template context that contains certain values.

Note that the test client is not intended to be a replacement for Selenium_ or
other "in-browser" frameworks. Django's test client has a different focus. In
short:

* Use Django's test client to establish that the correct template is being
  rendered and that the template is passed the correct context data.

* Use in-browser frameworks like Selenium_ to test *rendered* HTML and the
  *behavior* of Web pages, namely JavaScript functionality. Django also
  provides special support for those frameworks; see the section on
  :class:`~django.test.LiveServerTestCase` for more details.

A comprehensive test suite should use a combination of both test types.

Overview and a quick example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To use the test client, instantiate ``django.test.Client`` and retrieve
Web pages::

    >>> from django.test import Client
    >>> c = Client()
    >>> response = c.post('/login/', {'username': 'john', 'password': 'smith'})
    >>> response.status_code
    200
    >>> response = c.get('/customer/details/')
    >>> response.content
    '<!DOCTYPE html...'

As this example suggests, you can instantiate ``Client`` from within a session
of the Python interactive interpreter.

Note a few important things about how the test client works:

* The test client does *not* require the Web server to be running. In fact,
  it will run just fine with no Web server running at all! That's because
  it avoids the overhead of HTTP and deals directly with the Django
  framework. This helps make the unit tests run quickly.

* When retrieving pages, remember to specify the *path* of the URL, not the
  whole domain. For example, this is correct::

      >>> c.get('/login/')

  This is incorrect::

      >>> c.get('http://www.example.com/login/')

  The test client is not capable of retrieving Web pages that are not
  powered by your Django project. If you need to retrieve other Web pages,
  use a Python standard library module such as :mod:`urllib`.

* To resolve URLs, the test client uses whatever URLconf is pointed-to by
  your ``ROOT_URLCONF`` setting.

* Although the above example would work in the Python interactive
  interpreter, some of the test client's functionality, notably the
  template-related functionality, is only available *while tests are
  running*.

  The reason for this is that Django's test runner performs a bit of black
  magic in order to determine which template was loaded by a given view.
  This black magic (essentially a patching of Django's template system in
  memory) only happens during test running.

* By default, the test client will disable any CSRF checks
  performed by your site.

  If, for some reason, you *want* the test client to perform CSRF
  checks, you can create an instance of the test client that
  enforces CSRF checks. To do this, pass in the
  ``enforce_csrf_checks`` argument when you construct your
  client::

      >>> from django.test import Client
      >>> csrf_client = Client(enforce_csrf_checks=True)

Making requests
~~~~~~~~~~~~~~~

Use the ``django.test.Client`` class to make requests.

.. class:: Client(enforce_csrf_checks=False, \*\*defaults)

    It requires no arguments at time of construction. However, you can use
    keywords arguments to specify some default headers. For example, this will
    send a ``User-Agent`` HTTP header in each request::

        >>> c = Client(HTTP_USER_AGENT='Mozilla/5.0')

    The values from the ``extra`` keywords arguments passed to
    :meth:`~django.test.Client.get()`,
    :meth:`~django.test.Client.post()`, etc. have precedence over
    the defaults passed to the class constructor.

    The ``enforce_csrf_checks`` argument can be used to test CSRF
    protection (see above).

    Once you have a ``Client`` instance, you can call any of the following
    methods:

    .. method:: Client.get(path, data=None, follow=False, secure=False, \*\*extra)

        Makes a GET request on the provided ``path`` and returns a ``Response``
        object, which is documented below.

        The key-value pairs in the ``data`` dictionary are used to create a GET
        data payload. For example::

            >>> c = Client()
            >>> c.get('/customers/details/', {'name': 'fred', 'age': 7})

        ...will result in the evaluation of a GET request equivalent to::

            /customers/details/?name=fred&age=7

        The ``extra`` keyword arguments parameter can be used to specify
        headers to be sent in the request. For example::

            >>> c = Client()
            >>> c.get('/customers/details/', {'name': 'fred', 'age': 7},
            ...       HTTP_X_REQUESTED_WITH='XMLHttpRequest')

        ...will send the HTTP header ``HTTP_X_REQUESTED_WITH`` to the
        details view, which is a good way to test code paths that use the
        :meth:`django.http.HttpRequest.is_ajax()` method.

        .. admonition:: CGI specification

            The headers sent via ``**extra`` should follow CGI_ specification.
            For example, emulating a different "Host" header as sent in the
            HTTP request from the browser to the server should be passed
            as ``HTTP_HOST``.

            .. _CGI: http://www.w3.org/CGI/

        If you already have the GET arguments in URL-encoded form, you can
        use that encoding instead of using the data argument. For example,
        the previous GET request could also be posed as::

        >>> c = Client()
        >>> c.get('/customers/details/?name=fred&age=7')

        If you provide a URL with both an encoded GET data and a data argument,
        the data argument will take precedence.

        If you set ``follow`` to ``True`` the client will follow any redirects
        and a ``redirect_chain`` attribute will be set in the response object
        containing tuples of the intermediate urls and status codes.

        If you had a URL ``/redirect_me/`` that redirected to ``/next/``, that
        redirected to ``/final/``, this is what you'd see::

            >>> response = c.get('/redirect_me/', follow=True)
            >>> response.redirect_chain
            [('http://testserver/next/', 302), ('http://testserver/final/', 302)]

        If you set ``secure`` to ``True`` the client will emulate an HTTPS
        request.

    .. method:: Client.post(path, data=None, content_type=MULTIPART_CONTENT, follow=False, secure=False, \*\*extra)

        Makes a POST request on the provided ``path`` and returns a
        ``Response`` object, which is documented below.

        The key-value pairs in the ``data`` dictionary are used to submit POST
        data. For example::

            >>> c = Client()
            >>> c.post('/login/', {'name': 'fred', 'passwd': 'secret'})

        ...will result in the evaluation of a POST request to this URL::

            /login/

        ...with this POST data::

            name=fred&passwd=secret

        If you provide ``content_type`` (e.g. :mimetype:`text/xml` for an XML
        payload), the contents of ``data`` will be sent as-is in the POST
        request, using ``content_type`` in the HTTP ``Content-Type`` header.

        If you don't provide a value for ``content_type``, the values in
        ``data`` will be transmitted with a content type of
        :mimetype:`multipart/form-data`. In this case, the key-value pairs in
        ``data`` will be encoded as a multipart message and used to create the
        POST data payload.

        To submit multiple values for a given key -- for example, to specify
        the selections for a ``<select multiple>`` -- provide the values as a
        list or tuple for the required key. For example, this value of ``data``
        would submit three selected values for the field named ``choices``::

            {'choices': ('a', 'b', 'd')}

        Submitting files is a special case. To POST a file, you need only
        provide the file field name as a key, and a file handle to the file you
        wish to upload as a value. For example::

            >>> c = Client()
            >>> with open('wishlist.doc') as fp:
            ...     c.post('/customers/wishes/', {'name': 'fred', 'attachment': fp})

        (The name ``attachment`` here is not relevant; use whatever name your
        file-processing code expects.)

        You may also provide any file-like object (e.g., :class:`~io.StringIO` or
        :class:`~io.BytesIO`) as a file handle.

        Note that if you wish to use the same file handle for multiple
        ``post()`` calls then you will need to manually reset the file
        pointer between posts. The easiest way to do this is to
        manually close the file after it has been provided to
        ``post()``, as demonstrated above.

        You should also ensure that the file is opened in a way that
        allows the data to be read. If your file contains binary data
        such as an image, this means you will need to open the file in
        ``rb`` (read binary) mode.

        The ``extra`` argument acts the same as for :meth:`Client.get`.

        If the URL you request with a POST contains encoded parameters, these
        parameters will be made available in the request.GET data. For example,
        if you were to make the request::

        >>> c.post('/login/?visitor=true', {'name': 'fred', 'passwd': 'secret'})

        ... the view handling this request could interrogate request.POST
        to retrieve the username and password, and could interrogate request.GET
        to determine if the user was a visitor.

        If you set ``follow`` to ``True`` the client will follow any redirects
        and a ``redirect_chain`` attribute will be set in the response object
        containing tuples of the intermediate urls and status codes.

        If you set ``secure`` to ``True`` the client will emulate an HTTPS
        request.

    .. method:: Client.head(path, data=None, follow=False, secure=False, \*\*extra)

        Makes a HEAD request on the provided ``path`` and returns a
        ``Response`` object. This method works just like :meth:`Client.get`,
        including the ``follow``, ``secure`` and ``extra`` arguments, except
        it does not return a message body.

    .. method:: Client.options(path, data='', content_type='application/octet-stream', follow=False, secure=False, \*\*extra)

        Makes an OPTIONS request on the provided ``path`` and returns a
        ``Response`` object. Useful for testing RESTful interfaces.

        When ``data`` is provided, it is used as the request body, and
        a ``Content-Type`` header is set to ``content_type``.

        The ``follow``, ``secure`` and ``extra`` arguments act the same as for
        :meth:`Client.get`.

    .. method:: Client.put(path, data='', content_type='application/octet-stream', follow=False, secure=False, \*\*extra)

        Makes a PUT request on the provided ``path`` and returns a
        ``Response`` object. Useful for testing RESTful interfaces.

        When ``data`` is provided, it is used as the request body, and
        a ``Content-Type`` header is set to ``content_type``.

        The ``follow``, ``secure`` and ``extra`` arguments act the same as for
        :meth:`Client.get`.

    .. method:: Client.patch(path, data='', content_type='application/octet-stream', follow=False, secure=False, \*\*extra)

        Makes a PATCH request on the provided ``path`` and returns a
        ``Response`` object. Useful for testing RESTful interfaces.

        The ``follow``, ``secure`` and ``extra`` arguments act the same as for
        :meth:`Client.get`.

    .. method:: Client.delete(path, data='', content_type='application/octet-stream', follow=False, secure=False, \*\*extra)

        Makes an DELETE request on the provided ``path`` and returns a
        ``Response`` object. Useful for testing RESTful interfaces.

        When ``data`` is provided, it is used as the request body, and
        a ``Content-Type`` header is set to ``content_type``.

        The ``follow``, ``secure`` and ``extra`` arguments act the same as for
        :meth:`Client.get`.

    .. method:: Client.trace(path, follow=False, secure=False, \*\*extra)

        Makes a TRACE request on the provided ``path`` and returns a
        ``Response`` object. Useful for simulating diagnostic probes.

        Unlike the other request methods, ``data`` is not provided as a keyword
        parameter in order to comply with :rfc:`2616`, which mandates that
        TRACE requests should not have an entity-body.

        The ``follow``, ``secure``, and ``extra`` arguments act the same as for
        :meth:`Client.get`.

    .. method:: Client.login(\*\*credentials)

        If your site uses Django's authentication system
        and you deal with logging in users, you can use the test client's
        ``login()`` method to simulate the effect of a user logging into the
        site.

        After you call this method, the test client will have all the cookies
        and session data required to pass any login-based tests that may form
        part of a view.

        The format of the ``credentials`` argument depends on which
        authentication backend you're using
        (which is configured by your ``AUTHENTICATION_BACKENDS``
        setting). If you're using the standard authentication backend provided
        by Django (``ModelBackend``), ``credentials`` should be the user's
        username and password, provided as keyword arguments::

            >>> c = Client()
            >>> c.login(username='fred', password='secret')

            # Now you can access a view that's only available to logged-in users.

        If you're using a different authentication backend, this method may
        require different credentials. It requires whichever credentials are
        required by your backend's ``authenticate()`` method.

        ``login()`` returns ``True`` if it the credentials were accepted and
        login was successful.

        Finally, you'll need to remember to create user accounts before you can
        use this method. As we explained above, the test runner is executed
        using a test database, which contains no users by default. As a result,
        user accounts that are valid on your production site will not work
        under test conditions. You'll need to create users as part of the test
        suite -- either manually (using the Django model API) or with a test
        fixture. Remember that if you want your test user to have a password,
        you can't set the user's password by setting the password attribute
        directly -- you must use the
        :meth:`~django.contrib.auth.models.User.set_password()` function to
        store a correctly hashed password. Alternatively, you can use the
        :meth:`~django.contrib.auth.models.UserManager.create_user` helper
        method to create a new user with a correctly hashed password.

    .. method:: Client.logout()

        If your site uses Django's authentication system,
        the ``logout()`` method can be used to simulate the effect of a user
        logging out of your site.

        After you call this method, the test client will have all the cookies
        and session data cleared to defaults. Subsequent requests will appear
        to come from an :class:`~django.contrib.auth.models.AnonymousUser`.

Testing responses
~~~~~~~~~~~~~~~~~

The ``get()`` and ``post()`` methods both return a ``Response`` object. This
``Response`` object is *not* the same as the ``HttpResponse`` object returned
by Django views; the test response object has some additional data useful for
test code to verify.

Specifically, a ``Response`` object has the following attributes:

.. class:: Response()

    .. attribute:: client

        The test client that was used to make the request that resulted in the
        response.

    .. attribute:: content

        The body of the response, as a string. This is the final page content as
        rendered by the view, or any error message.

    .. attribute:: context

        The template ``Context`` instance that was used to render the template that
        produced the response content.

        If the rendered page used multiple templates, then ``context`` will be a
        list of ``Context`` objects, in the order in which they were rendered.

        Regardless of the number of templates used during rendering, you can
        retrieve context values using the ``[]`` operator. For example, the
        context variable ``name`` could be retrieved using::

            >>> response = client.get('/foo/')
            >>> response.context['name']
            'Arthur'

    .. attribute:: request

        The request data that stimulated the response.

    .. attribute:: wsgi_request

        The ``WSGIRequest`` instance generated by the test handler that
        generated the response.

    .. attribute:: status_code

        The HTTP status of the response, as an integer. See
        :rfc:`2616#section-10` for a full list of HTTP status codes.

    .. attribute:: templates

        A list of ``Template`` instances used to render the final content, in
        the order they were rendered. For each template in the list, use
        ``template.name`` to get the template's file name, if the template was
        loaded from a file. (The name is a string such as
        ``'admin/index.html'``.)

    .. attribute:: resolver_match

        An instance of :class:`~django.core.urlresolvers.ResolverMatch` for the
        response. You can use the
        :attr:`~django.core.urlresolvers.ResolverMatch.func` attribute, for
        example, to verify the view that served the response::

            # my_view here is a function based view
            self.assertEqual(response.resolver_match.func, my_view)

            # class based views need to be compared by name, as the functions
            # generated by as_view() won't be equal
            self.assertEqual(response.resolver_match.func.__name__, MyView.as_view().__name__)

        If the given URL is not found, accessing this attribute will raise a
        :exc:`~django.core.urlresolvers.Resolver404` exception.

You can also use dictionary syntax on the response object to query the value
of any settings in the HTTP headers. For example, you could determine the
content type of a response using ``response['Content-Type']``.

Exceptions
~~~~~~~~~~

If you point the test client at a view that raises an exception, that exception
will be visible in the test case. You can then use a standard ``try ... except``
block or :meth:`~unittest.TestCase.assertRaises` to test for exceptions.

The only exceptions that are not visible to the test client are
:class:`~django.http.Http404`,
:class:`~django.core.exceptions.PermissionDenied`, :exc:`SystemExit`, and
:class:`~django.core.exceptions.SuspiciousOperation`. Django catches these
exceptions internally and converts them into the appropriate HTTP response
codes. In these cases, you can check ``response.status_code`` in your test.

Persistent state
~~~~~~~~~~~~~~~~

The test client is stateful. If a response returns a cookie, then that cookie
will be stored in the test client and sent with all subsequent ``get()`` and
``post()`` requests.

Expiration policies for these cookies are not followed. If you want a cookie
to expire, either delete it manually or create a new ``Client`` instance (which
will effectively delete all cookies).

A test client has two attributes that store persistent state information. You
can access these properties as part of a test condition.

.. attribute:: Client.cookies

    A Python :class:`~http.cookies.SimpleCookie` object, containing the current
    values of all the client cookies. See the documentation of the
    :mod:`http.cookies` module for more.

.. attribute:: Client.session

    A dictionary-like object containing session information. See the
    session documentation for full details.

    To modify the session and then save it, it must be stored in a variable
    first (because a new ``SessionStore`` is created every time this property
    is accessed)::

        def test_something(self):
            session = self.client.session
            session['somekey'] = 'test'
            session.save()

Example
~~~~~~~

The following is a simple unit test using the test client::

    import unittest
    from django.test import Client

    class SimpleTest(unittest.TestCase):
        def setUp(self):
            # Every test needs a client.
            self.client = Client()

        def test_details(self):
            # Issue a GET request.
            response = self.client.get('/customer/details/')

            # Check that the response is 200 OK.
            self.assertEqual(response.status_code, 200)

            # Check that the rendered context contains 5 customers.
            self.assertEqual(len(response.context['customers']), 5)

.. _django-testcase-subclasses:

Provided test case classes
--------------------------

Normal Python unit test classes extend a base class of
:class:`unittest.TestCase`. Django provides a few extensions of this base class:

.. _testcase_hierarchy_diagram:

.. figure:: _images/django_unittest_classes_hierarchy.*
   :alt: Hierarchy of Django unit testing classes (TestCase subclasses)
   :width: 508
   :height: 328

   Hierarchy of Django unit testing classes

SimpleTestCase
~~~~~~~~~~~~~~

.. class:: SimpleTestCase()

A thin subclass of :class:`unittest.TestCase`, it extends it with some basic
functionality like:

* Saving and restoring the Python warning machinery state.
* Some useful assertions like:

  * Checking that a callable :meth:`raises a certain exception
    <SimpleTestCase.assertRaisesMessage>`.
  * Testing form field :meth:`rendering and error treatment
    <SimpleTestCase.assertFieldOutput>`.
  * Testing :meth:`HTML responses for the presence/lack of a given fragment
    <SimpleTestCase.assertContains>`.
  * Verifying that a template :meth:`has/hasn't been used to generate a given
    response content <SimpleTestCase.assertTemplateUsed>`.
  * Verifying a HTTP :meth:`redirect <SimpleTestCase.assertRedirects>` is
    performed by the app.
  * Robustly testing two :meth:`HTML fragments <SimpleTestCase.assertHTMLEqual>`
    for equality/inequality or :meth:`containment <SimpleTestCase.assertInHTML>`.
  * Robustly testing two :meth:`XML fragments <SimpleTestCase.assertXMLEqual>`
    for equality/inequality.
  * Robustly testing two :meth:`JSON fragments <SimpleTestCase.assertJSONEqual>`
    for equality.

* The ability to run tests with modified settings.
* Using the :attr:`~SimpleTestCase.client` :class:`~django.test.Client`.
* Custom test-time :attr:`URL maps <SimpleTestCase.urls>`.

If you need any of the other more complex and heavyweight Django-specific
features like:

* Testing or using the ORM.
* Database :attr:`~TransactionTestCase.fixtures`.
* Test skipping based on database backend features.
* The remaining specialized :meth:`assert*
  <TransactionTestCase.assertQuerysetEqual>` methods.

then you should use :class:`~django.test.TransactionTestCase` or
:class:`~django.test.TestCase` instead.

``SimpleTestCase`` inherits from ``unittest.TestCase``.

.. warning::

    ``SimpleTestCase`` and its subclasses (e.g. ``TestCase``, ...) rely on
    ``setUpClass()`` and ``tearDownClass()`` to perform some class-wide
    initialization (e.g. overriding settings). If you need to override those
    methods, don't forget to call the ``super`` implementation::

        class MyTestCase(TestCase):

            @classmethod
            def setUpClass(cls):
                super(cls, MyTestCase).setUpClass()     # Call parent first
                ...

            @classmethod
            def tearDownClass(cls):
                ...
                super(cls, MyTestCase).tearDownClass()  # Call parent last

TransactionTestCase
~~~~~~~~~~~~~~~~~~~

.. class:: TransactionTestCase()

Django's ``TestCase`` class (described below) makes use of database transaction
facilities to speed up the process of resetting the database to a known state
at the beginning of each test. A consequence of this, however, is that some
database behaviors cannot be tested within a Django ``TestCase`` class. For
instance, you cannot test that a block of code is executing within a
transaction, as is required when using
:meth:`~django.db.models.query.QuerySet.select_for_update()`. In those cases,
you should use ``TransactionTestCase``.

In older versions of Django, the effects of transaction commit and rollback
could not be tested within a ``TestCase``.  With the completion of the
deprecation cycle of the old-style transaction management in Django 1.8,
transaction management commands (e.g. ``transaction.commit()``) are no
longer disabled within ``TestCase``.

``TransactionTestCase`` and ``TestCase`` are identical except for the manner
in which the database is reset to a known state and the ability for test code
to test the effects of commit and rollback:

* A ``TransactionTestCase`` resets the database after the test runs by
  truncating all tables. A ``TransactionTestCase`` may call commit and rollback
  and observe the effects of these calls on the database.

* A ``TestCase``, on the other hand, does not truncate tables after a test.
  Instead, it encloses the test code in a database transaction that is rolled
  back at the end of the test. This guarantees that the rollback at the end of
  the test restores the database to its initial state.

.. warning::

  ``TestCase`` running on a database that does not support rollback (e.g. MySQL
  with the MyISAM storage engine), and all instances of ``TransactionTestCase``,
  will roll back at the end of the test by deleting all data from the test
  database.

  Apps will not see their data reloaded;
  if you need this functionality (for example, third-party apps should enable
  this) you can set ``serialized_rollback = True`` inside the
  ``TestCase`` body.

``TransactionTestCase`` inherits from :class:`~django.test.SimpleTestCase`.

TestCase
~~~~~~~~

.. class:: TestCase()

This class provides some additional capabilities that can be useful for testing
Web sites.

Converting a normal :class:`unittest.TestCase` to a Django :class:`TestCase` is
easy: Just change the base class of your test from ``'unittest.TestCase'`` to
``'django.test.TestCase'``. All of the standard Python unit test functionality
will continue to be available, but it will be augmented with some useful
additions, including:

* Automatic loading of fixtures.

* Wraps the tests within two nested ``atomic`` blocks: one for the whole class
  and one for each test.

* Creates a TestClient instance.

* Django-specific assertions for testing for things like redirection and form
  errors.

.. classmethod:: TestCase.setUpTestData()


The class-level ``atomic`` block described above allows the creation of
initial data at the class level, once for the whole ``TestCase``. This
technique allows for faster tests as compared to using ``setUp()``.

For example::

    from django.test import TestCase

    class MyTests(TestCase):
        @classmethod
        def setUpTestData(cls):
            # Set up data for the whole TestCase
            cls.foo = Foo.objects.create(bar="Test")
            ...

        def test1(self):
            # Some test using self.foo
            ...

        def test2(self):
            # Some other test using self.foo
            ...

Note that if the tests are run on a database with no transaction support
(for instance, MySQL with the MyISAM engine), ``setUpTestData()`` will be
called before each test, negating the speed benefits.

.. warning::

    If you want to test some specific database transaction behavior, you should
    use ``TransactionTestCase``, as ``TestCase`` wraps test execution within an
    :func:`~django.db.transaction.atomic()` block.

``TestCase`` inherits from :class:`~django.test.TransactionTestCase`.

.. _live-test-server:

LiveServerTestCase
~~~~~~~~~~~~~~~~~~

.. class:: LiveServerTestCase()

``LiveServerTestCase`` does basically the same as
:class:`~django.test.TransactionTestCase` with one extra feature: it launches a
live Django server in the background on setup, and shuts it down on teardown.
This allows the use of automated test clients other than the
Django dummy client such as, for example, the Selenium_
client, to execute a series of functional tests inside a browser and simulate a
real user's actions.

By default the live server's address is ``'localhost:8081'`` and the full URL
can be accessed during the tests with ``self.live_server_url``. If you'd like
to change the default address (in the case, for example, where the 8081 port is
already taken) then you may pass a different one to the test command
via the ``--liveserver`` option, for example:

.. code-block:: bash

    ./manage.py test --liveserver=localhost:8082

Another way of changing the default server address is by setting the
`DJANGO_LIVE_TEST_SERVER_ADDRESS` environment variable somewhere in your
code (for example, in a custom test runner)::

    import os
    os.environ['DJANGO_LIVE_TEST_SERVER_ADDRESS'] = 'localhost:8082'

In the case where the tests are run by multiple processes in parallel (for
example, in the context of several simultaneous `continuous integration`_
builds), the processes will compete for the same address, and therefore your
tests might randomly fail with an "Address already in use" error. To avoid this
problem, you can pass a comma-separated list of ports or ranges of ports (at
least as many as the number of potential parallel processes). For example:

.. code-block:: bash

    ./manage.py test --liveserver=localhost:8082,8090-8100,9000-9200,7041

Then, during test execution, each new live test server will try every specified
port until it finds one that is free and takes it.

.. _continuous integration: http://en.wikipedia.org/wiki/Continuous_integration

To demonstrate how to use ``LiveServerTestCase``, let's write a simple Selenium
test. First of all, you need to install the `selenium package`_ into your
Python path:

.. code-block:: bash

   pip install selenium

Then, add a ``LiveServerTestCase``-based test to your app's tests module
(for example: ``myapp/tests.py``). The code for this test may look as follows::

    from django.test import LiveServerTestCase
    from selenium.webdriver.firefox.webdriver import WebDriver

    class MySeleniumTests(LiveServerTestCase):
        fixtures = ['user-data.json']

        @classmethod
        def setUpClass(cls):
            super(MySeleniumTests, cls).setUpClass()
            cls.selenium = WebDriver()

        @classmethod
        def tearDownClass(cls):
            cls.selenium.quit()
            super(MySeleniumTests, cls).tearDownClass()

        def test_login(self):
            self.selenium.get('%s%s' % (self.live_server_url, '/login/'))
            username_input = self.selenium.find_element_by_name("username")
            username_input.send_keys('myuser')
            password_input = self.selenium.find_element_by_name("password")
            password_input.send_keys('secret')
            self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()

Finally, you may run the test as follows:

.. code-block:: bash

    ./manage.py test myapp.tests.MySeleniumTests.test_login

This example will automatically open Firefox then go to the login page, enter
the credentials and press the "Log in" button. Selenium offers other drivers in
case you do not have Firefox installed or wish to use another browser. The
example above is just a tiny fraction of what the Selenium client can do; check
out the `full reference`_ for more details.

.. _Selenium: http://seleniumhq.org/
.. _selenium package: https://pypi.python.org/pypi/selenium
.. _full reference: http://selenium-python.readthedocs.org/en/latest/api.html
.. _Firefox: http://www.mozilla.com/firefox/

.. tip::

    If you use the :mod:`~django.contrib.staticfiles` app in your project and
    need to perform live testing, then you might want to use the
    :class:`~django.contrib.staticfiles.testing.StaticLiveServerTestCase`
    subclass which transparently serves all the assets during execution of
    its tests in a way very similar to what we get at development time with
    ``DEBUG=True``, i.e. without having to collect them using
    ``collectstatic``.

.. note::

    When using an in-memory SQLite database to run the tests, the same database
    connection will be shared by two threads in parallel: the thread in which
    the live server is run and the thread in which the test case is run. It's
    important to prevent simultaneous database queries via this shared
    connection by the two threads, as that may sometimes randomly cause the
    tests to fail. So you need to ensure that the two threads don't access the
    database at the same time. In particular, this means that in some cases
    (for example, just after clicking a link or submitting a form), you might
    need to check that a response is received by Selenium and that the next
    page is loaded before proceeding with further test execution.
    Do this, for example, by making Selenium wait until the ``<body>`` HTML tag
    is found in the response (requires Selenium > 2.13)::

        def test_login(self):
            from selenium.webdriver.support.wait import WebDriverWait
            timeout = 2
            ...
            self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()
            # Wait until the response is received
            WebDriverWait(self.selenium, timeout).until(
                lambda driver: driver.find_element_by_tag_name('body'))

    The tricky thing here is that there's really no such thing as a "page load,"
    especially in modern Web apps that generate HTML dynamically after the
    server generates the initial document. So, simply checking for the presence
    of ``<body>`` in the response might not necessarily be appropriate for all
    use cases. Please refer to the `Selenium FAQ`_ and
    `Selenium documentation`_ for more information.

    .. _Selenium FAQ: http://code.google.com/p/selenium/wiki/FrequentlyAskedQuestions#Q:_WebDriver_fails_to_find_elements_/_Does_not_block_on_page_loa
    .. _Selenium documentation: http://seleniumhq.org/docs/04_webdriver_advanced.html#explicit-waits

Test cases features
-------------------

Default test client
~~~~~~~~~~~~~~~~~~~

.. attribute:: SimpleTestCase.client

Every test case in a ``django.test.*TestCase`` instance has access to an
instance of a Django test client. This client can be accessed as
``self.client``. This client is recreated for each test, so you don't have to
worry about state (such as cookies) carrying over from one test to another.

This means, instead of instantiating a ``Client`` in each test::

    import unittest
    from django.test import Client

    class SimpleTest(unittest.TestCase):
        def test_details(self):
            client = Client()
            response = client.get('/customer/details/')
            self.assertEqual(response.status_code, 200)

        def test_index(self):
            client = Client()
            response = client.get('/customer/index/')
            self.assertEqual(response.status_code, 200)

...you can just refer to ``self.client``, like so::

    from django.test import TestCase

    class SimpleTest(TestCase):
        def test_details(self):
            response = self.client.get('/customer/details/')
            self.assertEqual(response.status_code, 200)

        def test_index(self):
            response = self.client.get('/customer/index/')
            self.assertEqual(response.status_code, 200)

Customizing the test client
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. attribute:: SimpleTestCase.client_class

If you want to use a different ``Client`` class (for example, a subclass
with customized behavior), use the :attr:`~SimpleTestCase.client_class` class
attribute::

    from django.test import TestCase, Client

    class MyTestClient(Client):
        # Specialized methods for your environment
        ...

    class MyTest(TestCase):
        client_class = MyTestClient

        def test_my_stuff(self):
            # Here self.client is an instance of MyTestClient...
            call_some_test_code()

.. _topics-testing-fixtures:

Fixture loading
~~~~~~~~~~~~~~~

.. attribute:: TransactionTestCase.fixtures

A test case for a database-backed Web site isn't much use if there isn't any
data in the database. To make it easy to put test data into the database,
Django's custom ``TransactionTestCase`` class provides a way of loading
**fixtures**.

A fixture is a collection of data that Django knows how to import into a
database. For example, if your site has user accounts, you might set up a
fixture of fake user accounts in order to populate your database during tests.

The most straightforward way of creating a fixture is to use the
manage.py dumpdata  command. This assumes you
already have some data in your database. See the dumpdata
documentation for more details.

Once you've created a fixture and placed it in a ``fixtures`` directory in one
of your ``INSTALLED_APPS``, you can use it in your unit tests by
specifying a ``fixtures`` class attribute on your :class:`django.test.TestCase`
subclass::

    from django.test import TestCase
    from myapp.models import Animal

    class AnimalTestCase(TestCase):
        fixtures = ['mammals.json', 'birds']

        def setUp(self):
            # Test definitions as before.
            call_setup_methods()

        def testFluffyAnimals(self):
            # A test that uses the fixtures.
            call_some_test_code()

Here's specifically what will happen:

* At the start of each test case, before ``setUp()`` is run, Django will
  flush the database, returning the database to the state it was in
  directly after ``migrate`` was called.

* Then, all the named fixtures are installed. In this example, Django will
  install any JSON fixture named ``mammals``, followed by any fixture named
  ``birds``. See the ``loaddata`` documentation for more
  details on defining and installing fixtures.

This flush/load procedure is repeated for each test in the test case, so you
can be certain that the outcome of a test will not be affected by another test,
or by the order of test execution.

By default, fixtures are only loaded into the ``default`` database. If you are
using multiple databases and set :attr:`multi_db=True
<TransactionTestCase.multi_db>`, fixtures will be loaded into all databases.

URLconf configuration
~~~~~~~~~~~~~~~~~~~~~

.. attribute:: SimpleTestCase.urls

.. deprecated:: 1.8

    Use ``@override_settings(ROOT_URLCONF=...)`` instead for URLconf
    configuration.

If your application provides views, you may want to include tests that use the
test client to exercise those views. However, an end user is free to deploy the
views in your application at any URL of their choosing. This means that your
tests can't rely upon the fact that your views will be available at a
particular URL.

In order to provide a reliable URL space for your test,
``django.test.*TestCase`` classes provide the ability to customize the URLconf
configuration for the duration of the execution of a test suite. If your
``*TestCase`` instance defines an ``urls`` attribute, the ``*TestCase`` will use
the value of that attribute as the ``ROOT_URLCONF`` for the duration
of that test.

For example::

    from django.test import TestCase

    class TestMyViews(TestCase):
        urls = 'myapp.test_urls'

        def testIndexPageView(self):
            # Here you'd test your view using ``Client``.
            call_some_test_code()

This test case will use the contents of ``myapp.test_urls`` as the
URLconf for the duration of the test case.

.. _emptying-test-outbox:

Multi-database support
~~~~~~~~~~~~~~~~~~~~~~

.. attribute:: TransactionTestCase.multi_db

Django sets up a test database corresponding to every database that is
defined in the ``DATABASES`` definition in your settings
file. However, a big part of the time taken to run a Django TestCase
is consumed by the call to ``flush`` that ensures that you have a
clean database at the start of each test run. If you have multiple
databases, multiple flushes are required (one for each database),
which can be a time consuming activity -- especially if your tests
don't need to test multi-database activity.

As an optimization, Django only flushes the ``default`` database at
the start of each test run. If your setup contains multiple databases,
and you have a test that requires every database to be clean, you can
use the ``multi_db`` attribute on the test suite to request a full
flush.

For example::

    class TestMyViews(TestCase):
        multi_db = True

        def testIndexPageView(self):
            call_some_test_code()

This test case will flush *all* the test databases before running
``testIndexPageView``.

The ``multi_db`` flag also affects into which databases the
attr:`TransactionTestCase.fixtures` are loaded. By default (when
``multi_db=False``), fixtures are only loaded into the ``default`` database.
If ``multi_db=True``, fixtures are loaded into all databases.

.. _overriding-settings:

Overriding settings
~~~~~~~~~~~~~~~~~~~

.. warning::

    Use the functions below to temporarily alter the value of settings in tests.
    Don't manipulate ``django.conf.settings`` directly as Django won't restore
    the original values after such manipulations.

.. method:: SimpleTestCase.settings()

For testing purposes it's often useful to change a setting temporarily and
revert to the original value after running the testing code. For this use case
Django provides a standard Python context manager (see :pep:`343`) called
:meth:`~django.test.SimpleTestCase.settings`, which can be used like this::

    from django.test import TestCase

    class LoginTestCase(TestCase):

        def test_login(self):

            # First check for the default behavior
            response = self.client.get('/sekrit/')
            self.assertRedirects(response, '/accounts/login/?next=/sekrit/')

            # Then override the LOGIN_URL setting
            with self.settings(LOGIN_URL='/other/login/'):
                response = self.client.get('/sekrit/')
                self.assertRedirects(response, '/other/login/?next=/sekrit/')

This example will override the ``LOGIN_URL`` setting for the code
in the ``with`` block and reset its value to the previous state afterwards.

.. method:: SimpleTestCase.modify_settings()

It can prove unwieldy to redefine settings that contain a list of values. In
practice, adding or removing values is often sufficient. The
:meth:`~django.test.SimpleTestCase.modify_settings` context manager makes it
easy::

    from django.test import TestCase

    class MiddlewareTestCase(TestCase):

        def test_cache_middleware(self):
            with self.modify_settings(MIDDLEWARE_CLASSES={
                'append': 'django.middleware.cache.FetchFromCacheMiddleware',
                'prepend': 'django.middleware.cache.UpdateCacheMiddleware',
                'remove': [
                    'django.contrib.sessions.middleware.SessionMiddleware',
                    'django.contrib.auth.middleware.AuthenticationMiddleware',
                    'django.contrib.messages.middleware.MessageMiddleware',
                ],
            }):
                response = self.client.get('/')
                # ...

For each action, you can supply either a list of values or a string. When the
value already exists in the list, ``append`` and ``prepend`` have no effect;
neither does ``remove`` when the value doesn't exist.

.. function:: override_settings

In case you want to override a setting for a test method, Django provides the
:func:`~django.test.override_settings` decorator (see :pep:`318`). It's used
like this::

    from django.test import TestCase, override_settings

    class LoginTestCase(TestCase):

        @override_settings(LOGIN_URL='/other/login/')
        def test_login(self):
            response = self.client.get('/sekrit/')
            self.assertRedirects(response, '/other/login/?next=/sekrit/')

The decorator can also be applied to :class:`~django.test.TestCase` classes::

    from django.test import TestCase, override_settings

    @override_settings(LOGIN_URL='/other/login/')
    class LoginTestCase(TestCase):

        def test_login(self):
            response = self.client.get('/sekrit/')
            self.assertRedirects(response, '/other/login/?next=/sekrit/')

.. function:: modify_settings

Likewise, Django provides the :func:`~django.test.modify_settings`
decorator::

    from django.test import TestCase, modify_settings

    class MiddlewareTestCase(TestCase):

        @modify_settings(MIDDLEWARE_CLASSES={
            'append': 'django.middleware.cache.FetchFromCacheMiddleware',
            'prepend': 'django.middleware.cache.UpdateCacheMiddleware',
        })
        def test_cache_middleware(self):
            response = self.client.get('/')
            # ...

The decorator can also be applied to test case classes::

    from django.test import TestCase, modify_settings

    @modify_settings(MIDDLEWARE_CLASSES={
        'append': 'django.middleware.cache.FetchFromCacheMiddleware',
        'prepend': 'django.middleware.cache.UpdateCacheMiddleware',
    })
    class MiddlewareTestCase(TestCase):

        def test_cache_middleware(self):
            response = self.client.get('/')
            # ...

.. note::

    When given a class, these decorators modify the class directly and return
    it; they don't create and return a modified copy of it. So if you try to
    tweak the above examples to assign the return value to a different name
    than ``LoginTestCase`` or ``MiddlewareTestCase``, you may be surprised to
    find that the original test case classes are still equally affected by the
    decorator. For a given class, :func:`~django.test.modify_settings` is
    always applied after :func:`~django.test.override_settings`.

.. warning::

    The settings file contains some settings that are only consulted during
    initialization of Django internals. If you change them with
    ``override_settings``, the setting is changed if you access it via the
    ``django.conf.settings`` module, however, Django's internals access it
    differently. Effectively, using :func:`~django.test.override_settings` or
    :func:`~django.test.modify_settings` with these settings is probably not
    going to do what you expect it to do.

    We do not recommend altering the ``DATABASES`` setting. Altering
    the ``CACHES`` setting is possible, but a bit tricky if you are
    using internals that make using of caching, like
    :mod:`django.contrib.sessions`. For example, you will have to reinitialize
    the session backend in a test that uses cached sessions and overrides
    ``CACHES``.

    Finally, avoid aliasing your settings as module-level constants as
    ``override_settings()`` won't work on such values since they are
    only evaluated the first time the module is imported.

You can also simulate the absence of a setting by deleting it after settings
have been overridden, like this::

    @override_settings()
    def test_something(self):
        del settings.LOGIN_URL
        ...

When overriding settings, make sure to handle the cases in which your app's
code uses a cache or similar feature that retains state even if the setting is
changed. Django provides the :data:`django.test.signals.setting_changed`
signal that lets you register callbacks to clean up and otherwise reset state
when settings are changed.

Django itself uses this signal to reset various data:

================================ ========================
Overridden settings              Data reset
================================ ========================
USE_TZ, TIME_ZONE                Databases timezone
TEMPLATES                        Template engines
SERIALIZATION_MODULES            Serializers cache
LOCALE_PATHS, LANGUAGE_CODE      Default translation and loaded translations
MEDIA_ROOT, DEFAULT_FILE_STORAGE Default file storage
================================ ========================

Emptying the test outbox
~~~~~~~~~~~~~~~~~~~~~~~~

If you use any of Django's custom ``TestCase`` classes, the test runner will
clear the contents of the test email outbox at the start of each test case.

For more detail on email services during tests, see `Email services`_ below.

.. _assertions:

Assertions
~~~~~~~~~~

As Python's normal :class:`unittest.TestCase` class implements assertion methods
such as :meth:`~unittest.TestCase.assertTrue` and
:meth:`~unittest.TestCase.assertEqual`, Django's custom :class:`TestCase` class
provides a number of custom assertion methods that are useful for testing Web
applications:

The failure messages given by most of these assertion methods can be customized
with the ``msg_prefix`` argument. This string will be prefixed to any failure
message generated by the assertion. This allows you to provide additional
details that may help you to identify the location and cause of an failure in
your test suite.

.. method:: SimpleTestCase.assertRaisesMessage(expected_exception, expected_message, callable_obj=None, \*args, \*\*kwargs)

    Asserts that execution of callable ``callable_obj`` raised the
    ``expected_exception`` exception and that such exception has an
    ``expected_message`` representation. Any other outcome is reported as a
    failure. Similar to unittest's :meth:`~unittest.TestCase.assertRaisesRegex`
    with the difference that ``expected_message`` isn't a regular expression.

.. method:: SimpleTestCase.assertFieldOutput(fieldclass, valid, invalid, field_args=None, field_kwargs=None, empty_value='')

    Asserts that a form field behaves correctly with various inputs.

    :param fieldclass: the class of the field to be tested.
    :param valid: a dictionary mapping valid inputs to their expected cleaned
        values.
    :param invalid: a dictionary mapping invalid inputs to one or more raised
        error messages.
    :param field_args: the args passed to instantiate the field.
    :param field_kwargs: the kwargs passed to instantiate the field.
    :param empty_value: the expected clean output for inputs in ``empty_values``.

    For example, the following code tests that an ``EmailField`` accepts
    ``a@a.com`` as a valid email address, but rejects ``aaa`` with a reasonable
    error message::

        self.assertFieldOutput(EmailField, {'a@a.com': 'a@a.com'}, {'aaa': ['Enter a valid email address.']})

.. method:: SimpleTestCase.assertFormError(response, form, field, errors, msg_prefix='')

    Asserts that a field on a form raises the provided list of errors when
    rendered on the form.

    ``form`` is the name the ``Form`` instance was given in the template
    context.

    ``field`` is the name of the field on the form to check. If ``field``
    has a value of ``None``, non-field errors (errors you can access via
    :meth:`form.non_field_errors() <django.forms.Form.non_field_errors>`) will
    be checked.

    ``errors`` is an error string, or a list of error strings, that are
    expected as a result of form validation.

.. method:: SimpleTestCase.assertFormsetError(response, formset, form_index, field, errors, msg_prefix='')

    Asserts that the ``formset`` raises the provided list of errors when
    rendered.

    ``formset`` is the name the ``Formset`` instance was given in the template
    context.

    ``form_index`` is the number of the form within the ``Formset``.  If
    ``form_index`` has a value of ``None``, non-form errors (errors you can
    access via ``formset.non_form_errors()``) will be checked.

    ``field`` is the name of the field on the form to check. If ``field``
    has a value of ``None``, non-field errors (errors you can access via
    :meth:`form.non_field_errors() <django.forms.Form.non_field_errors>`) will
    be checked.

    ``errors`` is an error string, or a list of error strings, that are
    expected as a result of form validation.

.. method:: SimpleTestCase.assertContains(response, text, count=None, status_code=200, msg_prefix='', html=False)

    Asserts that a ``Response`` instance produced the given ``status_code`` and
    that ``text`` appears in the content of the response. If ``count`` is
    provided, ``text`` must occur exactly ``count`` times in the response.

    Set ``html`` to ``True`` to handle ``text`` as HTML. The comparison with
    the response content will be based on HTML semantics instead of
    character-by-character equality. Whitespace is ignored in most cases,
    attribute ordering is not significant. See
    :meth:`~SimpleTestCase.assertHTMLEqual` for more details.

.. method:: SimpleTestCase.assertNotContains(response, text, status_code=200, msg_prefix='', html=False)

    Asserts that a ``Response`` instance produced the given ``status_code`` and
    that ``text`` does not appears in the content of the response.

    Set ``html`` to ``True`` to handle ``text`` as HTML. The comparison with
    the response content will be based on HTML semantics instead of
    character-by-character equality. Whitespace is ignored in most cases,
    attribute ordering is not significant. See
    :meth:`~SimpleTestCase.assertHTMLEqual` for more details.

.. method:: SimpleTestCase.assertTemplateUsed(response, template_name, msg_prefix='', count=None)

    Asserts that the template with the given name was used in rendering the
    response.

    The name is a string such as ``'admin/index.html'``.

    The count argument is an integer indicating the number of times the
    template should be rendered. Default is ``None``, meaning that the
    template should be rendered one or more times.

    You can use this as a context manager, like this::

        with self.assertTemplateUsed('index.html'):
            render_to_string('index.html')
        with self.assertTemplateUsed(template_name='index.html'):
            render_to_string('index.html')

.. method:: SimpleTestCase.assertTemplateNotUsed(response, template_name, msg_prefix='')

    Asserts that the template with the given name was *not* used in rendering
    the response.

    You can use this as a context manager in the same way as
    :meth:`~SimpleTestCase.assertTemplateUsed`.

.. method:: SimpleTestCase.assertRedirects(response, expected_url, status_code=302, target_status_code=200, host=None, msg_prefix='', fetch_redirect_response=True)

    Asserts that the response returned a ``status_code`` redirect status,
    redirected to ``expected_url`` (including any ``GET`` data), and that the
    final page was received with ``target_status_code``.

    If your request used the ``follow`` argument, the ``expected_url`` and
    ``target_status_code`` will be the url and status code for the final
    point of the redirect chain.

    The ``host`` argument sets a default host if ``expected_url`` doesn't
    include one (e.g. ``"/bar/"``).  If ``expected_url`` is an absolute URL that
    includes a host (e.g. ``"http://testhost/bar/"``), the ``host`` parameter
    will be ignored. Note that the test client doesn't support fetching external
    URLs, but the parameter may be useful if you are testing with a custom HTTP
    host (for example, initializing the test client with
    ``Client(HTTP_HOST="testhost")``.

    If ``fetch_redirect_response`` is ``False``, the final page won't be
    loaded. Since the test client can't fetch externals URLs, this is
    particularly useful if ``expected_url`` isn't part of your Django app.

    Scheme is handled correctly when making comparisons between two URLs. If
    there isn't any scheme specified in the location where we are redirected to,
    the original request's scheme is used. If present, the scheme in
    ``expected_url`` is the one used to make the comparisons to.

.. method:: SimpleTestCase.assertHTMLEqual(html1, html2, msg=None)

    Asserts that the strings ``html1`` and ``html2`` are equal. The comparison
    is based on HTML semantics. The comparison takes following things into
    account:

    * Whitespace before and after HTML tags is ignored.
    * All types of whitespace are considered equivalent.
    * All open tags are closed implicitly, e.g. when a surrounding tag is
      closed or the HTML document ends.
    * Empty tags are equivalent to their self-closing version.
    * The ordering of attributes of an HTML element is not significant.
    * Attributes without an argument are equal to attributes that equal in
      name and value (see the examples).

    The following examples are valid tests and don't raise any
    ``AssertionError``::

        self.assertHTMLEqual('<p>Hello <b>world!</p>',
            '''<p>
                Hello   <b>world! <b/>
            </p>''')
        self.assertHTMLEqual(
            '<input type="checkbox" checked="checked" id="id_accept_terms" />',
            '<input id="id_accept_terms" type='checkbox' checked>')

    ``html1`` and ``html2`` must be valid HTML. An ``AssertionError`` will be
    raised if one of them cannot be parsed.

    Output in case of error can be customized with the ``msg`` argument.

.. method:: SimpleTestCase.assertHTMLNotEqual(html1, html2, msg=None)

    Asserts that the strings ``html1`` and ``html2`` are *not* equal. The
    comparison is based on HTML semantics. See
    :meth:`~SimpleTestCase.assertHTMLEqual` for details.

    ``html1`` and ``html2`` must be valid HTML. An ``AssertionError`` will be
    raised if one of them cannot be parsed.

    Output in case of error can be customized with the ``msg`` argument.

.. method:: SimpleTestCase.assertXMLEqual(xml1, xml2, msg=None)

    Asserts that the strings ``xml1`` and ``xml2`` are equal. The
    comparison is based on XML semantics. Similarly to
    :meth:`~SimpleTestCase.assertHTMLEqual`, the comparison is
    made on parsed content, hence only semantic differences are considered, not
    syntax differences. When invalid XML is passed in any parameter, an
    ``AssertionError`` is always raised, even if both string are identical.

    Output in case of error can be customized with the ``msg`` argument.

.. method:: SimpleTestCase.assertXMLNotEqual(xml1, xml2, msg=None)

    Asserts that the strings ``xml1`` and ``xml2`` are *not* equal. The
    comparison is based on XML semantics. See
    :meth:`~SimpleTestCase.assertXMLEqual` for details.

    Output in case of error can be customized with the ``msg`` argument.

.. method:: SimpleTestCase.assertInHTML(needle, haystack, count=None, msg_prefix='')

    Asserts that the HTML fragment ``needle`` is contained in the ``haystack`` one.

    If the ``count`` integer argument is specified, then additionally the number
    of ``needle`` occurrences will be strictly verified.

    Whitespace in most cases is ignored, and attribute ordering is not
    significant. The passed-in arguments must be valid HTML.

.. method:: SimpleTestCase.assertJSONEqual(raw, expected_data, msg=None)

    Asserts that the JSON fragments ``raw`` and ``expected_data`` are equal.
    Usual JSON non-significant whitespace rules apply as the heavyweight is
    delegated to the :mod:`json` library.

    Output in case of error can be customized with the ``msg`` argument.

.. method:: SimpleTestCase.assertJSONNotEqual(raw, expected_data, msg=None)

    Asserts that the JSON fragments ``raw`` and ``expected_data`` are *not* equal.
    See :meth:`~SimpleTestCase.assertJSONEqual` for further details.

    Output in case of error can be customized with the ``msg`` argument.

.. method:: TransactionTestCase.assertQuerysetEqual(qs, values, transform=repr, ordered=True, msg=None)

    Asserts that a queryset ``qs`` returns a particular list of values ``values``.

    The comparison of the contents of ``qs`` and ``values`` is performed using
    the function ``transform``; by default, this means that the ``repr()`` of
    each value is compared. Any other callable can be used if ``repr()`` doesn't
    provide a unique or helpful comparison.

    By default, the comparison is also ordering dependent. If ``qs`` doesn't
    provide an implicit ordering, you can set the ``ordered`` parameter to
    ``False``, which turns the comparison into a ``collections.Counter`` comparison.
    If the order is undefined (if the given ``qs`` isn't ordered and the
    comparison is against more than one ordered values), a ``ValueError`` is
    raised.

    Output in case of error can be customized with the ``msg`` argument.

.. method:: TransactionTestCase.assertNumQueries(num, func, \*args, \*\*kwargs)

    Asserts that when ``func`` is called with ``*args`` and ``**kwargs`` that
    ``num`` database queries are executed.

    If a ``"using"`` key is present in ``kwargs`` it is used as the database
    alias for which to check the number of queries.  If you wish to call a
    function with a ``using`` parameter you can do it by wrapping the call with
    a ``lambda`` to add an extra parameter::

        self.assertNumQueries(7, lambda: my_function(using=7))

    You can also use this as a context manager::

        with self.assertNumQueries(2):
            Person.objects.create(name="Aaron")
            Person.objects.create(name="Daniel")

.. _topics-testing-email:

Email services
--------------

If any of your Django views send email using Django's email
functionality, you probably don't want to send email each time
you run a test using that view. For this reason, Django's test runner
automatically redirects all Django-sent email to a dummy outbox. This lets you
test every aspect of sending email -- from the number of messages sent to the
contents of each message -- without actually sending the messages.

The test runner accomplishes this by transparently replacing the normal
email backend with a testing backend.
(Don't worry -- this has no effect on any other email senders outside of
Django, such as your machine's mail server, if you're running one.)

.. currentmodule:: django.core.mail

.. data:: django.core.mail.outbox

During test running, each outgoing email is saved in
``django.core.mail.outbox``. This is a simple list of all
:class:`~django.core.mail.EmailMessage` instances that have been sent.
The ``outbox`` attribute is a special attribute that is created *only* when
the ``locmem`` email backend is used. It doesn't normally exist as part of the
:mod:`django.core.mail` module and you can't import it directly. The code
below shows how to access this attribute correctly.

Here's an example test that examines ``django.core.mail.outbox`` for length
and contents::

    from django.core import mail
    from django.test import TestCase

    class EmailTest(TestCase):
        def test_send_email(self):
            # Send message.
            mail.send_mail('Subject here', 'Here is the message.',
                'from@example.com', ['to@example.com'],
                fail_silently=False)

            # Test that one message has been sent.
            self.assertEqual(len(mail.outbox), 1)

            # Verify that the subject of the first message is correct.
            self.assertEqual(mail.outbox[0].subject, 'Subject here')

As noted previously, the test outbox is emptied
at the start of every test in a Django ``*TestCase``. To empty the outbox
manually, assign the empty list to ``mail.outbox``::

    from django.core import mail

    # Empty the test outbox
    mail.outbox = []

.. _topics-testing-management-commands:

Management Commands
-------------------

Management commands can be tested with the
:func:`~django.core.management.call_command` function. The output can be
redirected into a ``StringIO`` instance::

    from django.core.management import call_command
    from django.test import TestCase
    from django.utils.six import StringIO

    class ClosepollTest(TestCase):
        def test_command_output(self):
            out = StringIO()
            call_command('closepoll', stdout=out)
            self.assertIn('Expected output', out.getvalue())

.. _skipping-tests:

Skipping tests
--------------

.. currentmodule:: django.test

The unittest library provides the :func:`@skipIf <unittest.skipIf>` and
:func:`@skipUnless <unittest.skipUnless>` decorators to allow you to skip tests
if you know ahead of time that those tests are going to fail under certain
conditions.

For example, if your test requires a particular optional library in order to
succeed, you could decorate the test case with :func:`@skipIf
<unittest.skipIf>`. Then, the test runner will report that the test wasn't
executed and why, instead of failing the test or omitting the test altogether.

To supplement these test skipping behaviors, Django provides two
additional skip decorators. Instead of testing a generic boolean,
these decorators check the capabilities of the database, and skip the
test if the database doesn't support a specific named feature.

The decorators use a string identifier to describe database features.
This string corresponds to attributes of the database connection
features class. See ``django.db.backends.BaseDatabaseFeatures``
class for a full list of database features that can be used as a basis
for skipping tests.

.. function:: skipIfDBFeature(*feature_name_strings)

Skip the decorated test or ``TestCase`` if all of the named database features
are supported.

For example, the following test will not be executed if the database
supports transactions (e.g., it would *not* run under PostgreSQL, but
it would under MySQL with MyISAM tables)::

    class MyTests(TestCase):
        @skipIfDBFeature('supports_transactions')
        def test_transaction_behavior(self):
            # ... conditional test code

    ``skipIfDBFeature`` can accept multiple feature strings.

.. function:: skipUnlessDBFeature(*feature_name_strings)

Skip the decorated test or ``TestCase`` if any of the named database features
are *not* supported.

For example, the following test will only be executed if the database
supports transactions (e.g., it would run under PostgreSQL, but *not*
under MySQL with MyISAM tables)::

    class MyTests(TestCase):
        @skipUnlessDBFeature('supports_transactions')
        def test_transaction_behavior(self):
            # ... conditional test code

    ``skipUnlessDBFeature`` can accept multiple feature strings.

Advanced testing topics
=======================

The request factory
===================

.. currentmodule:: django.test

.. class:: RequestFactory

The :class:`~django.test.RequestFactory` shares the same API as
the test client. However, instead of behaving like a browser, the
RequestFactory provides a way to generate a request instance that can
be used as the first argument to any view. This means you can test a
view function the same way as you would test any other function -- as
a black box, with exactly known inputs, testing for specific outputs.

The API for the :class:`~django.test.RequestFactory` is a slightly
restricted subset of the test client API:

* It only has access to the HTTP methods :meth:`~Client.get()`,
  :meth:`~Client.post()`, :meth:`~Client.put()`,
  :meth:`~Client.delete()`, :meth:`~Client.head()`,
  :meth:`~Client.options()`, and :meth:`~Client.trace()`.

* These methods accept all the same arguments *except* for
  ``follows``. Since this is just a factory for producing
  requests, it's up to you to handle the response.

* It does not support middleware. Session and authentication
  attributes must be supplied by the test itself if required
  for the view to function properly.

Example
-------

The following is a simple unit test using the request factory::

    from django.contrib.auth.models import AnonymousUser, User
    from django.test import TestCase, RequestFactory

    class SimpleTest(TestCase):
        def setUp(self):
            # Every test needs access to the request factory.
            self.factory = RequestFactory()
            self.user = User.objects.create_user(
                username='jacob', email='jacob@…', password='top_secret')

        def test_details(self):
            # Create an instance of a GET request.
            request = self.factory.get('/customer/details')

            # Recall that middleware are not supported. You can simulate a
            # logged-in user by setting request.user manually.
            request.user = self.user

            # Or you can simulate an anonymous user by setting request.user to
            # an AnonymousUser instance.
            request.user = AnonymousUser()

            # Test my_view() as if it were deployed at /customer/details
            response = my_view(request)
            self.assertEqual(response.status_code, 200)

.. _topics-testing-advanced-multidb:

Tests and multiple databases
============================

.. _topics-testing-primaryreplica:

Testing primary/replica configurations
--------------------------------------

If you're testing a multiple database configuration with primary/replica
(referred to as master/slave by some databases) replication, this strategy of
creating test databases poses a problem.
When the test databases are created, there won't be any replication,
and as a result, data created on the primary won't be seen on the
replica.

To compensate for this, Django allows you to define that a database is
a *test mirror*. Consider the following (simplified) example database
configuration::

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'myproject',
            'HOST': 'dbprimary',
             # ... plus some other settings
        },
        'replica': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'myproject',
            'HOST': 'dbreplica',
            'TEST_MIRROR': 'default'
            # ... plus some other settings
        }
    }

In this setup, we have two database servers: ``dbprimary``, described
by the database alias ``default``, and ``dbreplica`` described by the
alias ``replica``. As you might expect, ``dbreplica`` has been configured
by the database administrator as a read replica of ``dbprimary``, so in
normal activity, any write to ``default`` will appear on ``replica``.

If Django created two independent test databases, this would break any
tests that expected replication to occur. However, the ``replica``
database has been configured as a test mirror (using the
``TEST_MIRROR`` setting), indicating that under testing,
``replica`` should be treated as a mirror of ``default``.

When the test environment is configured, a test version of ``replica``
will *not* be created. Instead the connection to ``replica``
will be redirected to point at ``default``. As a result, writes to
``default`` will appear on ``replica`` -- but because they are actually
the same database, not because there is data replication between the
two databases.

.. _topics-testing-creation-dependencies:

Controlling creation order for test databases
---------------------------------------------

By default, Django will assume all databases depend on the ``default``
database and therefore always create the ``default`` database first.
However, no guarantees are made on the creation order of any other
databases in your test setup.

If your database configuration requires a specific creation order, you
can specify the dependencies that exist using the
``TEST_DEPENDENCIES`` setting. Consider the following
(simplified) example database configuration::

    DATABASES = {
        'default': {
             # ... db settings
             'TEST_DEPENDENCIES': ['diamonds']
        },
        'diamonds': {
            # ... db settings
             'TEST_DEPENDENCIES': []
        },
        'clubs': {
            # ... db settings
            'TEST_DEPENDENCIES': ['diamonds']
        },
        'spades': {
            # ... db settings
            'TEST_DEPENDENCIES': ['diamonds','hearts']
        },
        'hearts': {
            # ... db settings
            'TEST_DEPENDENCIES': ['diamonds','clubs']
        }
    }

Under this configuration, the ``diamonds`` database will be created first,
as it is the only database alias without dependencies. The ``default`` and
``clubs`` alias will be created next (although the order of creation of this
pair is not guaranteed); then ``hearts``; and finally ``spades``.

If there are any circular dependencies in the
``TEST_DEPENDENCIES`` definition, an ``ImproperlyConfigured``
exception will be raised.

Advanced features of ``TransactionTestCase``
============================================

.. attribute:: TransactionTestCase.available_apps

    .. warning::

        This attribute is a private API. It may be changed or removed without
        a deprecation period in the future, for instance to accommodate changes
        in application loading.

        It's used to optimize Django's own test suite, which contains hundreds
        of models but no relations between models in different applications.

    By default, ``available_apps`` is set to ``None``. After each test, Django
    calls ``flush`` to reset the database state. This empties all tables
    and emits the :data:`~django.db.models.signals.post_migrate` signal, which
    re-creates one content type and three permissions for each model. This
    operation gets expensive proportionally to the number of models.

    Setting ``available_apps`` to a list of applications instructs Django to
    behave as if only the models from these applications were available. The
    behavior of ``TransactionTestCase`` changes as follows:

    - :data:`~django.db.models.signals.post_migrate` is fired before each
      test to create the content types and permissions for each model in
      available apps, in case they're missing.
    - After each test, Django empties only tables corresponding to models in
      available apps. However, at the database level, truncation may cascade to
      related models in unavailable apps. Furthermore
      :data:`~django.db.models.signals.post_migrate` isn't fired; it will be
      fired by the next ``TransactionTestCase``, after the correct set of
      applications is selected.

    Since the database isn't fully flushed, if a test creates instances of
    models not included in ``available_apps``, they will leak and they may
    cause unrelated tests to fail. Be careful with tests that use sessions;
    the default session engine stores them in the database.

    Since :data:`~django.db.models.signals.post_migrate` isn't emitted after
    flushing the database, its state after a ``TransactionTestCase`` isn't the
    same as after a ``TestCase``: it's missing the rows created by listeners
    to :data:`~django.db.models.signals.post_migrate`. Considering the
    order in which tests are executed, this isn't an
    issue, provided either all ``TransactionTestCase`` in a given test suite
    declare ``available_apps``, or none of them.

    ``available_apps`` is mandatory in Django's own test suite.

.. attribute:: TransactionTestCase.reset_sequences

    Setting ``reset_sequences = True`` on a ``TransactionTestCase`` will make
    sure sequences are always reset before the test run::

        class TestsThatDependsOnPrimaryKeySequences(TransactionTestCase):
            reset_sequences = True

            def test_animal_pk(self):
                lion = Animal.objects.create(name="lion", sound="roar")
                # lion.pk is guaranteed to always be 1
                self.assertEqual(lion.pk, 1)

    Unless you are explicitly testing primary keys sequence numbers, it is
    recommended that you do not hard code primary key values in tests.

    Using ``reset_sequences = True`` will slow down the test, since the primary
    key reset is an relatively expensive database operation.


Using the Django test runner to test reusable applications
==========================================================

If you are writing a reusable application 
you may want to use the Django test runner to run your own test suite
and thus benefit from the Django testing infrastructure.

A common practice is a *tests* directory next to the application code, with the
following structure::

    runtests.py
    polls/
        __init__.py
        models.py
        ...
    tests/
        __init__.py
        models.py
        test_settings.py
        tests.py

Let's take a look inside a couple of those files:

.. code-block:: python
    
    # runtests.py

    #!/usr/bin/env python
    import os
    import sys

    import django
    from django.conf import settings
    from django.test.utils import get_runner

    if __name__ == "__main__":
        os.environ['DJANGO_SETTINGS_MODULE'] = 'tests.test_settings'
        django.setup()
        TestRunner = get_runner(settings)
        test_runner = TestRunner()
        failures = test_runner.run_tests(["tests"])
        sys.exit(bool(failures))


This is the script that you invoke to run the test suite. It sets up the
Django environment, creates the test database and runs the tests.

For the sake of clarity, this example contains only the bare minimum
necessary to use the Django test runner. You may want to add
command-line options for controlling verbosity, passing in specific test
labels to run, etc.

.. code-block:: python
    
   # tests/test_settings.py

    SECRET_KEY = 'fake-key'
    INSTALLED_APPS = [
        "tests",
    ]

This file contains the Django settings 
required to run your app's tests.

Again, this is a minimal example; your tests may require additional
settings to run.

Since the *tests* package is included in ``INSTALLED_APPS`` when
running your tests, you can define test-only models in its ``models.py``
file.


.. _other-testing-frameworks:

Using different testing frameworks
==================================

Clearly, :mod:`unittest` is not the only Python testing framework. While Django
doesn't provide explicit support for alternative frameworks, it does provide a
way to invoke tests constructed for an alternative framework as if they were
normal Django tests.

When you run ``./manage.py test``, Django looks at the ``TEST_RUNNER``
setting to determine what to do. By default,````TEST_RUNNER`` points to
``'django.test.runner.DiscoverRunner'``. This class defines the default Django
testing behavior. This behavior involves:

#. Performing global pre-test setup.

#. Looking for tests in any file below the current directory whose name matches
   the pattern ``test*.py``.

#. Creating the test databases.

#. Running ``migrate`` to install models and initial data into the test
   databases.

#. Running the tests that were found.

#. Destroying the test databases.

#. Performing global post-test teardown.

If you define your own test runner class and point ``TEST_RUNNER`` at
that class, Django will execute your test runner whenever you run
``./manage.py test``. In this way, it is possible to use any test framework
that can be executed from Python code, or to modify the Django test execution
process to satisfy whatever testing requirements you may have.

.. _topics-testing-test_runner:

Defining a test runner
----------------------

.. currentmodule:: django.test.runner

A test runner is a class defining a ``run_tests()`` method. Django ships
with a ``DiscoverRunner`` class that defines the default Django testing
behavior. This class defines the ``run_tests()`` entry point, plus a
selection of other methods that are used to by ``run_tests()`` to set up,
execute and tear down the test suite.

.. class:: DiscoverRunner(pattern='test*.py', top_level=None, verbosity=1, interactive=True, failfast=True, keepdb=False, reverse=False, debug_sql=False, \*\*kwargs)

    ``DiscoverRunner`` will search for tests in any file matching ``pattern``.

    ``top_level`` can be used to specify the directory containing your
    top-level Python modules. Usually Django can figure this out automatically,
    so it's not necessary to specify this option. If specified, it should
    generally be the directory containing your ``manage.py`` file.

    ``verbosity`` determines the amount of notification and debug information
    that will be printed to the console; ``0`` is no output, ``1`` is normal
    output, and ``2`` is verbose output.

    If ``interactive`` is ``True``, the test suite has permission to ask the
    user for instructions when the test suite is executed. An example of this
    behavior would be asking for permission to delete an existing test
    database. If ``interactive`` is ``False``, the test suite must be able to
    run without any manual intervention.

    If ``failfast`` is ``True``, the test suite will stop running after the
    first test failure is detected.

    If ``keepdb`` is ``True``, the test suite will use the existing database,
    or create one if necessary. If ``False``, a new database will be created,
    prompting the user to remove the existing one, if present.

    If ``reverse`` is ``True``, test cases will be executed in the opposite
    order. This could be useful to debug tests that aren't properly isolated
    and have side effects. Grouping by test class is
    preserved when using this option.

    If ``debug_sql`` is ``True``, failing test cases will output SQL queries
    logged to the django.db.backends logger as well
    as the traceback. If ``verbosity`` is ``2``, then queries in all tests are
    output.

    Django may, from time to time, extend the capabilities of the test runner
    by adding new arguments. The ``**kwargs`` declaration allows for this
    expansion. If you subclass ``DiscoverRunner`` or write your own test
    runner, ensure it accepts ``**kwargs``.

    Your test runner may also define additional command-line options.
    Create or override an ``add_arguments(cls, parser)`` class method and add
    custom arguments by calling ``parser.add_argument()`` inside the method, so
    that the ``test`` command will be able to use those arguments.


Attributes
~~~~~~~~~~

.. attribute:: DiscoverRunner.test_suite

    The class used to build the test suite. By default it is set to
    ``unittest.TestSuite``. This can be overridden if you wish to implement
    different logic for collecting tests.

.. attribute:: DiscoverRunner.test_runner

    This is the class of the low-level test runner which is used to execute
    the individual tests and format the results. By default it is set to
    ``unittest.TextTestRunner``. Despite the unfortunate similarity in
    naming conventions, this is not the same type of class as
    ``DiscoverRunner``, which covers a broader set of responsibilities. You
    can override this attribute to modify the way tests are run and reported.

.. attribute:: DiscoverRunner.test_loader

    This is the class that loads tests, whether from TestCases or modules or
    otherwise and bundles them into test suites for the runner to execute.
    By default it is set to ``unittest.defaultTestLoader``. You can override
    this attribute if your tests are going to be loaded in unusual ways.

.. attribute:: DiscoverRunner.option_list

    This is the tuple of ``optparse`` options which will be fed into the
    management command's ``OptionParser`` for parsing arguments. See the
    documentation for Python's ``optparse`` module for more details.

    .. deprecated:: 1.8

        You should now override the :meth:`~DiscoverRunner.add_arguments` class
        method to add custom arguments accepted by the ``test``
        management command.

Methods
~~~~~~~

.. method:: DiscoverRunner.run_tests(test_labels, extra_tests=None, \*\*kwargs)

    Run the test suite.

    ``test_labels`` allows you to specify which tests to run and supports
    several formats (see :meth:`DiscoverRunner.build_suite` for a list of
    supported formats).

    ``extra_tests`` is a list of extra ``TestCase`` instances to add to the
    suite that is executed by the test runner. These extra tests are run
    in addition to those discovered in the modules listed in ``test_labels``.

    This method should return the number of tests that failed.

.. classmethod:: DiscoverRunner.add_arguments(parser)

    Override this class method to add custom arguments accepted by the
    ``test`` management command. See
    :py:meth:`argparse.ArgumentParser.add_argument()` for details about adding
    arguments to a parser.

.. method:: DiscoverRunner.setup_test_environment(\*\*kwargs)

    Sets up the test environment by calling
    :func:`~django.test.utils.setup_test_environment` and setting
    ``DEBUG`` to ``False``.

.. method:: DiscoverRunner.build_suite(test_labels, extra_tests=None, \*\*kwargs)

    Constructs a test suite that matches the test labels provided.

    ``test_labels`` is a list of strings describing the tests to be run. A test
    label can take one of four forms:

    * ``path.to.test_module.TestCase.test_method`` -- Run a single test method
      in a test case.
    * ``path.to.test_module.TestCase`` -- Run all the test methods in a test
      case.
    * ``path.to.module`` -- Search for and run all tests in the named Python
      package or module.
    * ``path/to/directory`` -- Search for and run all tests below the named
      directory.

    If ``test_labels`` has a value of ``None``, the test runner will search for
    tests in all files below the current directory whose names match its
    ``pattern`` (see above).

    ``extra_tests`` is a list of extra ``TestCase`` instances to add to the
    suite that is executed by the test runner. These extra tests are run
    in addition to those discovered in the modules listed in ``test_labels``.

    Returns a ``TestSuite`` instance ready to be run.

.. method:: DiscoverRunner.setup_databases(\*\*kwargs)

    Creates the test databases.

    Returns a data structure that provides enough detail to undo the changes
    that have been made. This data will be provided to the ``teardown_databases()``
    function at the conclusion of testing.

.. method:: DiscoverRunner.run_suite(suite, \*\*kwargs)

    Runs the test suite.

    Returns the result produced by the running the test suite.

.. method:: DiscoverRunner.teardown_databases(old_config, \*\*kwargs)

    Destroys the test databases, restoring pre-test conditions.

    ``old_config`` is a data structure defining the changes in the
    database configuration that need to be reversed. It is the return
    value of the ``setup_databases()`` method.

.. method:: DiscoverRunner.teardown_test_environment(\*\*kwargs)

    Restores the pre-test environment.

.. method:: DiscoverRunner.suite_result(suite, result, \*\*kwargs)

    Computes and returns a return code based on a test suite, and the result
    from that test suite.


Testing utilities
-----------------

django.test.utils
~~~~~~~~~~~~~~~~~

.. module:: django.test.utils
   :synopsis: Helpers to write custom test runners.

To assist in the creation of your own test runner, Django provides a number of
utility methods in the ``django.test.utils`` module.

.. function:: setup_test_environment()

    Performs any global pre-test setup, such as the installing the
    instrumentation of the template rendering system and setting up
    the dummy email outbox.

.. function:: teardown_test_environment()

    Performs any global post-test teardown, such as removing the black
    magic hooks into the template system and restoring normal email
    services.

django.db.connection.creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. currentmodule:: django.db.connection.creation

The creation module of the database backend also provides some utilities that
can be useful during testing.

.. function:: create_test_db([verbosity=1, autoclobber=False, serialize=True, keepdb=False])

    Creates a new test database and runs ``migrate`` against it.

    ``verbosity`` has the same behavior as in ``run_tests()``.

    ``autoclobber`` describes the behavior that will occur if a
    database with the same name as the test database is discovered:

    * If ``autoclobber`` is ``False``, the user will be asked to
      approve destroying the existing database. ``sys.exit`` is
      called if the user does not approve.

    * If autoclobber is ``True``, the database will be destroyed
      without consulting the user.

    ``serialize`` determines if Django serializes the database into an
    in-memory JSON string before running tests (used to restore the database
    state between tests if you don't have transactions). You can set this to
    ``False`` to speed up creation time if you don't have any test classes
    with serialized_rollback=True.

    If you are using the default test runner, you can control this with the
    the ``SERIALIZE <TEST_SERIALIZE>`` entry in the ``TEST``

    ``keepdb`` determines if the test run should use an existing
    database, or create a new one. If ``True``, the existing
    database will be used, or created if not present. If ``False``,
    a new database will be created, prompting the user to remove
    the existing one, if present.

    Returns the name of the test database that it created.

    ``create_test_db()`` has the side effect of modifying the value of
    ``NAME`` in ``DATABASES`` to match the name of the test
    database.

.. function:: destroy_test_db(old_database_name, [verbosity=1, keepdb=False])

    Destroys the database whose name is the value of ``NAME`` in
    ``DATABASES``, and sets ``NAME`` to the value of
    ``old_database_name``.

    The ``verbosity`` argument has the same behavior as for
    :class:`~django.test.runner.DiscoverRunner`.

    If the ``keepdb`` argument is ``True``, then the connection to the
    database will be closed, but the database will not be destroyed.

.. _topics-testing-code-coverage:

Integration with coverage.py
============================

Code coverage describes how much source code has been tested. It shows which
parts of your code are being exercised by tests and which are not. It's an
important part of testing applications, so it's strongly recommended to check
the coverage of your tests.

Django can be easily integrated with `coverage.py`_, a tool for measuring code
coverage of Python programs. First, `install coverage.py`_. Next, run the
following from your project folder containing ``manage.py``::

   coverage run --source='.' manage.py test myapp

This runs your tests and collects coverage data of the executed files in your
project. You can see a report of this data by typing following command::

   coverage report

Note that some Django code was executed while running tests, but it is not
listed here because of the ``source`` flag passed to the previous command.

For more options like annotated HTML listings detailing missed lines, see the
`coverage.py`_ docs.

.. _coverage.py: http://nedbatchelder.com/code/coverage/
.. _install coverage.py: https://pypi.python.org/pypi/coverage

