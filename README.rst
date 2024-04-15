.. image:: https://img.shields.io/pypi/v/django-dynamic-raw-id.svg
    :target: https://pypi.org/project/django-dynamic-raw-id/

.. image:: https://github.com/lincolnloop/django-dynamic-raw-id/actions/workflows/test.yml/badge.svg
    :target: https://travis-ci.org/lincolnloop/django-dynamic-raw-id

----

=====================
django-dynamic-raw-id
=====================

A Django admin raw_id_fields widget replacement that handles display of an
object's string value on change and can be overridden via a template.
See this example:

.. image:: http://d.pr/i/10GtM.png
    :target: http://d.pr/i/1kv7d.png

Compatibility Matrix:
=====================

========= === === === ====
Py/Dj     3.7 3.8 3.9 3.10
========= === === === ====
3.2         ✓   ✓   ✓   ✓
4.0         ✓   ✓   ✓   ✓
========= === === === ====

Rationale
=========

By default, Django’s admin loads all possible related instances into a select-box interface (<select>) for fields that are ForeignKey. This can result in long load times and unresponsive admin pages for models with thousands of instances, or with multiple ForeinKeys.

The normal fix is to use Django's ModelAdmin.raw_id_fields_, but by default it *only* shows the raw id of the related model instance, which is somewhat unhelpful.

This package improve the user experience by providing the string representation or other customized text for the related instance, linked to that instance's admin change form, in addition to the raw id itself.

Installation
============


Install the package with ``pip``:

.. code-block:: bash

    $ pip install django-dynamic-raw-id

Put ``dynamic_raw_id`` to your list of ``INSTALLED_APPS``:

.. code-block:: python

    INSTALLED_APPS = (
        # ... other apps
        'dynamic_raw_id',
    )

And add the ``urlpattern``. Make sure its listed *before* the generic
``admin.site.urls`` urlpattern include:

.. code-block:: python

    urlpatterns = [
        # ...
        path('admin/dynamic_raw_id/', include('dynamic_raw_id.urls')),
        path("admin/", admin.site.urls),
    ]

``dynamic_raw_id`` comes with some static files so don't forget to run
``manage.py collectstatic``.

Usage
=====

To start using django-dynamic-raw-id in your application all you need to do is
implement ``DynamicRawIDMixin`` in your  ``ModelAdmin`` class and add the desired
fields to a list of ``dynamic_raw_id_fields``:

.. code-block:: python

    from dynamic_raw_id.admin import DynamicRawIDMixin

    class UserProfileAdmin(DynamicRawIDMixin, admin.ModelAdmin):
        dynamic_raw_id_fields = ('user',)

You can use dynamic_raw_id widgets in a Admin filter as well:

.. code-block:: python

    from dynamic_raw_id.admin import DynamicRawIDMixin
    from dynamic_raw_id.filters import DynamicRawIDFilter

    class UserProfileAdmin(DynamicRawIDMixin, admin.ModelAdmin):
       list_filter = (
           ('dynamic_raw_id_fk', DynamicRawIDFilter),
       )


Customizing the value of the dynamic widget
===========================================

The coolest feature of django-dynamic-raw-id is the ability to customize the output
of the value displayed alongside the ``DynamicRawIDWidget``.  There is a basic
implementation if all you want is your object's ``__unicode__`` value. To change
the value displayed all you need to do is implement the correct template.

django-dynamic-raw-id looks for this template structure ``dynamic_raw_id/<app>/<model>.html``
and ``dynamic_raw_id/<app>/multi_<model>.html`` (for multi-value lookups).

For instance, if I have a blog post with a ``User`` dynamic_raw_id field that I want
display as ``Firstname Lastname``, I would create the template
``dynamic_raw_id/auth/user.html`` with:

.. code-block:: html+django

    <span>{{ object.0.first_name }} {{ object.0.last_name }}</span>

A custom admin URL prefix
=========================

If you have your admin *and* the dynamic_raw_id scripts located on a different
prefix than ``/admin/dynamic_raw_id/`` you need adjust the ``DYNAMIC_RAW_ID_MOUNT_URL``
JS variable.

Example:

.. code-block::

    # In case the app is setup at /foobar/dynamic_raw_id/
    url(r'^foobar/dynamic_raw_id/', include('dynamic_raw_id.urls')),

    # Provide a
    <script>
        window.DYNAMIC_RAW_ID_MOUNT_URL = "{% url "admin:index" %}";
    </script>

An ideal place is the admin ``base_site.html`` template. Full example:

.. code-block:: html+django

    {% extends "admin/base.html" %}

    {% block title %}{{ title }} | {{ site_title|default:_('Django site admin') }}{% endblock %}

    {% block extrahead %}
      {{ block.super }}
      <script>
        window.DYNAMIC_RAW_ID_MOUNT_URL = "{% url "admin:index" %}";
      </script>
    {% endblock %}

    {% block branding %}
    <h1 id="site-name"><a href="{% url 'admin:index' %}">{{ site_header|default:_('Django administration') }}</a></h1>
    {% endblock %}

    {% block nav-global %}{% endblock %}


Testing and Local Development
=============================

The testsuite uses Selenium to do frontend tests, we require Firefox and
geckodriver_ to be installed. You can install geckodriver on OS X with
Homebrew:

.. code-block:: bash

    $ brew install geckodriver

Run the testsuite in your local environment using:

.. code-block:: bash

    $ cd django-dynamic-raw-id/
    $ pipenv install --dev
    $ pipenv run pytest

Or use tox to test against various Django and Python versions:

.. code-block:: bash

    $ tox -r

You can also invoke the test suite or other 'manage.py' commands by calling
the ``django-admin`` tool with the test app settings:

.. code-block:: bash

    $ cd django-dynamic-raw-id/
    $ pipenv install --dev
    $ pipenv run pytest

This also allows you to run the internal testing app in a testserver, to
preview a sample of what django-dynamic-raw-id is doing:

.. code-block:: bash

    $ pipenv run django-admin migrate
    $ pipenv run django-admin createsuperuser
    $ pipenv run django-admin runserver

.. note:: The default settings file is set in the ``.env`` file which
   pipenv automatically exposes:

.. code-block:: bash

    DJANGO_SETTINGS_MODULE=dynamic_raw_id.tests.testapp.settings


.. _geckodriver: https://github.com/mozilla/geckodriver
.. _ModelAdmin.raw_id_fields: https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.raw_id_fields
