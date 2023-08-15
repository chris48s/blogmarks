<!--
.. title: git --word-diff
.. slug: git-word-diff
.. date: 2021-02-25 00:00:00
.. tags: terminal,git,terminal
.. category: terminal
.. link: 
.. description: 
.. type: text
-->

Git's default diff behaviour can make it difficult to parse edits to long lines of text. This is often an issue when editing documentation

![unhelpful git diff](/images/word-diff1.png)
_Thanks git, but what actually changed?_

In these situations, the [`--word-diff`](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---word-diffltmodegt) option can be used to generate a diff that is easier to read.

![helpful git diff](/images/word-diff2.png)
_Aah that's better!_
