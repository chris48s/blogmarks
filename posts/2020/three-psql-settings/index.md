<!--
.. title: Three useful psql settings
.. slug: three-psql-settings
.. date: 2020-11-26 00:00:00
.. tags: sql
.. category: 
.. link: 
.. description: 
.. type: text
-->

* By default `psql` doesn't display `NULL`s, but you can configure it to show NULLs as a printable character or string e.g: `\pset null '█'` or `\pset null '(null)'`. This makes it is easier to distinguish `NULL` from empty string.
* `\timing on` appends an execution time to each result e.g: `Time: 0.412 ms`. `\timing off` turns it off.
* By default, `psql` stores every line in history. Setting `\set HISTCONTROL ignoredups` ignores lines which match the previous history line. This is a common default for shells and REPLs.

These can be used on an ad-hoc basis or added to `.psqlrc`.
