<!--
.. title: pspg - Postgres Pager
.. slug: pspg
.. date: 2020-05-28 00:00:00
.. tags: terminal,sql
.. category: 
.. link: 
.. description: 
.. type: text
-->

[pspg](https://github.com/okbob/pspg) is a pager with support for tabular data

Install it:

```bash
sudo apt install pspg
```

Then set it as the default pager for `psql` in `~/.psqlrc`:

```text
\x auto
\pset pager on
\pset linestyle unicode
\pset border 2
\setenv PAGER 'pspg -bX --no-mouse'
```

This is particularly useful when combined with [yesterday's post](link://slug/ssh-tunnel)
