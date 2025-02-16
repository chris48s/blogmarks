<!--
.. title: Django DB Routers: Pitfalls
.. slug: django-db-routers-1
.. date: 2025-02-16 00:00:00
.. tags: python,django
.. category: 
.. link: 
.. description: 
.. type: text
-->

Django has a feature called [Database Routers](https://docs.djangoproject.com/en/5.2/topics/db/multi-db/). This gives you an application-level abstraction to manage multiple database connections. This can be useful if you are working with a primary/replica setup or if you're implementing database partitioning or sharding in your application.

There are a couple of ways in which it is can be a bit of a leaky abstraction though.

## Raw SQL

This one is pretty obvious if you think about it, but anything you define in your database routers only applies to database I/O that happens via the ORM. As soon as you use a `cursor` to execute raw SQL your DB routers are bypassed and it is your responsibility to manage which connection is being used. You can do this by importing `from django.db import connections` and selecting a named connection.

This isn't a big surprise, but it is worth being aware of if you are converting an existing application to use multiple DB connections. This is not normally something you implement from day one.

The other thing worth noting here is that even if your own application code only interacts with the DB through the ORM, if there is any package anywhere in your dependency tree that uses `connection.cursor()`, that is going to bypass any logic in your database router. This could lead to reads or writes performed by a third party package which don't happen on the expected DB connection.

## Transactions

This one is perhaps less obvious. Any time you use a transaction, you are also responsible for explicitly managing your own DB connections. Imagine we have an application with the following setup:

```py
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db1.sqlite3",
    },
    "auth": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db2.sqlite3",
    },
}

class AuthRouter:
    route_app_labels = {"auth", "contenttypes"}

    def db_for_read(self, model, **hints):
        if model._meta.app_label in self.route_app_labels:
            return "auth"
        return None

    def db_for_write(self, model, **hints):
        if model._meta.app_label in self.route_app_labels:
            return "auth"
        return None
```

If I write the following code:

```py
with transaction.atomic():
    User.objects.create_user(
        username='user1',
        email='user1@example.com',
        password='changeme1'
    )
    User.objects.create_user(
        username='user2',
        email='user2@example.com',
        password='changeme2'
    )
```

this will create 2 users, writing the records to our `auth` DB. But the transaction wrapping those writes uses the `default` DB connection. This won't throw any exception or error. It just means these two writes are not wrapped in a meaningful transaction. However, to a casual reader of the code, it probably looks like they are.

The correct code in this case would be `with transaction.atomic(using="auth"):` but it is worth being aware of this. Using multiple DBs opens the door to a class of wrong code that looks right.
