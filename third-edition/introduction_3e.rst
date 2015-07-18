=================================
Introduction to the third edition
=================================

This year, it will be 30 years since I plugged the 5.25" DOS 3.3 disk
into my school's very first Apple IIe computer and discovered BASIC. 

In the intervening years I have written more lines of code than I could
guess in about a dozen languages. I still write code every
week - although the list of languages, and number of lines are somewhat
diminished these days. Over the years I have seen plenty of horrible
code and some really good stuff too. In my own work, I have written my
fair share of good and bad.

Interestingly, not once in my entire career have I been employed as a
programmer. I had my own IT business for five years, and have been in
businesses large and small - mostly in R&D, technical and operations
management - but never working solely as a programmer. What I have been
is the tech-head that gets called up to **Get Stuff Done**.

Emphasized for good reason - business is all about Getting Stuff Done.

When everything has to work yesterday, religious wars over curly braces
and pontification over which language is best for what application
become trivialities. 

Having read dozens and dozens of textbooks on all the various
programming languages I have used, I know why you are here reading
the introduction, so let's get right to the point.

Why should you care about Django?
=================================

While it is a given that Django is not the only web framework that will allow
you to Get Stuff Done, I can confidently say one thing - if you want to write
clean, intelligible code and build high performance, good looking modern
websites quickly, then you will definitely benefit from working through this
book.

I have deliberately not rattled off comparisons with other languages and
frameworks because that is not the point - all languages and the frameworks and
tools built on them have strengths and weaknesses. However, having worked with
many of them over the years, I am totally convinced that Django stands way out
in front for ease of use and ability to allow a programmer to produce robust,
secure, and bug free code quickly.

Django is spectacularly good at getting out of your way when you just
need to Get Something Done, but still exposes all the good stuff just
under the surface when you want to dig down further. It is also built
with Python, arguably the most intelligible and easy to learn
programming language.

Of course these strengths do bring one challenge. Because both Python and
Django hide an enormous amount of power and functionality just below the
surface, it can be a bit confusing for beginners. This is where this
book comes in. It's designed to quickly get you moving on your own
Django projects, and then ultimately teach you everything you need to
know to successfully design, develop, and deploy a site that you'll be
proud of.

Adrian and Jacob wrote the original Django Book because they firmly
believed that Django makes Web development better. I think Django's
longevity and exponential growth in the 6 years since the publication of
the original Django Book is testament to this belief.

As per the original, this book is open source and all are welcome to
improve it by either submitting comments and suggestions at the
`website`_ or sending me an email to nigel at newdjangobook dot com.

I, like many, get a great deal of pleasure out of working with Django -
it truly is as exciting, fun and useful as Adrian and Jacob had hoped it
would be!

.. _website: http://djangobook.com/

About This Book
===============

This book is about Django, a Web development framework that saves you time
and makes Web development a joy. Using Django, you can build and maintain
high-quality Web applications with minimal fuss.

The main goal of this book is to make you a Django expert. The focus is twofold.
First, we explain in depth what Django does and how to build Web
applications with it. Second, we discuss higher-level concepts where
appropriate, answering the question "How can I apply these tools effectively
in my own projects?" By reading this book, you'll learn the skills needed to
develop powerful Web sites quickly, with code that is clean and easy to
maintain.

The secondary, but no less important, goal of this book is to provide a
programmers manual that covers the current LTS version of Django. Django has
matured to the point where it is seeing many commercial and business critical
deployments. As such, this book is intended to provide the definitive
up-to-date resource for commercial deployment of Django 1.8LTS.  The electronic
version of this book will be kept in sync with Django 1.8 right up until the
end of extended support (2018). 

How to Read This Book
=====================

In writing this Third Edition, I have tried to maintain a similar balance
between readability and reference as the first book, however Django has grown
considerably in the last 8 years and with increased power and flexibility,
comes some additional complexity.

Django still has one of the shortest learning curves of all the web
application frameworks, but there is still some solid work ahead of you if you
want to become a Django expert.

This book retains the same "learn by example" philosophy as the original book,
however some of the more complex sections (database configuration for example)
have been moved to later chapters, so that you can first learn how Django
works with a simple, out-of-the-box configuration and then build on your
knowledge with more advanced topics later.

With that in mind, I recommend that you read Chapters 1 through 13 in order.
They form the foundation of how to use Django; once you've read them, you'll be
able to build and deploy Django-powered Web sites. Specifically, Chapters 1
through 6 are the "core curriculum," Chapters 7 through 12 cover more advanced
Django usage, and Chapter 13 covers deployment. The remaining chapters, 14
through 23, focus on specific Django features and can be read in any order.

The appendices are for reference. They, along with the free documentation at
http://www.djangoproject.com/, are probably what you'll flip back to
occasionally to recall syntax or find quick synopses of what certain parts of
Django do.

Required Programming Knowledge
------------------------------

Readers of this book should understand the basics of procedural and
object-oriented programming: control structures (e.g., ``if``, ``while``,
``for``), data structures (lists, hashes/dictionaries), variables, classes and
objects.

Experience in Web development is, as you may expect, very helpful, but it's
not required to understand this book. Throughout the book, we try to promote
best practices in Web development for readers who lack this experience.

Required Python Knowledge
-------------------------

At its core, Django is simply a collection of libraries written in the Python
programming language. To develop a site using Django, you write Python code
that uses these libraries. Learning Django, then, is a matter of learning how
to program in Python and understanding how the Django libraries work.

If you have experience programming in Python, you should have no trouble diving
in. By and large, the Django code doesn't perform a lot of "magic" (i.e.,
programming trickery whose implementation is difficult to explain or
understand). For you, learning Django will be a matter of learning Django's
conventions and APIs.

If you don't have experience programming in Python, you're in for a treat.
It's easy to learn and a joy to use! Although this book doesn't include a full
Python tutorial, it highlights Python features and functionality where
appropriate, particularly when code doesn't immediately make sense. Still, we
recommend you read the official Python tutorial, available online at
http://docs.python.org/tut/. We also recommend Mark Pilgrim's free book
*Dive Into Python*, available at http://www.diveintopython.net/ and published in
print by Apress.

Required Django Version
-----------------------

This book covers Django 1.8 LTS.

This is the long term support version of Django, with full support from the
Django developers until at least April 2018.

If you have an early version of Django, it is recommended that you upgrade to
the latest version of Django 1.8LTS.

At the time of printing, the most current production version of Django 1.8LTS
is 1.8.2.

If you have installed a later version of Django, please note that while
Django's developers maintain backwards compatibility as much as possible, some
backwards incompatible changes do get introduced occasionally.  The changes in
each release are always covered in the release notes, which you can find here:
https://docs.djangoproject.com/en/dev/releases/


Getting Help
------------

One of the greatest benefits of Django is its kind and helpful user community.
For help with any aspect of Django -- from installation, to application design,
to database design, to deployment -- feel free to ask questions online.

* The django-users mailing list is where thousands of Django users hang out
  to ask and answer questions. Sign up for free at http://www.djangoproject.com/r/django-users.

* The Django IRC channel is where Django users hang out to chat and help
  each other in real time. Join the fun by logging on to #django on the
  Freenode IRC network.

What's Next
-----------

In `Chapter 2`_, we'll get started with Django, covering installation and
initial setup.

.. _Chapter 2: chapter02.html
