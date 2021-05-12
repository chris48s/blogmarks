---
layout: post
title: Failing the CI build if django migrations are out of date
categories: ['python']
tags: ['python']
---

## Failing the CI build if django migrations are out of date

A common mistake in django is to make a model change but forget to run `makemigrations` to generate a migration for the model change. Sometimes it is not entirely obvious when this need to happen. For example, let's say I'm using the [django-extensions](https://github.com/django-extensions/django-extensions) library and I define a model like:

```py
# models.py

from django.db import models
from django_extensions.db.models import TimeStampedModel

class MyModel(TimeStampedModel, models.Model):
    pass
```

In this situation, upgrading django-extensions to a new version might require me to regenerate the migrations in my app, even though I haven't made any changes to `models.py` and overlooking this could generate unexpected results.

Fortunately there is a simple thing I can do to detect and warn if this happens: If I run

```
python manage.py makemigrations --check
```

in my CI build, this will cause the build to fail if my migrations are out of sync with the models, warning me about the problem.
