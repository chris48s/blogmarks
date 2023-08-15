<!--
.. title: pipx
.. slug: pipx
.. date: 2020-08-15 00:00:00
.. tags: python,packaging
.. category: 
.. link: 
.. description: 
.. type: text
-->

Poetry is my go-to solution for managing libraries and project dependencies in python, but there are also a variety of python-based CLI tools like [aws-cli](https://github.com/aws/aws-cli) and [tldr](https://pypi.org/project/tldr/) that I want globally available. But what if they have incompatible dependencies? No worries - [pipx](https://pipxproject.github.io/pipx/) has got your back. Pipx installs packages into an isolated virtual environment for each application, keeping their dependencies separate and your global environment clean. This makes the applications globally available while avoiding the possibility of conflicts in the global environment.
