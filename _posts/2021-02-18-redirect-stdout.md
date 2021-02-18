---
layout: post
title: Capturing stdout in python
categories: ['python']
tags: ['python', 'testing']
---

## Capturing stdout in python

Sometimes it is helpful to capture stdout/stderr. I usually use this when writing tests to make assertions about terminal output or just suppress it when the tests are running. Normally I would do a little switcheroo like this to manually manipulate `sys.stdout`:

```py
from io import StringIO 
import sys

sys.stdout = StringIO()
print('foobar')
val = sys.stdout.getvalue()
sys.stdout = sys.__stdout__
```

but today I leaned about python's [`contextlib.redirect_stdout`](https://docs.python.org/3/library/contextlib.html#contextlib.redirect_stdout) and [`contextlib.redirect_stderr`](https://docs.python.org/3/library/contextlib.html#contextlib.redirect_stderr). These standard library context managers provide a built-in abstraction over this operation:

```py
from io import StringIO 
from contextlib import redirect_stdout

with StringIO() as buf, redirect_stdout(buf):
    print('foobar')
    val = buf.getvalue()
```
