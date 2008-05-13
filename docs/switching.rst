Switching from other Template Engines
=====================================

.. highlight:: html+jinja

If you have used a different template engine in the past and want to swtich
to Jinja2 here is a small guide that shows the basic syntatic and semantic
changes between some common, similar text template engines for Python.

Jinja1
------

Jinja2 is mostly compatible with Jinja1 in terms of API usage and template
syntax.  The differences between Jinja1 and 2 are explained in the following
list.

API
~~~

Loaders
    Jinja2 uses a different loader API.  Because the internal representation
    of templates changed there is no longer support for external caching
    systems such as memcached.  The memory consumed by templates is comparable
    with regular Python modules now and external caching doesn't give any
    advantage.  If you have used a custom loader in the past have a look at
    the new :ref:`loader API <loaders>`.

Loading templates from strings
    In the past it was possible to generate templates from a string with the
    default environment configuration by using `jinja.from_string`.  Jinja2
    provides a :class:`Template` class that can be used to do the same, but
    with optional additional configuration.

Automatic unicode conversion
    Jinja1 performed automatic conversion of bytestrings in a given encoding
    into unicode objects.  This conversion is no longer implemented as it
    was inconsistent as most libraries are using the regular Python ASCII
    bytestring to Unicode conversion.  An application powered by Jinja2
    *has to* use unicode internally everywhere or make sure that Jinja2 only
    gets unicode strings passed.

i18n
    Jinja1 used custom translators for internationalization.  i18n is now
    available as Jinja2 extension and uses a simpler, more gettext friendly
    interface and has support for babel.  For more details see
    :ref:`i18n-extension`.

Internal methods
    Jinja1 exposed a few internal methods on the environment object such
    as `call_function`, `get_attribute` and others.  While they were marked
    as being an internal method it was possible to override them.  Jinja2
    doesn't have equivalent methods.

Sandbox
    Jinja1 was running sandbox mode by default.  Few applications actually
    used that feature so it became optional in Jinja2.  For more details
    about the sandboxed execution see :class:`SandboxedEnvironment`.

Context
    Jinja1 had a stacked context as storage for variables passed to the
    environment.  In Jinja2 a similar object exists but it doesn't allow
    modifications nor is it a singleton.  As inheritance is dynamic now
    multiple context objects may exist during template evaluation.

Filters and Tests
    Filters and tests are regular functions now.  It's no longer necessary
    and allowed to use factory functions.


Templates
~~~~~~~~~

Jinja2 has mostly the same syntax as Jinja1.  The only difference is that
assigning variables doesn't use `set` as keyword now.  The following
example shows a Jinja1 variable assignment::

    {% set foo = 42 %}

In Jinja2 the `set` is ommited::

    {% foo = 42 %}

Additionally macros require parentheses around the argument list now.

Jinja2 allows dynamic inheritance now and dynamic includes.  The old helper
function `rendertemplate` is gone now, `include` can be used instead.
Additionally includes no longer import macros and variable assignments, for
that the new `import` tag is used.  This concept is explained in the
:ref:`import` documentation.

Currently there is no support for the `recursive` modifier of for loops!


Django
------

If you have previously worked with Django templates, you should find
Jinja2 very familiar.  In fact, most of the syntax elements look and
work the same.

However, Jinja2 provides some more syntax elements covered in the
documentation and some work a bit different.

This section covers the template changes.  As the API is fundamentally
different we won't cover it here.

Method Calls
~~~~~~~~~~~~

In Django method calls work implicitly.  With Jinja2 you have to specify that
you want to call an object.  Thus this Django code::

    {% for page in user.get_created_pages %}
        ...
    {% endfor %}
    
will look like this in Jinja::

    {% for page in user.get_created_pages() %}
        ...
    {% endfor %}

This allows you to pass variables to the function which is also used for macros
which is not possible in Django.

Conditions
~~~~~~~~~~

In Django you can use the following constructs to check for equality::

    {% ifequal foo "bar" %}
        ...
    {% else %}
        ...
    {% endifequal %}

In Jinja2 you can use the normal if statement in combination with operators::

    {% if foo == 'bar' %}
        ...
    {% else %}
        ...
    {% endif %}

You can also have multiple elif branches in your template::

    {% if something %}
        ...
    {% elif otherthing %}
        ...
    {% elif foothing %}
        ...
    {% else %}
        ...
    {% endif %}

Filter Arguments
~~~~~~~~~~~~~~~~

Jinja2 provides more than one argument for filters.  Also the syntax for
argument passing is different.  A template that looks like this in Django::

    {{ items|join:", " }}

looks like this in Jinja2::

    {{ items|join(', ') }}

In fact it's a bit more verbose but it allows different types of arguments -
including variables - and more than one of them.

Tests
~~~~~

In addition to filters there also are tests you can perform using the is
operator.  Here are some examples::

    {% if user.user_id is odd %}
        {{ user.username|e }} is odd
    {% else %}
        hmm. {{ user.username|e }} looks pretty normal
    {% endif %}


Mako
----

.. highlight:: html+mako

If you have used Mako so far and want to switch to Jinja2 you can configure
Jinja2 to look more like Mako:

.. sourcecode:: python

    env = Environment('<%', '%>', '${', '}', '%')

Once the environment is configure like that Jinja2 should be able to interpret
a small subset of Mako templates.  Jinja2 does not support embedded Python code
so you would have to move that out of the template.  The syntax for defs (in
Jinja2 defs are called macros) and template inheritance is different too.  The
following Mako template::

    <%inherit file="layout.html" />
    <%def name="title()">Page Title</%def>
    <ul>
    % for item in list:
        <li>${item}</li>
    % endfor
    </ul>

Looks like this in Jinja2 with the above configuration::

    <% extends "layout.html" %>
    <% block title %>Page Title<% endblock %>
    <% block body %>
    <ul>
    % for item in list:
        <li>${item}</li>
    % endfor
    </ul>
    <% endblock %>