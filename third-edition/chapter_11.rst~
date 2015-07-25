=========================================
Chapter 11: User authentication in Django
=========================================

A significant percentage of modern, interactive websites allow some form of
user interaction - from allowing simple comments on a blog, to full editorial
control of articles on a news site. If a site offers any sort of eCommerce,
authentication and authorization of paying customers is essential.

Just managing users -- lost usernames, forgotten passwords and keeping
information up to date -- can be a real pain. As a programmer, writing an
authentication system can be even worse.

Lucky for us, Django comes with a user authentication system installed
automatically for your convenience when your ran ``django-admin
startproject``. 

Django provides a default implementation for managing user accounts, groups,
permissions and cookie-based user sessions out of the box.

Like most things in Django, the default implementation is fully extendible and
customizable to suit your project's needs. So let's jump right in.

Overview
========

The Django authentication system handles both authentication and authorization.
Briefly, authentication verifies a user is who they claim to be, and
authorization determines what an authenticated user is allowed to do. Here the
term authentication is used to refer to both tasks.

The auth system consists of:

* Users
* Permissions: Binary (yes/no) flags designating whether a user may perform
  a certain task
* Groups: A generic way of applying labels and permissions to more than one
  user
* A configurable password hashing system
* Forms and view tools for logging in users, or restricting content
* A pluggable backend system

The authentication system in Django aims to be very generic and doesn't provide
some features commonly found in web authentication systems. Solutions for some
of these common problems have been implemented in third-party packages:

* Password strength checking
* Throttling of login attempts
* Authentication against third-parties (OAuth, for example)

Using the Django authentication system
======================================

Django's authentication system in its default configuration has evolved to
serve the most common project needs, handling a reasonably wide range of
tasks, and has a careful implementation of passwords and permissions. For
projects where authentication needs differ from the default, Django also supports
extensive extension and customization of authentication.

.. _user-objects:

User objects
============

:class:`~django.contrib.auth.models.User` objects are the core of the
authentication system. They typically represent the people interacting with
your site and are used to enable things like restricting access, registering
user profiles, associating content with creators etc. Only one class of user
exists in Django's authentication framework, i.e., :attr:`'superusers'
<django.contrib.auth.models.User.is_superuser>` or admin :attr:`'staff'
<django.contrib.auth.models.User.is_staff>` users are just user objects with
special attributes set, not different classes of user objects.

The primary attributes of the default user are:

* :attr:`~django.contrib.auth.models.User.username`
* :attr:`~django.contrib.auth.models.User.password`
* :attr:`~django.contrib.auth.models.User.email`
* :attr:`~django.contrib.auth.models.User.first_name`
* :attr:`~django.contrib.auth.models.User.last_name`

.. _topics-auth-creating-users:

Creating users
--------------

The simplest, and least error prone way to create and manage users is through
the Django admin. Django also provides built in views and forms to allow users
to log in and out and change their own password. 

We will be looking at user management via the admin and generic user forms a
bit later in this chapter, but first, lets look at how we would handle user
authentication directly. 

The most direct way to create users is to use the included
:meth:`~django.contrib.auth.models.UserManager.create_user` helper function::

    >>> from django.contrib.auth.models import User
    >>> user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')

    # At this point, user is a User object that has already been saved
    # to the database. You can continue to change its attributes
    # if you want to change other fields.
    >>> user.last_name = 'Lennon'
    >>> user.save()

.. _topics-auth-creating-superusers:

Creating superusers
-------------------

Create superusers using the ``createsuperuser`` command:

.. code-block:: bash

    $ python manage.py createsuperuser --username=joe --email=joe@example.com

You will be prompted for a password. After you enter one, the user will be
created immediately. If you leave off the ``--username`` or the
``--email`` options, it will prompt you for those values.

Changing passwords
------------------

Django does not store raw (clear text) passwords on the user model, but only a
hash. Because of this, do not attempt to manipulate the password attribute of
the user directly. This is why a helper function is used when creating a user.

To change a user's password, you have two options:

``manage.py changepassword *username* <changepassword>`` offers a method
of changing a User's password from the command line. It prompts you to
change the password of a given user which you must enter twice. If
they both match, the new password will be changed immediately. If you
do not supply a user, the command will attempt to change the password
of the user whose username matches the current system user.

You can also change a password programmatically, using
:meth:`~django.contrib.auth.models.User.set_password()`:

.. code-block:: pycon

    >>> from django.contrib.auth.models import User
    >>> u = User.objects.get(username='john')
    >>> u.set_password('new password')
    >>> u.save()

Changing a user's password will log out all their sessions if the
:class:`~django.contrib.auth.middleware.SessionAuthenticationMiddleware` is
enabled.

Authenticating Users
--------------------

.. function:: authenticate(\**credentials)

    To authenticate a given username and password, use
    :func:`~django.contrib.auth.authenticate()`. It takes credentials in the
    form of keyword arguments, for the default configuration this is
    ``username`` and ``password``, and it returns
    a :class:`~django.contrib.auth.models.User` object if the password is valid
    for the given username. If the password is invalid,
    :func:`~django.contrib.auth.authenticate()` returns ``None``. Example::

        from django.contrib.auth import authenticate
        user = authenticate(username='john', password='secret')
        if user is not None:
            # the password verified for the user
            if user.is_active:
                print("User is valid, active and authenticated")
            else:
                print("The password is valid, but the account has been disabled!")
        else:
            # the authentication system was unable to verify the username and password
            print("The username and password were incorrect.")

    .. note::

        This is a low level way to authenticate a set of credentials; for
        example, it's used by the
        :class:`~django.contrib.auth.middleware.RemoteUserMiddleware`. Unless
        you are writing your own authentication system, you probably won't use
        this. Rather if you are looking for a way to limit access to logged in
        users, see the :func:`~django.contrib.auth.decorators.login_required`
        decorator.

.. _topic-authorization:

Permissions and Authorization
=============================

Django comes with a simple permissions system. It provides a way to assign
permissions to specific users and groups of users.

It's used by the Django admin site, but you're welcome to use it in your own
code.

The Django admin site uses permissions as follows:

* Access to view the "add" form and add an object is limited to users with
  the "add" permission for that type of object
* Access to view the change list, view the "change" form and change an
  object is limited to users with the "change" permission for that type of
  object
* Access to delete an object is limited to users with the "delete"
  permission for that type of object

Permissions can be set not only per type of object, but also per specific
object instance. By using the
:meth:`~django.contrib.admin.ModelAdmin.has_add_permission`,
:meth:`~django.contrib.admin.ModelAdmin.has_change_permission` and
:meth:`~django.contrib.admin.ModelAdmin.has_delete_permission` methods provided
by the :class:`~django.contrib.admin.ModelAdmin` class, it is possible to
customize permissions for different object instances of the same type.

:class:`~django.contrib.auth.models.User` objects have two many-to-many
fields: ``groups`` and ``user_permissions``.
:class:`~django.contrib.auth.models.User` objects can access their related
objects in the same way as any other Django model.

Default permissions
-------------------

When ``django.contrib.auth`` is listed in your ``INSTALLED_APPS``
setting, it will ensure that three default permissions -- add, change and
delete -- are created for each Django model defined in one of your installed
applications.

These permissions will be created when you run ``manage.py migrate``
 the first time you run ``migrate`` after adding
``django.contrib.auth`` to ``INSTALLED_APPS``, the default permissions
will be created for all previously-installed models, as well as for any new
models being installed at that time. Afterward, it will create default
permissions for new models each time you run ``manage.py migrate``.

Assuming you have an application with an
:attr:`~django.db.models.Options.app_label` ``foo`` and a model named ``Bar``,
to test for basic permissions you should use:

* add: ``user.has_perm('foo.add_bar')``
* change: ``user.has_perm('foo.change_bar')``
* delete: ``user.has_perm('foo.delete_bar')``

The :class:`~django.contrib.auth.models.Permission` model is rarely accessed
directly.

Groups
------

:class:`django.contrib.auth.models.Group` models are a generic way of
categorizing users so you can apply permissions, or some other label, to those
users. A user can belong to any number of groups.

A user in a group automatically has the permissions granted to that group. For
example, if the group ``Site editors`` has the permission
``can_edit_home_page``, any user in that group will have that permission.

Beyond permissions, groups are a convenient way to categorize users to give
them some label, or extended functionality. For example, you could create a
group ``'Special users'``, and you could write code that could, say, give them
access to a members-only portion of your site, or send them members-only email
messages.

Programmatically creating permissions
-------------------------------------

While custom permissions can be defined within
a model's ``Meta`` class, you can also create permissions directly. For
example, you can create the ``can_publish`` permission for a ``BlogPost`` model
in ``myapp``::

    from myapp.models import BlogPost
    from django.contrib.auth.models import Group, Permission
    from django.contrib.contenttypes.models import ContentType

    content_type = ContentType.objects.get_for_model(BlogPost)
    permission = Permission.objects.create(codename='can_publish',
                                           name='Can Publish Posts',
                                           content_type=content_type)

The permission can then be assigned to a
:class:`~django.contrib.auth.models.User` via its ``user_permissions``
attribute or to a :class:`~django.contrib.auth.models.Group` via its
``permissions`` attribute.

Permission caching
------------------

The :class:`~django.contrib.auth.backends.ModelBackend` caches permissions on
the ``User`` object after the first time they need to be fetched for a
permissions check. This is typically fine for the request-response cycle since
permissions are not typically checked immediately after they are added (in the
admin, for example). If you are adding permissions and checking them immediately
afterward, in a test or view for example, the easiest solution is to re-fetch
the ``User`` from the database. For example::

    from django.contrib.auth.models import Permission, User
    from django.shortcuts import get_object_or_404

    def user_gains_perms(request, user_id):
        user = get_object_or_404(User, pk=user_id)
        # any permission check will cache the current set of permissions
        user.has_perm('myapp.change_bar')

        permission = Permission.objects.get(codename='change_bar')
        user.user_permissions.add(permission)

        # Checking the cached permission set
        user.has_perm('myapp.change_bar')  # False

        # Request new instance of User
        user = get_object_or_404(User, pk=user_id)

        # Permission cache is repopulated from the database
        user.has_perm('myapp.change_bar')  # True

        ...

.. _auth-web-requests:

Authentication in Web requests
==============================

Django uses sessions and middleware to hook the
authentication system into :class:`request objects <django.http.HttpRequest>`.

These provide a :attr:`request.user <django.http.HttpRequest.user>`  attribute
on every request which represents the current user. If the current user has not
logged in, this attribute will be set to an instance
of :class:`~django.contrib.auth.models.AnonymousUser`, otherwise it will be an
instance of :class:`~django.contrib.auth.models.User`.

You can tell them apart with
:meth:`~django.contrib.auth.models.User.is_authenticated()`, like so::

    if request.user.is_authenticated():
        # Do something for authenticated users.
    else:
        # Do something for anonymous users.

.. _how-to-log-a-user-in:

How to log a user in
--------------------

If you have an authenticated user you want to attach to the current session
- this is done with a :func:`~django.contrib.auth.login` function.

.. function:: login()

    To log a user in, from a view, use :func:`~django.contrib.auth.login()`. It
    takes an :class:`~django.http.HttpRequest` object and a
    :class:`~django.contrib.auth.models.User` object.
    :func:`~django.contrib.auth.login()` saves the user's ID in the session,
    using Django's session framework.

    Note that any data set during the anonymous session is retained in the
    session after a user logs in.

    This example shows how you might use both
    :func:`~django.contrib.auth.authenticate()` and
    :func:`~django.contrib.auth.login()`::

        from django.contrib.auth import authenticate, login

        def my_view(request):
            username = request.POST['username']
            password = request.POST['password']
            user = authenticate(username=username, password=password)
            if user is not None:
                if user.is_active:
                    login(request, user)
                    # Redirect to a success page.
                else:
                    # Return a 'disabled account' error message
            else:
                # Return an 'invalid login' error message.

.. admonition:: Calling ``authenticate()`` first

    When you're manually logging a user in, you *must* call
    :func:`~django.contrib.auth.authenticate()` before you call
    :func:`~django.contrib.auth.login()`.
    :func:`~django.contrib.auth.authenticate()`
    sets an attribute on the :class:`~django.contrib.auth.models.User` noting
    which authentication backend successfully authenticated that user, and
    this information is needed later during the login process. An error will be
    raised if you try to login a user object retrieved from the database
    directly.

How to log a user out
---------------------

.. function:: logout()

    To log out a user who has been logged in via
    :func:`django.contrib.auth.login()`, use
    :func:`django.contrib.auth.logout()` within your view. It takes an
    :class:`~django.http.HttpRequest` object and has no return value.
    Example::

        from django.contrib.auth import logout

        def logout_view(request):
            logout(request)
            # Redirect to a success page.

    Note that :func:`~django.contrib.auth.logout()` doesn't throw any errors if
    the user wasn't logged in.

    When you call :func:`~django.contrib.auth.logout()`, the session data for
    the current request is completely cleaned out. All existing data is
    removed. This is to prevent another person from using the same Web browser
    to log in and have access to the previous user's session data. If you want
    to put anything into the session that will be available to the user
    immediately after logging out, do that *after* calling
    :func:`django.contrib.auth.logout()`.

Limiting access to logged-in users
----------------------------------

The raw way
~~~~~~~~~~~

The simple, raw way to limit access to pages is to check
:meth:`request.user.is_authenticated()
<django.contrib.auth.models.User.is_authenticated()>` and either redirect to a
login page::

    from django.shortcuts import redirect

    def my_view(request):
        if not request.user.is_authenticated():
            return redirect('/login/?next=%s' % request.path)
        # ...

...or display an error message::

    from django.shortcuts import render

    def my_view(request):
        if not request.user.is_authenticated():
            return render(request, 'myapp/login_error.html')
        # ...

.. currentmodule:: django.contrib.auth.decorators

The login_required decorator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. function:: login_required([redirect_field_name=REDIRECT_FIELD_NAME, login_url=None])

    As a shortcut, you can use the convenient
    :func:`~django.contrib.auth.decorators.login_required` decorator::

        from django.contrib.auth.decorators import login_required

        @login_required
        def my_view(request):
            ...

    :func:`~django.contrib.auth.decorators.login_required` does the following:

    * If the user isn't logged in, redirect to
      ``LOGIN_URL``, passing the current absolute
      path in the query string. Example: ``/accounts/login/?next=/polls/3/``.

    * If the user is logged in, execute the view normally. The view code is
      free to assume the user is logged in.

    By default, the path that the user should be redirected to upon
    successful authentication is stored in a query string parameter called
    ``"next"``. If you would prefer to use a different name for this parameter,
    :func:`~django.contrib.auth.decorators.login_required` takes an
    optional ``redirect_field_name`` parameter::

        from django.contrib.auth.decorators import login_required

        @login_required(redirect_field_name='my_redirect_field')
        def my_view(request):
            ...

    Note that if you provide a value to ``redirect_field_name``, you will most
    likely need to customize your login template as well, since the template
    context variable which stores the redirect path will use the value of
    ``redirect_field_name`` as its key rather than ``"next"`` (the default).

    :func:`~django.contrib.auth.decorators.login_required` also takes an
    optional ``login_url`` parameter. Example::

        from django.contrib.auth.decorators import login_required

        @login_required(login_url='/accounts/login/')
        def my_view(request):
            ...

    Note that if you don't specify the ``login_url`` parameter, you'll need to
    ensure that the ``LOGIN_URL`` and your login
    view are properly associated. For example, using the defaults, add the
    following lines to your URLconf::

        from django.contrib.auth import views as auth_views

        url(r'^accounts/login/$', auth_views.login),

    The ``LOGIN_URL`` also accepts view function
    names and named URL patterns. This allows you
    to freely remap your login view within your URLconf without having to
    update the setting.

.. note::

    The login_required decorator does NOT check the is_active flag on a user.

Limiting access to logged-in users that pass a test
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To limit access based on certain permissions or some other test, you'd do
essentially the same thing as described in the previous section.

The simple way is to run your test on :attr:`request.user
<django.http.HttpRequest.user>` in the view directly. For example, this view
checks to make sure the user has an email in the desired domain::

    def my_view(request):
        if not request.user.email.endswith('@example.com'):
            return HttpResponse("You can't vote in this poll.")
        # ...

.. function:: user_passes_test(func, [login_url=None, redirect_field_name=REDIRECT_FIELD_NAME])

    As a shortcut, you can use the convenient ``user_passes_test`` decorator::

        from django.contrib.auth.decorators import user_passes_test

        def email_check(user):
            return user.email.endswith('@example.com')

        @user_passes_test(email_check)
        def my_view(request):
            ...

    :func:`~django.contrib.auth.decorators.user_passes_test` takes a required
    argument: a callable that takes a
    :class:`~django.contrib.auth.models.User` object and returns ``True`` if
    the user is allowed to view the page. Note that
    :func:`~django.contrib.auth.decorators.user_passes_test` does not
    automatically check that the :class:`~django.contrib.auth.models.User` is
    not anonymous.

    :func:`~django.contrib.auth.decorators.user_passes_test` takes two
    optional arguments:

    ``login_url``
       Lets you specify the URL that users who don't pass the test will be
       redirected to. It may be a login page and defaults to
       `LOGIN_URL`` if you don't specify one.

    ``redirect_field_name``
       Same as for :func:`~django.contrib.auth.decorators.login_required`.
       Setting it to ``None`` removes it from the URL, which you may want to do
       if you are redirecting users that don't pass the test to a non-login
       page where there's no "next page".

    For example::

        @user_passes_test(email_check, login_url='/login/')
        def my_view(request):
            ...

