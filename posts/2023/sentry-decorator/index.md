<!--
.. title: Adding a custom tag to a Sentry event
.. slug: sentry-decorator
.. date: 2023-05-29 00:00:00
.. tags: python
.. category: 
.. link: 
.. description: 
.. type: text
-->

Sentry allows you to enrich captured events by applying
[custom tags and attributes](https://docs.sentry.io/product/sentry-basics/integrate-backend/capturing-errors/#enriching-your-event-data).
I was recently working on a python application where I needed a re-usable
abstraction to express the logic "if function X throws exception Y
then apply this custom `key=value` tag when we log the exception to Sentry"
in a bunch of places. Here's what I came up with:

```py
from functools import wraps
from sentry_sdk import capture_exception, push_scope

def tag_error(exc_class, key, value):
    def decorator(fn):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            try:
                return fn(*args, **kwargs)
            except (KeyboardInterrupt, SystemExit):
                raise
            except Exception as err:
                if isinstance(err, exc_class):
                    with push_scope() as scope:
                        scope.set_tag(key, value)
                        capture_exception(err)
                raise
        return wrapper
    return decorator
```

This gives us a `@tag_error` decorator, which can be applied to any function. For example:


```py
@tag_error(ValueError, "custom-key", "custom-value")
def do_a_thing():
    ...
    raise ValueError("Oh no. A terrible thing has happened.")
    ...
```

This will tag any `ValueError`s raised by calling `do_a_thing()` with
`custom-key=custom-value` when we log the exception to sentry.
