<!--
.. title: Django DB Routers: Avoiding Race Conditions
.. slug: django-db-routers-2
.. date: 2025-04-04 00:00:00
.. tags: python,django
.. category: 
.. link: 
.. description: 
.. type: text
-->

If you are using django with a Postgres primary/replica setup for the first time, your first instinct may be to write something like this:

```py
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        ...
    },
    "replica1": {
        "ENGINE": "django.db.backends.postgresql",
        ...
    },
    "replica2": {
        "ENGINE": "django.db.backends.postgresql",
        ...
    },
}


import random

class AuthRouter:
    def db_for_read(self, model, **hints):
        return random.choice(
            ["replica1", "replica2"]
        )

    def db_for_write(self, model, **hints):
        return "default"
    ...
```

Read from the replicas, write to the primary - simple. There are tutorials online that will tell you to do this.

However, there's a problem with this setup: You now have a race condition. Replication is not instant. If you write something to the primary and then read back the same record from one of the replicas, the data may be stale.

How you choose to solve this depends on your application, but here's one strategy I have recently used.

A site I am working on has essentially a read-only public-facing site with a large number of users. Writes are made either by a handful of internal users via django's admin backend, or by management commands. We want to prevent race conditions in contexts where we will perform writes and scale DB reads in the public-facing parts of the site.

In order to do this, we used a package called [django-middleware-global-request](https://pypi.org/project/django-middleware-global-request/). This package provides a middleware that can store the current request in a thread-local variable so it can be accessed in contexts where a request would not normally be in scope.

Using this package allows us to write the following code:

```py
import random
from django_middleware_global_request import get_request

class AuthRouter:
    def db_for_read(self, model, **hints):
        request = get_request()

        if request:
            if request.path.startswith("/admin"):
                # always read from the primary
                # in the admin interface
                return "default"

            # use replicas to serve read traffic
            # on the public site
            return random.choice(
                ["replica1", "replica2"]
            )

        # if there's no request in scope,
        # read from the primary
        # we don't care about trying to "scale"
        # management commands
        return "default"

    def db_for_write(self, model, **hints):
        return "default"
    ...
```

There are some situations where this wouldn't work. Obviously if you can't generalise about traffic based on the URL this strategy isn't going to hold water. Perhaps less obviously, if you've got an application which uses a lot of background tasks (which also will not have a request in scope) you might want to also serve some reads from the replicas in that situation. This approach would serve all the reads from the primary in that context.

For my use-case, this worked like a charm.
