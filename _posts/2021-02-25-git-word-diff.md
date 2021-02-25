---
layout: post
title: git --word-diff
categories: ['terminal']
tags: ['terminal', 'git']
---

## git --word-diff

Git's default diff behaviour can make it difficult to parse edits to long lines of text. This is often an issue when editing documentation

![unhelpful git diff]({{site.baseurl}}/assets/word-diff1.png)
_Thanks git, but what actually changed?_

In these situations, the [`--word-diff`](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---word-diffltmodegt) option can be used to generate a diff that is easier to read.

![helpful git diff]({{site.baseurl}}/assets/word-diff2.png)
_Aah that's better!_
