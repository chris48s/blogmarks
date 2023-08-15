<!--
.. title: time-machine
.. slug: time-machine
.. date: 2020-06-06 00:00:00
.. tags: python,testing
.. category: 
.. link: 
.. description: 
.. type: text
-->

I usually reach for [freezegun](https://github.com/spulec/freezegun)
when I need to mock calls to the standard library datetime functions,
but Adam Johnson explains in this recent
[blog post](https://adamj.eu/tech/2020/06/03/introducing-time-machine/)
why he has chosen to create a new library for this.
[Time-machine](https://github.com/adamchainz/time-machine) uses a similar
decorator-based syntax to freezegun but promises substantial performance
gains with large codebases.

I'm looking forward to giving this a spin next time I need to write a test
that mocks the current time.
