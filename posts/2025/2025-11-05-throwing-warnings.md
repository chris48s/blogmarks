<!--
.. title: Throwing warnings as if they are exceptions
.. slug: throwing-warnings
.. date: 2025-11-05 00:00:00
.. tags: python,testing
.. category: 
.. link: 
.. description: 
.. type: text
-->

Sometimes a warning is logged from third-party code in a way that doesn't make it clear where the problem is in your own code. For example, today I was trying to track down the cause of this:

```text
.venv/lib/python3.12/site-packages/django/template/base.py:1047: RemovedInDjango50Warning: The "default.html" templates for forms and formsets will be removed. These were proxies to the equivalent "table.html" templates, but the new "div.html" templates will be the default from Django 5.0. Transitional renderers are provided to allow you to opt-in to the new output style now. See https://docs.djangoproject.com/en/4.2/releases/4.1/ for more details
```

I understand the message, but `.venv/lib/python3.12/site-packages/django/template/base.py` doesn't tell me where in my own codebase to look for the problem.

Today I learned that you can tell pytest (and indeed, python itself) to throw warnings as if they are an exception. So you can run

```bash
pytest -W error
```

to treat all warnings as if they are an exception, or something like

```bash
pytest -W error::django.utils.deprecation.RemovedInDjango50Warning
```

to only apply this behaviour to a particular class of warning.

Either way, this will give you a stacktrace from the point where the warning is logged which can give you some breadcrumbs to follow.
