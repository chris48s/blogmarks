<!--
.. title: Pathlib Cheat Sheet
.. slug: pathlib-cheat-sheet
.. date: 2020-09-02 00:00:00
.. tags: python,python
.. category: python
.. link: 
.. description: 
.. type: text
-->

```py
>>> from pathlib import Path
>>> f = Path.home() / 'foo.tar.gz'  # same as f = Path('/home/chris/foo.tar.gz')

>>> f.is_file()
True

>>> f.is_dir()
False

>>> f.exists()
True

>>> f.absolute()
PosixPath('/home/chris/foo.tar.gz')

>>> f.as_uri()
'file:///home/chris/foo.tar.gz'

>>> f.as_posix()
'/home/chris/foo.tar.gz'

>>> f.parts
('/', 'home', 'chris', 'foo.tar.gz')

>>> f.suffix
'.gz'

>>> f.suffixes
['.tar', '.gz']

>>> f.parent
PosixPath('/home/chris')
```
