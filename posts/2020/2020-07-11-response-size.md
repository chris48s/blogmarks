<!--
.. title: Get the size of a HTTP response
.. slug: response-size
.. date: 2020-07-11 00:00:00
.. tags: terminal,curl,terminal
.. category: terminal
.. link: 
.. description: 
.. type: text
-->

Vanilla verison:

```bash
curl "https://foo.bar/file" -w 'size: %{size_download}\n' -o /dev/null
```

or with a little extra flourish:

```bash
curl "https://foo.bar/file" -w 'size: %{size_download}\n' -o /dev/null | sed 's/[^0-9]//g' | numfmt --to=iec
```