The permission_required decorator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. function:: permission_required(perm, [login_url=None, raise_exception=False])

    It's a relatively common task to check whether a user has a particular
    permission. For that reason, Django provides a shortcut for that case: the
    :func:`~django.contrib.auth.decorators.permission_required()` decorator.::

        from django.contrib.auth.decorators import permission_required

        @permission_required('polls.can_vote')
        def my_view(request):
            ...

    Just like the :meth:`~django.contrib.auth.models.User.has_perm` method,
    permission names take the form ``"<app label>.<permission codename>"``
    (i.e. ``polls.can_vote`` for a permission on a model in the ``polls``
    application).

    The decorator may also take a list of permissions.

    Note that :func:`~django.contrib.auth.decorators.permission_required()`
    also takes an optional ``login_url`` parameter. Example::

        from django.contrib.auth.decorators import permission_required

        @permission_required('polls.can_vote', login_url='/loginpage/')
        def my_view(request):
            ...

    As in the :func:`~django.contrib.auth.decorators.login_required` decorator,
    ``login_url`` defaults to `LOGIN_URL``.

    If the ``raise_exception`` parameter is given, the decorator will raise
    :exc:`~django.core.exceptions.PermissionDenied`, prompting the 403
    (HTTP Forbidden) view instead of redirecting to the
    login page.

.. _applying-permissions-to-generic-views:

Applying permissions to generic views
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To apply a permission to a class-based generic view
, decorate the :meth:`View.dispatch
<django.views.generic.base.View.dispatch>` method on the class. Another approach is to
write a mixin that wraps as_view().

.. _session-invalidation-on-password-change:

Session invalidation on password change
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your ``AUTH_USER_MODEL`` inherits from
:class:`~django.contrib.auth.models.AbstractBaseUser` or implements its own
:meth:`~django.contrib.auth.models.AbstractBaseUser.get_session_auth_hash()`
method, authenticated sessions will include the hash returned by this function.
In the :class:`~django.contrib.auth.models.AbstractBaseUser` case, this is an
HMAC of the password field. If the
:class:`~django.contrib.auth.middleware.SessionAuthenticationMiddleware` is
enabled, Django verifies that the hash sent along with each request matches
the one that's computed server-side. This allows a user to log out all of their
sessions by changing their password.

The default password change views included with Django,
:func:`django.contrib.auth.views.password_change` and the
``user_change_password`` view in the :mod:`django.contrib.auth` admin, update
the session with the new password hash so that a user changing their own
password won't log themselves out. If you have a custom password change view
and wish to have similar behavior, use this function:

.. function:: update_session_auth_hash(request, user)

    This function takes the current request and the updated user object from
    which the new session hash will be derived and updates the session hash
    appropriately. Example usage::

        from django.contrib.auth import update_session_auth_hash

        def password_change(request):
            if request.method == 'POST':
                form = PasswordChangeForm(user=request.user, data=request.POST)
                if form.is_valid():
                    form.save()
                    update_session_auth_hash(request, form.user)
            else:
                ...

.. note::

    Since
    :meth:`~django.contrib.auth.models.AbstractBaseUser.get_session_auth_hash()`
    is based on ``SECRET_KEY``, updating your site to use a new secret
    will invalidate all existing sessions.

.. _built-in-auth-views:

Authentication Views
--------------------

.. module:: django.contrib.auth.views

Django provides several views that you can use for handling login, logout, and
password management. These make use of the built-in auth forms but you can pass in your own forms as well.

Django provides no default template for the authentication views - however the
template context is documented for each view below.

The built-in views all return
a :class:`~django.template.response.TemplateResponse` instance, which allows
you to easily customize the response data before rendering. 
Most built-in authentication views provide a URL name for easier reference.

.. function:: login(request, [template_name, redirect_field_name, authentication_form, current_app, extra_context])

    **URL name:** ``login``

    See the URL documentation for details on using
    named URL patterns.

    **Optional arguments:**

    * ``template_name``: The name of a template to display for the view used to
      log the user in. Defaults to :file:`registration/login.html`.

    * ``redirect_field_name``: The name of a ``GET`` field containing the
      URL to redirect to after login. Defaults to ``next``.

    * ``authentication_form``: A callable (typically just a form class) to
      use for authentication. Defaults to
      :class:`~django.contrib.auth.forms.AuthenticationForm`.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

    Here's what ``django.contrib.auth.views.login`` does:

    * If called via ``GET``, it displays a login form that POSTs to the
      same URL. More on this in a bit.

    * If called via ``POST`` with user submitted credentials, it tries to log
      the user in. If login is successful, the view redirects to the URL
      specified in ``next``. If ``next`` isn't provided, it redirects to
      ``LOGIN_REDIRECT_URL`` (which
      defaults to ``/accounts/profile/``). If login isn't successful, it
      redisplays the login form.

    It's your responsibility to provide the html for the login template
    , called ``registration/login.html`` by default. This template gets passed
    four template context variables:

    * ``form``: A :class:`~django.forms.Form` object representing the
      :class:`~django.contrib.auth.forms.AuthenticationForm`.

    * ``next``: The URL to redirect to after successful login. This may
      contain a query string, too.

    * ``site``: The current :class:`~django.contrib.sites.models.Site`,
      according to the ``SITE_ID`` setting. If you don't have the
      site framework installed, this will be set to an instance of
      :class:`~django.contrib.sites.requests.RequestSite`, which derives the
      site name and domain from the current
      :class:`~django.http.HttpRequest`.

    * ``site_name``: An alias for ``site.name``. If you don't have the site
      framework installed, this will be set to the value of
      :attr:`request.META['SERVER_NAME'] <django.http.HttpRequest.META>`.

    If you'd prefer not to call the template :file:`registration/login.html`,
    you can pass the ``template_name`` parameter via the extra arguments to
    the view in your URLconf. For example, this URLconf line would use
    :file:`myapp/login.html` instead::

        url(r'^accounts/login/$', auth_views.login, {'template_name': 'myapp/login.html'}),

    You can also specify the name of the ``GET`` field which contains the URL
    to redirect to after login by passing ``redirect_field_name`` to the view.
    By default, the field is called ``next``.

    Here's a sample :file:`registration/login.html` template you can use as a
    starting point. It assumes you have a :file:`base.html` template that
    defines a ``content`` block:

    .. code-block:: html+django

        {% extends "base.html" %}

        {% block content %}

        {% if form.errors %}
        <p>Your username and password didn't match. Please try again.</p>
        {% endif %}

        <form method="post" action="{% url 'django.contrib.auth.views.login' %}">
        {% csrf_token %}
        <table>
        <tr>
            <td>{{ form.username.label_tag }}</td>
            <td>{{ form.username }}</td>
        </tr>
        <tr>
            <td>{{ form.password.label_tag }}</td>
            <td>{{ form.password }}</td>
        </tr>
        </table>

        <input type="submit" value="login" />
        <input type="hidden" name="next" value="{{ next }}" />
        </form>

        {% endblock %}

    If you have customized authentication you can pass a custom authentication form
    to the login view via the ``authentication_form`` parameter. This form must
    accept a ``request`` keyword argument in its ``__init__`` method, and
    provide a ``get_user`` method which returns the authenticated user object
    (this method is only ever called after successful form validation).

    .. _forms documentation: ../forms/
    .. _site framework docs: ../sites/


.. function:: logout(request, [next_page, template_name, redirect_field_name, current_app, extra_context])

    Logs a user out.

    **URL name:** ``logout``

    **Optional arguments:**

    * ``next_page``: The URL to redirect to after logout.

    * ``template_name``: The full name of a template to display after
      logging the user out. Defaults to
      :file:`registration/logged_out.html` if no argument is supplied.

    * ``redirect_field_name``: The name of a ``GET`` field containing the
      URL to redirect to after log out. Defaults to ``next``. Overrides the
      ``next_page`` URL if the given ``GET`` parameter is passed.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

    **Template context:**

    * ``title``: The string "Logged out", localized.

    * ``site``: The current :class:`~django.contrib.sites.models.Site`,
      according to the ``SITE_ID`` setting. If you don't have the
      site framework installed, this will be set to an instance of
      :class:`~django.contrib.sites.requests.RequestSite`, which derives the
      site name and domain from the current
      :class:`~django.http.HttpRequest`.

    * ``site_name``: An alias for ``site.name``. If you don't have the site
      framework installed, this will be set to the value of
      :attr:`request.META['SERVER_NAME'] <django.http.HttpRequest.META>`.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

.. function:: logout_then_login(request[, login_url, current_app, extra_context])

    Logs a user out, then redirects to the login page.

    **URL name:** No default URL provided

    **Optional arguments:**

    * ``login_url``: The URL of the login page to redirect to.
      Defaults to ``LOGIN_URL`` if not supplied.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

.. function:: password_change(request[, template_name, post_change_redirect, password_change_form, current_app, extra_context])

    Allows a user to change their password.

    **URL name:** ``password_change``

    **Optional arguments:**

    * ``template_name``: The full name of a template to use for
      displaying the password change form. Defaults to
      :file:`registration/password_change_form.html` if not supplied.

    * ``post_change_redirect``: The URL to redirect to after a successful
      password change.

    * ``password_change_form``: A custom "change password" form which must
      accept a ``user`` keyword argument. The form is responsible for
      actually changing the user's password. Defaults to
      :class:`~django.contrib.auth.forms.PasswordChangeForm`.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

    **Template context:**

    * ``form``: The password change form (see ``password_change_form`` above).

.. function:: password_change_done(request[, template_name, current_app, extra_context])

    The page shown after a user has changed their password.

    **URL name:** ``password_change_done``

    **Optional arguments:**

    * ``template_name``: The full name of a template to use.
      Defaults to :file:`registration/password_change_done.html` if not
      supplied.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

.. function:: password_reset(request[, template_name, email_template_name, password_reset_form, token_generator, post_reset_redirect, from_email, current_app, extra_context, html_email_template_name])

    Allows a user to reset their password by generating a one-time use link
    that can be used to reset the password, and sending that link to the
    user's registered email address.

    If the email address provided does not exist in the system, this view
    won't send an email, but the user won't receive any error message either.
    This prevents information leaking to potential attackers. If you want to
    provide an error message in this case, you can subclass
    :class:`~django.contrib.auth.forms.PasswordResetForm` and use the
    ``password_reset_form`` argument.


    Users flagged with an unusable password (see
    :meth:`~django.contrib.auth.models.User.set_unusable_password()` aren't
    allowed to request a password reset to prevent misuse when using an
    external authentication source like LDAP. Note that they won't receive any
    error message since this would expose their account's existence but no
    mail will be sent either.

    **URL name:** ``password_reset``

    **Optional arguments:**

    * ``template_name``: The full name of a template to use for
      displaying the password reset form. Defaults to
      :file:`registration/password_reset_form.html` if not supplied.

    * ``email_template_name``: The full name of a template to use for
      generating the email with the reset password link. Defaults to
      :file:`registration/password_reset_email.html` if not supplied.

    * ``subject_template_name``: The full name of a template to use for
      the subject of the email with the reset password link. Defaults
      to :file:`registration/password_reset_subject.txt` if not supplied.

    * ``password_reset_form``: Form that will be used to get the email of
      the user to reset the password for. Defaults to
      :class:`~django.contrib.auth.forms.PasswordResetForm`.

    * ``token_generator``: Instance of the class to check the one time link.
      This will default to ``default_token_generator``, it's an instance of
      ``django.contrib.auth.tokens.PasswordResetTokenGenerator``.

    * ``post_reset_redirect``: The URL to redirect to after a successful
      password reset request.

    * ``from_email``: A valid email address. By default Django uses
      the ``DEFAULT_FROM_EMAIL``.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

    * ``html_email_template_name``: The full name of a template to use
      for generating a ``text/html`` multipart email with the password reset
      link. By default, HTML email is not sent.

    **Template context:**

    * ``form``: The form (see ``password_reset_form`` above) for resetting
      the user's password.

    **Email template context:**

    * ``email``: An alias for ``user.email``

    * ``user``: The current :class:`~django.contrib.auth.models.User`,
      according to the ``email`` form field. Only active users are able to
      reset their passwords (``User.is_active is True``).

    * ``site_name``: An alias for ``site.name``. If you don't have the site
      framework installed, this will be set to the value of
      :attr:`request.META['SERVER_NAME'] <django.http.HttpRequest.META>`.

    * ``domain``: An alias for ``site.domain``. If you don't have the site
      framework installed, this will be set to the value of
      ``request.get_host()``.

    * ``protocol``: http or https

    * ``uid``: The user's primary key encoded in base 64.

    * ``token``: Token to check that the reset link is valid.

    Sample ``registration/password_reset_email.html`` (email body template):

    .. code-block:: html+django

        Someone asked for password reset for email {{ email }}. Follow the link below:
        {{ protocol}}://{{ domain }}{% url 'password_reset_confirm' uidb64=uid token=token %}

    The same template context is used for subject template. Subject must be
    single line plain text string.

.. function:: password_reset_done(request[, template_name, current_app, extra_context])

    The page shown after a user has been emailed a link to reset their
    password. This view is called by default if the :func:`password_reset` view
    doesn't have an explicit ``post_reset_redirect`` URL set.

    **URL name:** ``password_reset_done``

    .. note::

        If the email address provided does not exist in the system, the user is inactive, or has an unusable password,
        the user will still be redirected to this view but no email will be sent.

    **Optional arguments:**

    * ``template_name``: The full name of a template to use.
      Defaults to :file:`registration/password_reset_done.html` if not
      supplied.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

.. function:: password_reset_confirm(request[, uidb64, token, template_name, token_generator, set_password_form, post_reset_redirect, current_app, extra_context])

    Presents a form for entering a new password.

    **URL name:** ``password_reset_confirm``

    **Optional arguments:**

    * ``uidb64``: The user's id encoded in base 64. Defaults to ``None``.

    * ``token``: Token to check that the password is valid. Defaults to
      ``None``.

    * ``template_name``: The full name of a template to display the confirm
      password view. Default value is :file:`registration/password_reset_confirm.html`.

    * ``token_generator``: Instance of the class to check the password. This
      will default to ``default_token_generator``, it's an instance of
      ``django.contrib.auth.tokens.PasswordResetTokenGenerator``.

    * ``set_password_form``: Form that will be used to set the password.
      Defaults to :class:`~django.contrib.auth.forms.SetPasswordForm`

    * ``post_reset_redirect``: URL to redirect after the password reset
      done. Defaults to ``None``.

    * ``current_app``: A hint indicating which application contains the current
      view. See the namespaced URL resolution strategy for more information.

    * ``extra_context``: A dictionary of context data that will be added to the
      default context data passed to the template.

    **Template context:**

    * ``form``: The form (see ``set_password_form`` above) for setting the
      new user's password.

    * ``validlink``: Boolean, True if the link (combination of ``uidb64`` and
      ``token``) is valid or unused yet.

.. function:: password_reset_complete(request[,template_name, current_app, extra_context])

   Presents a view which informs the user that the password has been
   successfully changed.

   **URL name:** ``password_reset_complete``

   **Optional arguments:**

   * ``template_name``: The full name of a template to display the view.
     Defaults to :file:`registration/password_reset_complete.html`.

   * ``current_app``: A hint indicating which application contains the current
     view. See the namespaced URL resolution strategy for more information.

   * ``extra_context``: A dictionary of context data that will be added to the
     default context data passed to the template.

Helper functions
----------------

.. currentmodule:: django.contrib.auth.views

.. function:: redirect_to_login(next[, login_url, redirect_field_name])

    Redirects to the login page, and then back to another URL after a
    successful login.

    **Required arguments:**

    * ``next``: The URL to redirect to after a successful login.

    **Optional arguments:**

    * ``login_url``: The URL of the login page to redirect to.
      Defaults to ``LOGIN_URL`` if not supplied.

    * ``redirect_field_name``: The name of a ``GET`` field containing the
      URL to redirect to after log out. Overrides ``next`` if the given
      ``GET`` parameter is passed.


.. _built-in-auth-forms:

Built-in forms
--------------

.. module:: django.contrib.auth.forms

If you don't want to use the built-in views, but want the convenience of not
having to write forms for this functionality, the authentication system
provides several built-in forms located in :mod:`django.contrib.auth.forms`:

.. note::
    The built-in authentication forms make certain assumptions about the user
    model that they are working with. If you're using a custom User model
    , it may be necessary to define your own forms for the
    authentication system. 

.. class:: AdminPasswordChangeForm

    A form used in the admin interface to change a user's password.

    Takes the ``user`` as the first positional argument.

.. class:: AuthenticationForm

    A form for logging a user in.

    Takes ``request`` as its first positional argument, which is stored on the
    form instance for use by sub-classes.

    .. method:: confirm_login_allowed(user)

        By default, ``AuthenticationForm`` rejects users whose ``is_active`` flag
        is set to ``False``. You may override this behavior with a custom policy to
        determine which users can log in. Do this with a custom form that subclasses
        ``AuthenticationForm`` and overrides the ``confirm_login_allowed`` method.
        This method should raise a :exc:`~django.core.exceptions.ValidationError`
        if the given user may not log in.

        For example, to allow all users to log in, regardless of "active" status::

            from django.contrib.auth.forms import AuthenticationForm

            class AuthenticationFormWithInactiveUsersOkay(AuthenticationForm):
                def confirm_login_allowed(self, user):
                    pass

        Or to allow only some active users to log in::

            class PickyAuthenticationForm(AuthenticationForm):
                def confirm_login_allowed(self, user):
                    if not user.is_active:
                        raise forms.ValidationError(
                            _("This account is inactive."),
                            code='inactive',
                        )
                    if user.username.startswith('b'):
                        raise forms.ValidationError(
                            _("Sorry, accounts starting with 'b' aren't welcome here."),
                            code='no_b_users',
                        )

.. class:: PasswordChangeForm

    A form for allowing a user to change their password.

.. class:: PasswordResetForm

    A form for generating and emailing a one-time use link to reset a
    user's password.

    .. method:: send_email(subject_template_name, email_template_name, context, from_email, to_email, [html_email_template_name=None])

        .. versionadded:: 1.8

        Uses the arguments to send an ``EmailMultiAlternatives``.
        Can be overridden to customize how the email is sent to the user.

        :param subject_template_name: the template for the subject.
        :param email_template_name: the template for the email body.
        :param context: context passed to the ``subject_template``, ``email_template``,
            and ``html_email_template`` (if it is not ``None``).
        :param from_email: the sender's email.
        :param to_email: the email of the requester.
        :param html_email_template_name: the template for the HTML body;
            defaults to ``None``, in which case a plain text email is sent.

        By default, ``save()`` populates the ``context`` with the
        same variables that :func:`~django.contrib.auth.views.password_reset`
        passes to its email context.

.. class:: SetPasswordForm

    A form that lets a user change their password without entering the old
    password.

.. class:: UserChangeForm

    A form used in the admin interface to change a user's information and
    permissions.

.. class:: UserCreationForm

    A form for creating a new user.

.. currentmodule:: django.contrib.auth


Authentication data in templates
--------------------------------

The currently logged-in user and their permissions are made available in the
template context when you use
:class:`~django.template.RequestContext`.

Users
~~~~~

When rendering a template :class:`~django.template.RequestContext`, the
currently logged-in user, either a  :class:`~django.contrib.auth.models.User`
instance or an :class:`~django.contrib.auth.models.AnonymousUser` instance, is
stored in the template variable ``{{ user }}``:

.. code-block:: html+django

    {% if user.is_authenticated %}
        <p>Welcome, {{ user.username }}. Thanks for logging in.</p>
    {% else %}
        <p>Welcome, new user. Please log in.</p>
    {% endif %}

This template context variable is not available if a ``RequestContext`` is not
being used.

Permissions
~~~~~~~~~~~

The currently logged-in user's permissions are stored in the template variable
``{{ perms }}``. This is an instance of
``django.contrib.auth.context_processors.PermWrapper``, which is a
template-friendly proxy of permissions.

In the ``{{ perms }}`` object, single-attribute lookup is a proxy to
:meth:`User.has_module_perms <django.contrib.auth.models.User.has_module_perms>`.
This example would display ``True`` if the logged-in user had any permissions
in the ``foo`` app::

    {{ perms.foo }}

Two-level-attribute lookup is a proxy to
:meth:`User.has_perm <django.contrib.auth.models.User.has_perm>`. This example
would display ``True`` if the logged-in user had the permission
``foo.can_vote``::

    {{ perms.foo.can_vote }}

Thus, you can check permissions in template ``{% if %}`` statements:

.. code-block:: html+django

    {% if perms.foo %}
        <p>You have permission to do something in the foo app.</p>
        {% if perms.foo.can_vote %}
            <p>You can vote!</p>
        {% endif %}
        {% if perms.foo.can_drive %}
            <p>You can drive!</p>
        {% endif %}
    {% else %}
        <p>You don't have permission to do anything in the foo app.</p>
    {% endif %}

It is possible to also look permissions up by ``{% if in %}`` statements.
For example:

.. code-block:: html+django

    {% if 'foo' in perms %}
        {% if 'foo.can_vote' in perms %}
            <p>In lookup works, too.</p>
        {% endif %}
    {% endif %}

.. _auth-admin:

Managing users in the admin
===========================

When you have both ``django.contrib.admin`` and ``django.contrib.auth``
installed, the admin provides a convenient way to view and manage users,
groups, and permissions. Users can be created and deleted like any Django
model. Groups can be created, and permissions can be assigned to users or
groups. A log of user edits to models made within the admin is also stored and
displayed.

[TODO add screenshots in here]

Creating Users
--------------

You should see a link to "Users" in the "Auth"
section of the main admin index page. The "Add user" admin page is different
than standard admin pages in that it requires you to choose a username and
password before allowing you to edit the rest of the user's fields.

Also note: if you want a user account to be able to create users using the
Django admin site, you'll need to give them permission to add users *and*
change users (i.e., the "Add user" and "Change user" permissions). If an
account has permission to add users but not to change them, that account won't
be able to add users. Why? Because if you have permission to add users, you
have the power to create superusers, which can then, in turn, change other
users. So Django requires add *and* change permissions as a slight security
measure.

Changing Passwords
------------------

User passwords are not displayed in the admin (nor stored in the database), but
the password storage details are displayed.
Included in the display of this information is a link to
a password change form that allows admins to change user passwords.

Password management in Django
=============================

Password management is something that should generally not be reinvented
unnecessarily, and Django endeavors to provide a secure and flexible set of
tools for managing user passwords. This document describes how Django stores
passwords, how the storage hashing can be configured, and some utilities to
work with hashed passwords.

.. _auth_password_storage:

How Django stores passwords
---------------------------

Django provides a flexible password storage system and uses PBKDF2 by default.

The :attr:`~django.contrib.auth.models.User.password` attribute of a
:class:`~django.contrib.auth.models.User` object is a string in this format::

    <algorithm>$<iterations>$<salt>$<hash>

Those are the components used for storing a User's password, separated by the
dollar-sign character and consist of: the hashing algorithm, the number of
algorithm iterations (work factor), the random salt, and the resulting password
hash.  The algorithm is one of a number of one-way hashing or password storage
algorithms Django can use; see below. Iterations describe the number of times
the algorithm is run over the hash. Salt is the random seed used and the hash
is the result of the one-way function.

By default, Django uses the PBKDF2_ algorithm with a SHA256 hash, a
password stretching mechanism recommended by NIST_. This should be
sufficient for most users: it's quite secure, requiring massive
amounts of computing time to break.

However, depending on your requirements, you may choose a different
algorithm, or even use a custom algorithm to match your specific
security situation. Again, most users shouldn't need to do this -- if
you're not sure, you probably don't.  If you do, please read on:

Django chooses the algorithm to use by consulting the
``PASSWORD_HASHERS`` setting. This is a list of hashing algorithm
classes that this Django installation supports. The first entry in this list
(that is, ``settings.PASSWORD_HASHERS[0]``) will be used to store passwords,
and all the other entries are valid hashers that can be used to check existing
passwords.  This means that if you want to use a different algorithm, you'll
need to modify ``PASSWORD_HASHERS`` to list your preferred algorithm
first in the list.

The default for ``PASSWORD_HASHERS`` is::

    PASSWORD_HASHERS = [
        'django.contrib.auth.hashers.PBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
        'django.contrib.auth.hashers.BCryptPasswordHasher',
        'django.contrib.auth.hashers.SHA1PasswordHasher',
        'django.contrib.auth.hashers.MD5PasswordHasher',
        'django.contrib.auth.hashers.CryptPasswordHasher',
    ]

This means that Django will use PBKDF2_ to store all passwords, but will support
checking passwords stored with PBKDF2SHA1, bcrypt_, SHA1_, etc. The next few
sections describe a couple of common ways advanced users may want to modify this
setting.

.. _bcrypt_usage:

Using bcrypt with Django
------------------------

Bcrypt_ is a popular password storage algorithm that's specifically designed
for long-term password storage. It's not the default used by Django since it
requires the use of third-party libraries, but since many people may want to
use it Django supports bcrypt with minimal effort.

To use Bcrypt as your default storage algorithm, do the following:

1. Install the `bcrypt library`_. This can be done by running ``pip install
   django[bcrypt]``, or by downloading the library and installing it with
   ``python setup.py install``.

2. Modify ``PASSWORD_HASHERS`` to list ``BCryptSHA256PasswordHasher``
   first. That is, in your settings file, you'd put::

        PASSWORD_HASHERS = [
            'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
            'django.contrib.auth.hashers.BCryptPasswordHasher',
            'django.contrib.auth.hashers.PBKDF2PasswordHasher',
            'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
            'django.contrib.auth.hashers.SHA1PasswordHasher',
            'django.contrib.auth.hashers.MD5PasswordHasher',
            'django.contrib.auth.hashers.CryptPasswordHasher',
        ]

   (You need to keep the other entries in this list, or else Django won't
   be able to upgrade passwords; see below).

That's it -- now your Django install will use bcrypt as the default storage
algorithm.

.. admonition:: Password truncation with BCryptPasswordHasher

    The designers of bcrypt truncate all passwords at 72 characters which means
    that ``bcrypt(password_with_100_chars) == bcrypt(password_with_100_chars[:72])``.
    The original ``BCryptPasswordHasher`` does not have any special handling and
    thus is also subject to this hidden password length limit.
    ``BCryptSHA256PasswordHasher`` fixes this by first first hashing the
    password using sha256. This prevents the password truncation and so should
    be preferred over the ``BCryptPasswordHasher``. The practical ramification
    of this truncation is pretty marginal as the average user does not have a
    password greater than 72 characters in length and even being truncated at 72
    the compute powered required to brute force bcrypt in any useful amount of
    time is still astronomical. Nonetheless, we recommend you use
    ``BCryptSHA256PasswordHasher`` anyway on the principle of "better safe than
    sorry".

.. admonition:: Other bcrypt implementations

   There are several other implementations that allow bcrypt to be
   used with Django. Django's bcrypt support is NOT directly
   compatible with these. To upgrade, you will need to modify the
   hashes in your database to be in the form ``bcrypt$(raw bcrypt
   output)``. For example:
   ``bcrypt$$2a$12$NT0I31Sa7ihGEWpka9ASYrEFkhuTNeBQ2xfZskIiiJeyFXhRgS.Sy``.

.. _increasing-password-algorithm-work-factor:

Increasing the work factor
--------------------------

The PBKDF2 and bcrypt algorithms use a number of iterations or rounds of
hashing. This deliberately slows down attackers, making attacks against hashed
passwords harder. However, as computing power increases, the number of
iterations needs to be increased. We've chosen a reasonable default (and will
increase it with each release of Django), but you may wish to tune it up or
down, depending on your security needs and available processing power. To do so,
you'll subclass the appropriate algorithm and override the ``iterations``
parameters. For example, to increase the number of iterations used by the
default PBKDF2 algorithm:

1. Create a subclass of ``django.contrib.auth.hashers.PBKDF2PasswordHasher``::

        from django.contrib.auth.hashers import PBKDF2PasswordHasher

        class MyPBKDF2PasswordHasher(PBKDF2PasswordHasher):
            """
            A subclass of PBKDF2PasswordHasher that uses 100 times more iterations.
            """
            iterations = PBKDF2PasswordHasher.iterations * 100

   Save this somewhere in your project. For example, you might put this in
   a file like ``myproject/hashers.py``.

2. Add your new hasher as the first entry in ``PASSWORD_HASHERS``::

        PASSWORD_HASHERS = [
            'myproject.hashers.MyPBKDF2PasswordHasher',
            'django.contrib.auth.hashers.PBKDF2PasswordHasher',
            'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
            'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
            'django.contrib.auth.hashers.BCryptPasswordHasher',
            'django.contrib.auth.hashers.SHA1PasswordHasher',
            'django.contrib.auth.hashers.MD5PasswordHasher',
            'django.contrib.auth.hashers.CryptPasswordHasher',
        ]


That's it -- now your Django install will use more iterations when it
stores passwords using PBKDF2.

.. _password-upgrades:

Password upgrading
------------------

When users log in, if their passwords are stored with anything other than
the preferred algorithm, Django will automatically upgrade the algorithm
to the preferred one. This means that old installs of Django will get
automatically more secure as users log in, and it also means that you
can switch to new (and better) storage algorithms as they get invented.

However, Django can only upgrade passwords that use algorithms mentioned in
``PASSWORD_HASHERS``, so as you upgrade to new systems you should make
sure never to *remove* entries from this list. If you do, users using
unmentioned algorithms won't be able to upgrade. Passwords will be upgraded
when changing the PBKDF2 iteration count.

.. _sha1: http://en.wikipedia.org/wiki/SHA1
.. _pbkdf2: http://en.wikipedia.org/wiki/PBKDF2
.. _nist: http://csrc.nist.gov/publications/nistpubs/800-132/nist-sp800-132.pdf
.. _bcrypt: http://en.wikipedia.org/wiki/Bcrypt
.. _`bcrypt library`: https://pypi.python.org/pypi/bcrypt/


Manually managing a user's password
===================================

.. module:: django.contrib.auth.hashers

The :mod:`django.contrib.auth.hashers` module provides a set of functions
to create and validate hashed password. You can use them independently
from the ``User`` model.

.. function:: check_password(password, encoded)

    If you'd like to manually authenticate a user by comparing a plain-text
    password to the hashed password in the database, use the convenience
    function :func:`check_password`. It takes two arguments: the plain-text
    password to check, and the full value of a user's ``password`` field in the
    database to check against, and returns ``True`` if they match, ``False``
    otherwise.

.. function:: make_password(password, salt=None, hasher='default')

    Creates a hashed password in the format used by this application. It takes
    one mandatory argument: the password in plain-text. Optionally, you can
    provide a salt and a hashing algorithm to use, if you don't want to use the
    defaults (first entry of ``PASSWORD_HASHERS`` setting).
    Currently supported algorithms are: ``'pbkdf2_sha256'``, ``'pbkdf2_sha1'``,
    ``'bcrypt_sha256'``, ``'bcrypt'``, ``'sha1'``,
    ``'md5'``, ``'unsalted_md5'`` (only for backward compatibility) and ``'crypt'``
    if you have the ``crypt`` library installed. If the password argument is
    ``None``, an unusable password is returned (a one that will be never
    accepted by :func:`check_password`).

.. function:: is_password_usable(encoded_password)

   Checks if the given string is a hashed password that has a chance
   of being verified against :func:`check_password`.

Customizing authentication in Django
====================================

The authentication that comes with Django is good enough for most common cases,
but you may have needs not met by the out-of-the-box defaults. To customize
authentication to your projects needs involves understanding what points of the
provided system are extensible or replaceable. This document provides details
about how the auth system can be customized.

`Authentication backends` provide an extensible
system for when a username and password stored with the User model need
to be authenticated against a different service than Django's default.

You can give your models custom permissions that can be
checked through Django's authorization system.

You can extend the default User model, or substitute
a completely customized model.

.. _authentication-backends:

Other authentication sources
============================

There may be times you have the need to hook into another authentication source
-- that is, another source of usernames and passwords or authentication
methods.

For example, your company may already have an LDAP setup that stores a username
and password for every employee. It'd be a hassle for both the network
administrator and the users themselves if users had separate accounts in LDAP
and the Django-based applications.

So, to handle situations like this, the Django authentication system lets you
plug in other authentication sources. You can override Django's default
database-based scheme, or you can use the default system in tandem with other
systems.

Specifying authentication backends
----------------------------------

Behind the scenes, Django maintains a list of "authentication backends" that it
checks for authentication. When somebody calls
:func:`django.contrib.auth.authenticate()` -- as described in the previous
section on logging a user in -- Django tries authenticating across
all of its authentication backends. If the first authentication method fails,
Django tries the second one, and so on, until all backends have been attempted.

The list of authentication backends to use is specified in the
``AUTHENTICATION_BACKENDS`` setting. This should be a list of Python
path names that point to Python classes that know how to authenticate. These
classes can be anywhere on your Python path.

By default, ``AUTHENTICATION_BACKENDS`` is set to::

    ['django.contrib.auth.backends.ModelBackend']

That's the basic authentication backend that checks the Django users database
and queries the built-in permissions. It does not provide protection against
brute force attacks via any rate limiting mechanism. You may either implement
your own rate limiting mechanism in a custom auth backend, or use the
mechanisms provided by most Web servers.

The order of ``AUTHENTICATION_BACKENDS`` matters, so if the same
username and password is valid in multiple backends, Django will stop
processing at the first positive match.

If a backend raises a :class:`~django.core.exceptions.PermissionDenied`
exception, authentication will immediately fail. Django won't check the
backends that follow.

.. note::

    Once a user has authenticated, Django stores which backend was used to
    authenticate the user in the user's session, and re-uses the same backend
    for the duration of that session whenever access to the currently
    authenticated user is needed. This effectively means that authentication
    sources are cached on a per-session basis, so if you change
    ``AUTHENTICATION_BACKENDS``, you'll need to clear out session data if
    you need to force users to re-authenticate using different methods. A simple
    way to do that is simply to execute ``Session.objects.all().delete()``.

Writing an authentication backend
---------------------------------

An authentication backend is a class that implements two required methods:
``get_user(user_id)`` and ``authenticate(**credentials)``, as well as a set of
optional permission related authorization methods.

The ``get_user`` method takes a ``user_id`` -- which could be a username,
database ID or whatever, but has to be the primary key of your ``User`` object
-- and returns a ``User`` object.

The ``authenticate`` method takes credentials as keyword arguments. Most of
the time, it'll just look like this::

    class MyBackend(object):
        def authenticate(self, username=None, password=None):
            # Check the username/password and return a User.
            ...

But it could also authenticate a token, like so::

    class MyBackend(object):
        def authenticate(self, token=None):
            # Check the token and return a User.
            ...

Either way, ``authenticate`` should check the credentials it gets, and it
should return a ``User`` object that matches those credentials, if the
credentials are valid. If they're not valid, it should return ``None``.

The Django admin system is tightly coupled to the Django ``User`` object
described at the beginning of this document. For now, the best way to deal with
this is to create a Django ``User`` object for each user that exists for your
backend (e.g., in your LDAP directory, your external SQL database, etc.) You
can either write a script to do this in advance, or your ``authenticate``
method can do it the first time a user logs in.

Here's an example backend that authenticates against a username and password
variable defined in your ``settings.py`` file and creates a Django ``User``
object the first time a user authenticates::

    from django.conf import settings
    from django.contrib.auth.models import User, check_password

    class SettingsBackend(object):
        """
        Authenticate against the settings ADMIN_LOGIN and ADMIN_PASSWORD.

        Use the login name, and a hash of the password. For example:

        ADMIN_LOGIN = 'admin'
        ADMIN_PASSWORD = 'sha1$4e987$afbcf42e21bd417fb71db8c66b321e9fc33051de'
        """

        def authenticate(self, username=None, password=None):
            login_valid = (settings.ADMIN_LOGIN == username)
            pwd_valid = check_password(password, settings.ADMIN_PASSWORD)
            if login_valid and pwd_valid:
                try:
                    user = User.objects.get(username=username)
                except User.DoesNotExist:
                    # Create a new user. Note that we can set password
                    # to anything, because it won't be checked; the password
                    # from settings.py will.
                    user = User(username=username, password='get from settings.py')
                    user.is_staff = True
                    user.is_superuser = True
                    user.save()
                return user
            return None

        def get_user(self, user_id):
            try:
                return User.objects.get(pk=user_id)
            except User.DoesNotExist:
                return None

.. _authorization_methods:

Handling authorization in custom backends
-----------------------------------------

Custom auth backends can provide their own permissions.

The user model will delegate permission lookup functions
(:meth:`~django.contrib.auth.models.User.get_group_permissions()`,
:meth:`~django.contrib.auth.models.User.get_all_permissions()`,
:meth:`~django.contrib.auth.models.User.has_perm()`, and
:meth:`~django.contrib.auth.models.User.has_module_perms()`) to any
authentication backend that implements these functions.

The permissions given to the user will be the superset of all permissions
returned by all backends. That is, Django grants a permission to a user that
any one backend grants.

If a backend raises a :class:`~django.core.exceptions.PermissionDenied`
exception in :meth:`~django.contrib.auth.models.User.has_perm()` or
:meth:`~django.contrib.auth.models.User.has_module_perms()`,
the authorization will immediately fail and Django
won't check the backends that follow.

The simple backend above could implement permissions for the magic admin
fairly simply::

    class SettingsBackend(object):
        ...
        def has_perm(self, user_obj, perm, obj=None):
            if user_obj.username == settings.ADMIN_LOGIN:
                return True
            else:
                return False

This gives full permissions to the user granted access in the above example.
Notice that in addition to the same arguments given to the associated
:class:`django.contrib.auth.models.User` functions, the backend auth functions
all take the user object, which may be an anonymous user, as an argument.

A full authorization implementation can be found in the ``ModelBackend`` class
in `django/contrib/auth/backends.py`_, which is the default backend and queries
the ``auth_permission`` table most of the time. If you wish to provide
custom behavior for only part of the backend API, you can take advantage of
Python inheritance and subclass ``ModelBackend`` instead of implementing the
complete API in a custom backend.

.. _django/contrib/auth/backends.py: https://github.com/django/django/blob/master/django/contrib/auth/backends.py

.. _anonymous_auth:

Authorization for anonymous users
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An anonymous user is one that is not authenticated i.e. they have provided no
valid authentication details. However, that does not necessarily mean they are
not authorized to do anything. At the most basic level, most Web sites
authorize anonymous users to browse most of the site, and many allow anonymous
posting of comments etc.

Django's permission framework does not have a place to store permissions for
anonymous users. However, the user object passed to an authentication backend
may be an :class:`django.contrib.auth.models.AnonymousUser` object, allowing
the backend to specify custom authorization behavior for anonymous users. This
is especially useful for the authors of re-usable apps, who can delegate all
questions of authorization to the auth backend, rather than needing settings,
for example, to control anonymous access.

.. _inactive_auth:

Authorization for inactive users
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An inactive user is a one that is authenticated but has its attribute
``is_active`` set to ``False``. However this does not mean they are not
authorized to do anything. For example they are allowed to activate their
account.

The support for anonymous users in the permission system allows for a scenario
where anonymous users have permissions to do something while inactive
authenticated users do not.

Do not forget to test for the ``is_active`` attribute of the user in your own
backend permission methods.


Handling object permissions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Django's permission framework has a foundation for object permissions, though
there is no implementation for it in the core. That means that checking for
object permissions will always return ``False`` or an empty list (depending on
the check performed). An authentication backend will receive the keyword
parameters ``obj`` and ``user_obj`` for each object related authorization
method and can return the object level permission as appropriate.

.. _custom-permissions:

Custom permissions
==================

To create custom permissions for a given model object, use the ``permissions``
model Meta attribute.

This example Task model creates three custom permissions, i.e., actions users
can or cannot do with Task instances, specific to your application::

    class Task(models.Model):
        ...
        class Meta:
            permissions = (
                ("view_task", "Can see available tasks"),
                ("change_task_status", "Can change the status of tasks"),
                ("close_task", "Can remove a task by setting its status as closed"),
            )

The only thing this does is create those extra permissions when you run
``manage.py migrate``. Your code is in charge of checking the
value of these permissions when a user is trying to access the functionality
provided by the application (viewing tasks, changing the status of tasks,
closing tasks.) Continuing the above example, the following checks if a user may
view tasks::

    user.has_perm('app.view_task')

.. _extending-user:

Extending the existing User model
=================================

There are two ways to extend the default
:class:`~django.contrib.auth.models.User` model without substituting your own
model. If the changes you need are purely behavioral, and don't require any
change to what is stored in the database, you can create a proxy model based
on :class:`~django.contrib.auth.models.User`. This allows for any of the
features offered by proxy models including default
ordering, custom managers, or custom model methods.

If you wish to store information related to ``User``, you can use a one-to-one
relationship to a model containing the fields for
additional information. This one-to-one model is often called a profile model,
as it might store non-auth related information about a site user. For example
you might create an Employee model::

    from django.contrib.auth.models import User

    class Employee(models.Model):
        user = models.OneToOneField(User)
        department = models.CharField(max_length=100)

Assuming an existing Employee Fred Smith who has both a User and Employee
model, you can access the related information using Django's standard related
model conventions::

    >>> u = User.objects.get(username='fsmith')
    >>> freds_department = u.employee.department

To add a profile model's fields to the user page in the admin, define an
:class:`~django.contrib.admin.InlineModelAdmin` (for this example, we'll use a
:class:`~django.contrib.admin.StackedInline`) in your app's ``admin.py`` and
add it to a ``UserAdmin`` class which is registered with the
:class:`~django.contrib.auth.models.User` class::

    from django.contrib import admin
    from django.contrib.auth.admin import UserAdmin
    from django.contrib.auth.models import User

    from my_user_profile_app.models import Employee

    # Define an inline admin descriptor for Employee model
    # which acts a bit like a singleton
    class EmployeeInline(admin.StackedInline):
        model = Employee
        can_delete = False
        verbose_name_plural = 'employee'

    # Define a new User admin
    class UserAdmin(UserAdmin):
        inlines = (EmployeeInline, )

    # Re-register UserAdmin
    admin.site.unregister(User)
    admin.site.register(User, UserAdmin)

These profile models are not special in any way - they are just Django models that
happen to have a one-to-one link with a User model. As such, they do not get
auto created when a user is created, but
a :attr:`django.db.models.signals.post_save` could be used to create or update
related models as appropriate.

Note that using related models results in additional queries or joins to
retrieve the related data, and depending on your needs substituting the User
model and adding the related fields may be your better option.  However
existing links to the default User model within your project's apps may justify
the extra database load.

.. _auth-custom-user:

Substituting a custom User model
================================

Some kinds of projects may have authentication requirements for which Django's
built-in :class:`~django.contrib.auth.models.User` model is not always
appropriate. For instance, on some sites it makes more sense to use an email
address as your identification token instead of a username.

Django allows you to override the default User model by providing a value for
the ``AUTH_USER_MODEL`` setting that references a custom model::

     AUTH_USER_MODEL = 'myapp.MyUser'

This dotted pair describes the name of the Django app (which must be in your
``INSTALLED_APPS``), and the name of the Django model that you wish to
use as your User model.

.. warning::

   Changing ``AUTH_USER_MODEL`` has a big effect on your database
   structure. It changes the tables that are available, and it will affect the
   construction of foreign keys and many-to-many relationships. If you intend
   to set ``AUTH_USER_MODEL``, you should set it before creating
   any migrations or running ``manage.py migrate`` for the first time.

   Changing this setting after you have tables created is not supported
   by ``makemigrations`` and will result in you having to manually
   fix your schema, port your data from the old user table, and possibly
   manually reapply some migrations.

.. warning::

   Due to limitations of Django's dynamic dependency feature for swappable
   models, you must ensure that the model referenced by ``AUTH_USER_MODEL``
   is created in the first migration of its app (usually called ``0001_initial``);
   otherwise, you will have dependency issues.

   In addition, you may run into a CircularDependencyError when running your
   migrations as Django won't be able to automatically break the dependency
   loop due to the dynamic dependency. If you see this error, you should
   break the loop by moving the models depended on by your User model
   into a second migration (you can try making two normal models that
   have a ForeignKey to each other and seeing how ``makemigrations`` resolves that
   circular dependency if you want to see how it's usually done)


Referencing the User model
--------------------------

.. currentmodule:: django.contrib.auth

If you reference :class:`~django.contrib.auth.models.User` directly (for
example, by referring to it in a foreign key), your code will not work in
projects where the ``AUTH_USER_MODEL`` setting has been changed to a
different User model.

.. function:: get_user_model()

    Instead of referring to :class:`~django.contrib.auth.models.User` directly,
    you should reference the user model using
    ``django.contrib.auth.get_user_model()``. This method will return the
    currently active User model -- the custom User model if one is specified, or
    :class:`~django.contrib.auth.models.User` otherwise.

    When you define a foreign key or many-to-many relations to the User model,
    you should specify the custom model using the ``AUTH_USER_MODEL``
    setting. For example::

        from django.conf import settings
        from django.db import models

        class Article(models.Model):
            author = models.ForeignKey(settings.AUTH_USER_MODEL)

    When connecting to signals sent by the ``User`` model, you should specify
    the custom model using the ``AUTH_USER_MODEL`` setting. For example::

        from django.conf import settings
        from django.db.models.signals import post_save

        def post_save_receiver(signal, sender, instance, \*\*kwargs):
            pass

        post_save.connect(post_save_receiver, sender=settings.AUTH_USER_MODEL)

    Generally speaking, you should reference the User model with the
    ``AUTH_USER_MODEL`` setting in code that is executed at import
    time. ``get_user_model()`` only works once Django has imported all models.

.. _specifying-custom-user-model:

Specifying a custom User model
------------------------------

.. admonition:: Model design considerations

    Think carefully before handling information not directly related to
    authentication in your custom User Model.

    It may be better to store app-specific user information in a model
    that has a relation with the User model. That allows each app to specify
    its own user data requirements without risking conflicts with other
    apps. On the other hand, queries to retrieve this related information
    will involve a database join, which may have an effect on performance.

Django expects your custom User model to meet some minimum requirements.

1. Your model must have an integer primary key.

2. Your model must have a single unique field that can be used for
   identification purposes. This can be a username, an email address,
   or any other unique attribute.

3. Your model must provide a way to address the user in a "short" and
   "long" form. The most common interpretation of this would be to use
   the user's given name as the "short" identifier, and the user's full
   name as the "long" identifier. However, there are no constraints on
   what these two methods return - if you want, they can return exactly
   the same value.

The easiest way to construct a compliant custom User model is to inherit from
:class:`~django.contrib.auth.models.AbstractBaseUser`.
:class:`~django.contrib.auth.models.AbstractBaseUser` provides the core
implementation of a ``User`` model, including hashed passwords and tokenized
password resets. You must then provide some key implementation details:

.. currentmodule:: django.contrib.auth

.. class:: models.CustomUser

    .. attribute:: USERNAME_FIELD

        A string describing the name of the field on the User model that is
        used as the unique identifier. This will usually be a username of
        some kind, but it can also be an email address, or any other unique
        identifier. The field *must* be unique (i.e., have ``unique=True``
        set in its definition).

        In the following example, the field ``identifier`` is used
        as the identifying field::

            class MyUser(AbstractBaseUser):
                identifier = models.CharField(max_length=40, unique=True)
                ...
                USERNAME_FIELD = 'identifier'

        :attr:`USERNAME_FIELD` supports
        :class:`~django.db.models.ForeignKey`\s. Since there is no way to pass
        model instances during the ``createsuperuser`` prompt, expect the
        user to enter the value of :attr:`~django.db.models.ForeignKey.to_field`
        value (the :attr:`~django.db.models.Field.primary_key` by default) of an
        existing instance.

    .. attribute:: REQUIRED_FIELDS

        A list of the field names that will be prompted for when creating a
        user via the ``createsuperuser`` management command. The user
        will be prompted to supply a value for each of these fields. It must
        include any field for which :attr:`~django.db.models.Field.blank` is
        ``False`` or undefined and may include additional fields you want
        prompted for when a user is created interactively.
        ``REQUIRED_FIELDS`` has no effect in other parts of Django, like
        creating a user in the admin.

        For example, here is the partial definition for a ``User`` model that
        defines two required fields - a date of birth and height::

            class MyUser(AbstractBaseUser):
                ...
                date_of_birth = models.DateField()
                height = models.FloatField()
                ...
                REQUIRED_FIELDS = ['date_of_birth', 'height']

        .. note::

            ``REQUIRED_FIELDS`` must contain all required fields on your
            ``User`` model, but should *not* contain the ``USERNAME_FIELD`` or
            ``password`` as these fields will always be prompted for.

        :attr:`REQUIRED_FIELDS` supports
        :class:`~django.db.models.ForeignKey`\s. Since there is no way to pass
        model instances during the ``createsuperuser`` prompt, expect the
        user to enter the value of :attr:`~django.db.models.ForeignKey.to_field`
        value (the :attr:`~django.db.models.Field.primary_key` by default) of an
        existing instance.

    .. attribute:: is_active

        A boolean attribute that indicates whether the user is considered
        "active".  This attribute is provided as an attribute on
        ``AbstractBaseUser`` defaulting to ``True``. How you choose to
        implement it will depend on the details of your chosen auth backends.
        See the documentation of the :attr:`is_active attribute on the built-in
        user model <django.contrib.auth.models.User.is_active>` for details.

    .. method:: get_full_name()

        A longer formal identifier for the user. A common interpretation
        would be the full name of the user, but it can be any string that
        identifies the user.

    .. method:: get_short_name()

        A short, informal identifier for the user. A common interpretation
        would be the first name of the user, but it can be any string that
        identifies the user in an informal way. It may also return the same
        value as :meth:`django.contrib.auth.models.User.get_full_name()`.

The following methods are available on any subclass of
:class:`~django.contrib.auth.models.AbstractBaseUser`:

.. class:: models.AbstractBaseUser

    .. method:: get_username()

        Returns the value of the field nominated by ``USERNAME_FIELD``.

    .. method:: models.AbstractBaseUser.is_anonymous()

        Always returns ``False``. This is a way of differentiating
        from  :class:`~django.contrib.auth.models.AnonymousUser` objects.
        Generally, you should prefer using
        :meth:`~django.contrib.auth.models.AbstractBaseUser.is_authenticated()` to this
        method.

    .. method:: models.AbstractBaseUser.is_authenticated()

        Always returns ``True``. This is a way to tell if the user has been
        authenticated. This does not imply any permissions, and doesn't check
        if the user is active - it only indicates that the user has provided a
        valid username and password.

    .. method:: models.AbstractBaseUser.set_password(raw_password)

        Sets the user's password to the given raw string, taking care of the
        password hashing. Doesn't save the
        :class:`~django.contrib.auth.models.AbstractBaseUser` object.

        When the raw_password is ``None``, the password will be set to an
        unusable password, as if
        :meth:`~django.contrib.auth.models.AbstractBaseUser.set_unusable_password()`
        were used.

    .. method:: models.AbstractBaseUser.check_password(raw_password)

        Returns ``True`` if the given raw string is the correct password for
        the user. (This takes care of the password hashing in making the
        comparison.)

    .. method:: models.AbstractBaseUser.set_unusable_password()

        Marks the user as having no password set.  This isn't the same as
        having a blank string for a password.
        :meth:`~django.contrib.auth.models.AbstractBaseUser.check_password()` for this user
        will never return ``True``. Doesn't save the
        :class:`~django.contrib.auth.models.AbstractBaseUser` object.

        You may need this if authentication for your application takes place
        against an existing external source such as an LDAP directory.

    .. method:: models.AbstractBaseUser.has_usable_password()

        Returns ``False`` if
        :meth:`~django.contrib.auth.models.AbstractBaseUser.set_unusable_password()` has
        been called for this user.

    .. method:: models.AbstractBaseUser.get_session_auth_hash()

        Returns an HMAC of the password field. Used for
        session invalidation on password change.

You should also define a custom manager for your ``User`` model. If your
``User`` model defines ``username``, ``email``, ``is_staff``, ``is_active``,
``is_superuser``, ``last_login``, and ``date_joined`` fields the same as
Django's default ``User``, you can just install Django's
:class:`~django.contrib.auth.models.UserManager`; however, if your ``User``
model defines different fields, you will need to define a custom manager that
extends :class:`~django.contrib.auth.models.BaseUserManager` providing two
additional methods:

.. class:: models.CustomUserManager

    .. method:: models.CustomUserManager.create_user(*username_field*, password=None, \**other_fields)

        The prototype of ``create_user()`` should accept the username field,
        plus all required fields as arguments. For example, if your user model
        uses ``email`` as the username field, and has ``date_of_birth`` as a
        required field, then ``create_user`` should be defined as::

            def create_user(self, email, date_of_birth, password=None):
                # create user here
                ...

    .. method:: models.CustomUserManager.create_superuser(*username_field*, password, \**other_fields)

        The prototype of ``create_superuser()`` should accept the username
        field, plus all required fields as arguments. For example, if your user
        model uses ``email`` as the username field, and has ``date_of_birth``
        as a required field, then ``create_superuser`` should be defined as::

            def create_superuser(self, email, date_of_birth, password):
                # create superuser here
                ...

        Unlike ``create_user()``, ``create_superuser()`` *must* require the
        caller to provide a password.

:class:`~django.contrib.auth.models.BaseUserManager` provides the following
utility methods:

.. class:: models.BaseUserManager

    .. method:: models.BaseUserManager.normalize_email(email)

        A ``classmethod`` that normalizes email addresses by lowercasing
        the domain portion of the email address.

    .. method:: models.BaseUserManager.get_by_natural_key(username)

        Retrieves a user instance using the contents of the field
        nominated by ``USERNAME_FIELD``.

    .. method:: models.BaseUserManager.make_random_password(length=10, allowed_chars='abcdefghjkmnpqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ23456789')

        Returns a random password with the given length and given string of
        allowed characters. Note that the default value of ``allowed_chars``
        doesn't contain letters that can cause user confusion, including:

        * ``i``, ``l``, ``I``, and ``1`` (lowercase letter i, lowercase
          letter L, uppercase letter i, and the number one)
        * ``o``, ``O``, and ``0`` (lowercase letter o, uppercase letter o,
          and zero)

Extending Django's default User
-------------------------------

If you're entirely happy with Django's :class:`~django.contrib.auth.models.User`
model and you just want to add some additional profile information, you could
simply subclass ``django.contrib.auth.models.AbstractUser`` and add your
custom profile fields, although we'd recommend a separate model.
``AbstractUser`` provides the full implementation of the default
:class:`~django.contrib.auth.models.User` as an abstract model.

.. _custom-users-and-the-built-in-auth-forms:

Custom users and the built-in auth forms
----------------------------------------

As you may expect, Django's built in forms and views make certain assumptions about the user
model that they are working with.

If your user model doesn't follow the same assumptions, it may be necessary to define
a replacement form, and pass that form in as part of the configuration of the
auth views.

* :class:`~django.contrib.auth.forms.UserCreationForm`

  Depends on the :class:`~django.contrib.auth.models.User` model.
  Must be re-written for any custom user model.

* :class:`~django.contrib.auth.forms.UserChangeForm`

  Depends on the :class:`~django.contrib.auth.models.User` model.
  Must be re-written for any custom user model.

* :class:`~django.contrib.auth.forms.AuthenticationForm`

  Works with any subclass of :class:`~django.contrib.auth.models.AbstractBaseUser`,
  and will adapt to use the field defined in ``USERNAME_FIELD``.

* :class:`~django.contrib.auth.forms.PasswordResetForm`

  Assumes that the user model has a field named ``email`` that can be used to
  identify the user and a boolean field named ``is_active`` to prevent
  password resets for inactive users.

* :class:`~django.contrib.auth.forms.SetPasswordForm`

  Works with any subclass of :class:`~django.contrib.auth.models.AbstractBaseUser`

* :class:`~django.contrib.auth.forms.PasswordChangeForm`

  Works with any subclass of :class:`~django.contrib.auth.models.AbstractBaseUser`

* :class:`~django.contrib.auth.forms.AdminPasswordChangeForm`

  Works with any subclass of :class:`~django.contrib.auth.models.AbstractBaseUser`


Custom users and :mod:`django.contrib.admin`
--------------------------------------------

If you want your custom User model to also work with Admin, your User model must
define some additional attributes and methods. These methods allow the admin to
control access of the User to admin content:

.. class:: models.CustomUser

.. attribute:: is_staff

    Returns ``True`` if the user is allowed to have access to the admin site.

.. attribute:: is_active

    Returns ``True`` if the user account is currently active.

.. method:: has_perm(perm, obj=None):

    Returns ``True`` if the user has the named permission. If ``obj`` is
    provided, the permission needs to be checked against a specific object
    instance.

.. method:: has_module_perms(app_label):

    Returns ``True`` if the user has permission to access models in
    the given app.

You will also need to register your custom User model with the admin. If
your custom User model extends ``django.contrib.auth.models.AbstractUser``,
you can use Django's existing ``django.contrib.auth.admin.UserAdmin``
class. However, if your User model extends
:class:`~django.contrib.auth.models.AbstractBaseUser`, you'll need to define
a custom ``ModelAdmin`` class. It may be possible to subclass the default
``django.contrib.auth.admin.UserAdmin``; however, you'll need to
override any of the definitions that refer to fields on
``django.contrib.auth.models.AbstractUser`` that aren't on your
custom User class.

Custom users and permissions
----------------------------

To make it easy to include Django's permission framework into your own User
class, Django provides :class:`~django.contrib.auth.models.PermissionsMixin`.
This is an abstract model you can include in the class hierarchy for your User
model, giving you all the methods and database fields necessary to support
Django's permission model.

:class:`~django.contrib.auth.models.PermissionsMixin` provides the following
methods and attributes:

.. class:: models.PermissionsMixin

    .. attribute:: models.PermissionsMixin.is_superuser

        Boolean. Designates that this user has all permissions without
        explicitly assigning them.

    .. method:: models.PermissionsMixin.get_group_permissions(obj=None)

        Returns a set of permission strings that the user has, through their
        groups.

        If ``obj`` is passed in, only returns the group permissions for
        this specific object.

    .. method:: models.PermissionsMixin.get_all_permissions(obj=None)

        Returns a set of permission strings that the user has, both through
        group and user permissions.

        If ``obj`` is passed in, only returns the permissions for this
        specific object.

    .. method:: models.PermissionsMixin.has_perm(perm, obj=None)

        Returns ``True`` if the user has the specified permission, where
        ``perm`` is in the format ``"<app label>.<permission codename>"``. If
        the user is inactive, this method will always return ``False``.

        If ``obj`` is passed in, this method won't check for a permission for
        the model, but for this specific object.

    .. method:: models.PermissionsMixin.has_perms(perm_list, obj=None)

        Returns ``True`` if the user has each of the specified permissions,
        where each perm is in the format
        ``"<app label>.<permission codename>"``. If the user is inactive,
        this method will always return ``False``.

        If ``obj`` is passed in, this method won't check for permissions for
        the model, but for the specific object.

    .. method:: models.PermissionsMixin.has_module_perms(package_name)

        Returns ``True`` if the user has any permissions in the given package
        (the Django app label). If the user is inactive, this method will
        always return ``False``.

.. admonition:: ModelBackend

    If you don't include the
    :class:`~django.contrib.auth.models.PermissionsMixin`, you must ensure you
    don't invoke the permissions methods on ``ModelBackend``. ``ModelBackend``
    assumes that certain fields are available on your user model. If your User
    model doesn't provide  those fields, you will receive database errors when
    you check permissions.

Custom users and Proxy models
-----------------------------

One limitation of custom User models is that installing a custom User model
will break any proxy model extending :class:`~django.contrib.auth.models.User`.
Proxy models must be based on a concrete base class; by defining a custom User
model, you remove the ability of Django to reliably identify the base class.

If your project uses proxy models, you must either modify the proxy to extend
the User model that is currently in use in your project, or merge your proxy's
behavior into your User subclass.

Custom users and signals
------------------------

Another limitation of custom User models is that you can't use
:func:`django.contrib.auth.get_user_model()` as the sender or target of a signal
handler. Instead, you must register the handler with the resulting User model.

Custom users and testing/fixtures
---------------------------------

If you are writing an application that interacts with the User model, you must
take some precautions to ensure that your test suite will run regardless of
the User model that is being used by a project. Any test that instantiates an
instance of User will fail if the User model has been swapped out. This
includes any attempt to create an instance of User with a fixture.

To ensure that your test suite will pass in any project configuration,
``django.contrib.auth.tests.utils`` defines a ``@skipIfCustomUser`` decorator.
This decorator will cause a test case to be skipped if any User model other
than the default Django user is in use. This decorator can be applied to a
single test, or to an entire test class.

Depending on your application, tests may also be needed to be added to ensure
that the application works with *any* user model, not just the default User
model. To assist with this, Django provides two substitute user models that
can be used in test suites:

.. class:: tests.custom_user.CustomUser

  A custom user model that uses an ``email`` field as the username, and has a basic
  admin-compliant permissions setup

.. class:: tests.custom_user.ExtensionUser

  A custom user model that extends ``django.contrib.auth.models.AbstractUser``,
  adding a ``date_of_birth`` field.

You can then use the ``@override_settings`` decorator to make that test run
with the custom User model. For example, here is a skeleton for a test that
would test three possible User models -- the default, plus the two User
models provided by ``auth`` app::

    from django.contrib.auth.tests.utils import skipIfCustomUser
    from django.contrib.auth.tests.custom_user import CustomUser, ExtensionUser
    from django.test import TestCase, override_settings


    class ApplicationTestCase(TestCase):
        @skipIfCustomUser
        def test_normal_user(self):
            "Run tests for the normal user model"
            self.assertSomething()

        @override_settings(AUTH_USER_MODEL='auth.CustomUser')
        def test_custom_user(self):
            "Run tests for a custom user model with email-based authentication"
            self.assertSomething()

        @override_settings(AUTH_USER_MODEL='auth.ExtensionUser')
        def test_extension_user(self):
            "Run tests for a simple extension of the built-in User."
            self.assertSomething()

A full example
--------------

Here is an example of an admin-compliant custom user app. This user model uses
an email address as the username, and has a required date of birth; it
provides no permission checking, beyond a simple ``admin`` flag on the user
account. This model would be compatible with all the built-in auth forms and
views, except for the User creation forms. This example illustrates how most of
the components work together, but is not intended to be copied directly into
projects for production use.

This code would all live in a ``models.py`` file for a custom
authentication app::

    from django.db import models
    from django.contrib.auth.models import (
        BaseUserManager, AbstractBaseUser
    )


    class MyUserManager(BaseUserManager):
        def create_user(self, email, date_of_birth, password=None):
            """
            Creates and saves a User with the given email, date of
            birth and password.
            """
            if not email:
                raise ValueError('Users must have an email address')

            user = self.model(
                email=self.normalize_email(email),
                date_of_birth=date_of_birth,
            )

            user.set_password(password)
            user.save(using=self._db)
            return user

        def create_superuser(self, email, date_of_birth, password):
            """
            Creates and saves a superuser with the given email, date of
            birth and password.
            """
            user = self.create_user(email,
                password=password,
                date_of_birth=date_of_birth
            )
            user.is_admin = True
            user.save(using=self._db)
            return user


    class MyUser(AbstractBaseUser):
        email = models.EmailField(
            verbose_name='email address',
            max_length=255,
            unique=True,
        )
        date_of_birth = models.DateField()
        is_active = models.BooleanField(default=True)
        is_admin = models.BooleanField(default=False)

        objects = MyUserManager()

        USERNAME_FIELD = 'email'
        REQUIRED_FIELDS = ['date_of_birth']

        def get_full_name(self):
            # The user is identified by their email address
            return self.email

        def get_short_name(self):
            # The user is identified by their email address
            return self.email

        def __str__(self):     
            return self.email

        def has_perm(self, perm, obj=None):
            "Does the user have a specific permission?"
            # Simplest possible answer: Yes, always
            return True

        def has_module_perms(self, app_label):
            "Does the user have permissions to view the app `app_label`?"
            # Simplest possible answer: Yes, always
            return True

        @property
        def is_staff(self):
            "Is the user a member of staff?"
            # Simplest possible answer: All admins are staff
            return self.is_admin

Then, to register this custom User model with Django's admin, the following
code would be required in the app's ``admin.py`` file::

    from django import forms
    from django.contrib import admin
    from django.contrib.auth.models import Group
    from django.contrib.auth.admin import UserAdmin
    from django.contrib.auth.forms import ReadOnlyPasswordHashField

    from customauth.models import MyUser


    class UserCreationForm(forms.ModelForm):
        """A form for creating new users. Includes all the required
        fields, plus a repeated password."""
        password1 = forms.CharField(label='Password', widget=forms.PasswordInput)
        password2 = forms.CharField(label='Password confirmation', widget=forms.PasswordInput)

        class Meta:
            model = MyUser
            fields = ('email', 'date_of_birth')

        def clean_password2(self):
            # Check that the two password entries match
            password1 = self.cleaned_data.get("password1")
            password2 = self.cleaned_data.get("password2")
            if password1 and password2 and password1 != password2:
                raise forms.ValidationError("Passwords don't match")
            return password2

        def save(self, commit=True):
            # Save the provided password in hashed format
            user = super(UserCreationForm, self).save(commit=False)
            user.set_password(self.cleaned_data["password1"])
            if commit:
                user.save()
            return user


    class UserChangeForm(forms.ModelForm):
        """A form for updating users. Includes all the fields on
        the user, but replaces the password field with admin's
        password hash display field.
        """
        password = ReadOnlyPasswordHashField()

        class Meta:
            model = MyUser
            fields = ('email', 'password', 'date_of_birth', 'is_active', 'is_admin')

        def clean_password(self):
            # Regardless of what the user provides, return the initial value.
            # This is done here, rather than on the field, because the
            # field does not have access to the initial value
            return self.initial["password"]


    class MyUserAdmin(UserAdmin):
        # The forms to add and change user instances
        form = UserChangeForm
        add_form = UserCreationForm

        # The fields to be used in displaying the User model.
        # These override the definitions on the base UserAdmin
        # that reference specific fields on auth.User.
        list_display = ('email', 'date_of_birth', 'is_admin')
        list_filter = ('is_admin',)
        fieldsets = (
            (None, {'fields': ('email', 'password')}),
            ('Personal info', {'fields': ('date_of_birth',)}),
            ('Permissions', {'fields': ('is_admin',)}),
        )
        # add_fieldsets is not a standard ModelAdmin attribute. UserAdmin
        # overrides get_fieldsets to use this attribute when creating a user.
        add_fieldsets = (
            (None, {
                'classes': ('wide',),
                'fields': ('email', 'date_of_birth', 'password1', 'password2')}
            ),
        )
        search_fields = ('email',)
        ordering = ('email',)
        filter_horizontal = ()

    # Now register the new UserAdmin...
    admin.site.register(MyUser, MyUserAdmin)
    # ... and, since we're not using Django's built-in permissions,
    # unregister the Group model from admin.
    admin.site.unregister(Group)

Finally, specify the custom model as the default user model for your project
using the ``AUTH_USER_MODEL`` setting in your ``settings.py``::

    AUTH_USER_MODEL = 'customauth.MyUser'

