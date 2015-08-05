======================
Introduction to Django
======================


Great open source software almost always comes about because one or more
clever developers had a problem to solve and no viable or cost effective
solution available. Django is no exception. 

Adrian and Jacob have long since "retired" from the project, but the
fundamentals of what drove them to create Django live on. It is this solid base
of real-world experience that has made Django as successful as it is. 

In recognition of their contribution, I think it best we let them introduce
Django in their own words (edited and reformatted from the original book).

Introducing Django
==================

By Adrian Holovaty and Jacob Kaplan-Moss - December 2009
--------------------------------------------------------

In the early days, Web developers wrote every page by hand. Updating a Web site
meant editing HTML; a "redesign" involved redoing every single page, one at a
time.

As Web sites grew and became more ambitious, it quickly became obvious that
that approach was tedious, time-consuming, and ultimately untenable. A group
of enterprising hackers at NCSA (the National Center for Supercomputing
Applications, where Mosaic, the first graphical Web browser, was developed)
solved this problem by letting the Web server spawn external programs that
could dynamically generate HTML. They called this protocol the Common Gateway
Interface, or CGI, and it changed the Web forever.

It's hard now to imagine what a revelation CGI must have been: instead of
treating HTML pages as simple files on disk, CGI allows you to think of your
pages as resources generated dynamically on demand. The development of CGI
ushered in the first generation of dynamic Web sites.

However, CGI has its problems: CGI scripts need to contain a lot of repetitive
"boilerplate" code, they make code reuse difficult, and they can be difficult
for first-time developers to write and understand.

PHP fixed many of these problems, and it took the world by storm -- it's now by
far the most popular tool used to create dynamic Web sites, and dozens of
similar languages and environments (ASP, JSP, etc.) followed PHP's design
closely. PHP's major innovation is its ease of use: PHP code is simply embedded
into plain HTML; the learning curve for someone who already knows HTML is
extremely shallow.

But PHP has its own problems; its very ease of use encourages sloppy,
repetitive, ill-conceived code. Worse, PHP does little to protect programmers
from security vulnerabilities, and thus many PHP developers found themselves
learning about security only once it was too late.

These and similar frustrations led directly to the development of the current
crop of "third-generation" Web development frameworks. With this new explosion of Web
development comes yet another increase in ambition; Web developers are expected
to do more and more every day.

Django was invented to meet these new ambitions. 

Django's History
================

Django grew organically from real-world applications written by a Web
development team in Lawrence, Kansas, USA. It was born in the fall of 2003,
when the Web programmers at the *Lawrence Journal-World* newspaper, Adrian
Holovaty and Simon Willison, began using Python to build applications.

The World Online team, responsible for the production and maintenance of
several local news sites, thrived in a development environment dictated by
journalism deadlines. For the sites -- including LJWorld.com, Lawrence.com and
KUsports.com -- journalists (and management) demanded that features be added
and entire applications be built on an intensely fast schedule, often with only
days' or hours' notice. Thus, Simon and Adrian developed a time-saving Web
development framework out of necessity -- it was the only way they could build
maintainable applications under the extreme deadlines.

In summer 2005, after having developed this framework to a point where it was
efficiently powering most of World Online's sites, the team, which now included
Jacob Kaplan-Moss, decided to release the framework as open source software.
They released it in July 2005 and named it Django, after the jazz guitarist
Django Reinhardt.

This history is relevant because it helps explain two key things. The first is
Django's "sweet spot." Because Django was born in a news environment, it offers
several features (such as its admin site, covered in Chapter 6) that are
particularly well suited for "content" sites -- sites like Amazon.com,
craigslist.org, and washingtonpost.com that offer dynamic, database-driven
information. Don't let that turn you off, though -- although Django is
particularly good for developing those sorts of sites, that doesn't preclude it
from being an effective tool for building any sort of dynamic Web site.
(There's a difference between being *particularly effective* at something and
being *ineffective* at other things.)

The second matter to note is how Django's origins have shaped the culture of
its open source community. Because Django was extracted from real-world code,
rather than being an academic exercise or commercial product, it is acutely
focused on solving Web development problems that Django's developers themselves
have faced -- and continue to face. As a result, Django itself is actively
improved on an almost daily basis. The framework's maintainers have a vested
interest in making sure Django saves developers time, produces applications
that are easy to maintain and performs well under load. 

Django lets you build deep, dynamic, interesting sites in an extremely short
time. Django is designed to let you focus on the fun, interesting parts of
your job while easing the pain of the repetitive bits. In doing so, it
provides high-level abstractions of common Web development patterns, shortcuts
for frequent programming tasks, and clear conventions on how to solve
problems. At the same time, Django tries to stay out of your way, letting you
work outside the scope of the framework as needed. We wrote this book because
we firmly believe that Django makes Web development better. It's designed to
quickly get you moving on your own Django projects, and then ultimately teach
you everything you need to know to successfully design, develop, and deploy a
site that you'll be proud of.

