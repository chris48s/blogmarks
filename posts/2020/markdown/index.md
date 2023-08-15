<!--
.. title: Documenting a python project with markdown
.. slug: markdown
.. date: 2020-10-20 00:00:00
.. tags: python,markdown
.. category: 
.. link: 
.. description: 
.. type: text
-->

Last time I looked at writing documentation for a python library from scratch (a few years ago), there wasn't really a solution that allowed me to write my docs in markdown, render it to a static site and incorporate docstrings from my code into the generated documentation. [MkDocs](https://www.mkdocs.org/) allows you to write your free text in markdown, but at the time there wasn't really a good solution for pulling docstrings into the rendered docs. Conversely, [Sphinx](https://www.sphinx-doc.org/) did have a good solution for pulling in docstrings but forced you to use reStructuredText for the free text.

Things have changed a bit since then though, on both sides of that equation. I wanted to write a docs site for my [geometry-to-spatialite](https://pypi.org/project/geometry-to-spatialite/) library, so I decided to investigate a few different options.

## MkDocs

In the MkDocs ecosystem there are now several plugins that allow you to incorporate docstrings into markdown documentation.

### [Mkdocstrings](https://github.com/pawamoy/mkdocstrings)

Mkdocstrings currently has one handler which parses docstrings written in the popular Google style e.g:

```py
"""
Function description

Args:
    param1 (int): The first parameter.
    param2 (str): The second parameter.

Returns:
    bool: True for success, False otherwise.
"""
```

At the time of writing the documentation says "This project currently only works with the Material theme". This project is young, but it does seem to have some traction around it and I wouldn't be surprised to see support for more themes and docstring styles added. The tabular rendered outputs of this plugin are really nice and it does a great job of mixing information about type hints, required params and default values from the function declarations with the descriptions from the docstrings. This minimises the need to repeat this information.

### [MkAutoDoc](https://github.com/tomchristie/mkautodoc)

MkAutoDoc takes different approach. MkAutoDoc doesn't support any of the popular python docstring formats (Google, reST, NumPy). Instead it allows you to embed markdown in code comments and include those in your documentation. One the one hand, this allows you to really go all-in on markdown on a completely new project, but on the other it doesn't set out any particular structure for arguments, return types, etc (and hence can't do anything smart with them, like mkdocstrings does), and if you've already got docstrings in your code in any of the common formats, it won't parse them.

MkAutoDoc is primarily used by projects under the [encode](https://github.com/encode) organisation to build the docs for projects like [httpx](https://www.python-httpx.org/) and [starlette](https://www.starlette.io/).

### [Jetblack-markdown](https://github.com/rob-blackbourn/jetblack-markdown)

Honourable mention for this one. Jetblack-markdown uses [docstring_parser](https://github.com/rr-/docstring_parser) under-the-hood so in theory should work with reST, Google, and Numpydoc-style docstrings, although the test suite currently only covers the Google-style. It does also require a bit of extra CSS for styling. I gave this one a quick go with the readthedocs theme and it produces attractive rendered outputs. This one is a very young project, but worth keeping an eye on 👀

## Sphinx

Meanwhile in the Sphinx ecosystem, there are already good solutions for working with docstrings in the form of the [Autodoc](https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html) extension to parse docstrings and include them in the rendered output and [Napoleon](https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html) which adds support for the Google and NumPy formats. There's also a new package that helps us out on the free text side.

### [MyST Parser](https://myst-parser.readthedocs.io)

MyST parser is a markdown flavour and parser for Sphinx. This is a bit of a game changer because the sphinx ecosystem has years of maturity and a large plugin ecosystem and MyST basically solves my biggest issue with sphinx. The main downside of MyST parser is that it is a slightly leaky abstraction. To access some plugin functionality, we have to drop in an `{eval-rst}` block and other Sphinx extensions still assume your source content is reST, so although it is possible to write the main body of text in markdown, you won't be able to use markdown _everywhere_.

## Conclusion

Predictably, there isn't a single obvious conclusion. There are no solutions, only tradeoffs. However in 2020 there are multiple viable options for documenting your python project with markdown 🎉

After conducing this roundup, I wrote a [docs site for geometry-to-spatialite](https://chris48s.github.io/geometry-to-spatialite/) using

* [Sphinx](https://www.sphinx-doc.org)
* [MyST Parser](https://myst-parser.readthedocs.io) for markdown parsing
* [Autodoc](https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html) to parse docstrings and include them in the rendered output
* [Napoleon](https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html) to pre-process Google-style docstrings for autodoc
* [ghp-import](https://pypi.org/project/ghp-import/) to deploy the generated HTML to github pages


Being able to use sphinx and write (mostly) markdown seems like a good place to be for now, but there are multiple projects in the MkDocs ecosystem which I'll be monitoring and revisting as they mature and gain traction.
