---
layout: post
title: Get the size of a HTTP response
categories: ['terminal']
tags: ['terminal', 'curl']
---

## Get the size of a HTTP response

Vanilla verison:

```sh
curl "https://foo.bar/file" -w 'size: %{size_download}\n' -o /dev/null
```

or with a little extra flourish:

```sh
curl "https://foo.bar/file" -w 'size: %{size_download}\n' -o /dev/null | sed 's/[^0-9]//g' | numfmt --to=iec
```
