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

There are a few ways in which it is can be a bit of a leaky abstraction though.

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
        username="user1",
        email="user1@example.com",
        password="changeme1",
    )
    User.objects.create_user(
        username="user2",
        email="user2@example.com",
        password="changeme2",
    )
```

this will create 2 users, writing the records to our `auth` DB. But the transaction wrapping those writes uses the `default` DB connection. This won't throw any exception or error. It just means these two writes are not wrapped in a meaningful transaction. However, to a casual reader of the code, it probably looks like they are.

The correct code in this case would be `with transaction.atomic(using="auth"):` but it is worth being aware of this. Using multiple DBs opens the door to a class of wrong code that looks right.

## Data Migrations

It is also important to be careful when writing custom migrations using `RunPython`. Continuing to use our example above where we have a `default` DB and an `auth` DB, consider the following migration code:

```py
def create_user(apps, schema_editor):
    User = apps.get_model("auth", "User")
    User.objects.create_user(
        username="user1",
        email="user1@example.com",
        password="changeme1",
    )

class Migration(migrations.Migration):
    operations = [
        migrations.RunPython(
            create_user,
            reverse_code=migrations.RunPython.noop
        )
    ]
```

This code looks fairly reasonable on the surface. If you are moving to using multiple DBs in an application which has historically run on a single DB, you probably have some migrations like this kicking about in the project history. The catch here is: If we run `./manage.py migrate` it will attempt to create the User object on the `default` DB connection. That will also happen if we run `./manage.py migrate --database auth`. This happens because the model retrieved via `apps.get_model()` is unaware of the database being used in the migration process. Django uses the default database unless explicitly specified.

The correct code in this case would be:

```py
def create_user(apps, schema_editor):
    User = apps.get_model("auth", "User")
    User.objects.using(
        schema_editor.connection.alias
    ).create_user(
        username="user1",
        email="user1@example.com",
        password="changeme1",
    )
```
