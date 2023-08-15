<!--
.. title: A python CLI app which accepts input from stdin or a file
.. slug: stdin-or-file
.. date: 2020-12-07 00:00:00
.. tags: python,terminal
.. category: 
.. link: 
.. description: 
.. type: text
-->

This excellent guide on [Command Line Interface Guidelines](https://clig.dev/) has been shared widely over the last few days and it includes a huge variety of advice on writing elegant and well-behaved command line tools. The whole thing is well worth a read, but I'm going to pick out one quote to focus on in this post:

> **If your command is expecting to have something piped to it and stdin is an interactive terminal, display help immediately and quit.** This means it doesn't just hang, like `cat`.

Tools that can optionally accept input via pipe are very useful and this pattern is entirely possible with python and argparse, but not completely obvious. Here's a simple example of a python program which:

- Can accept input as a file: `./myscript.py /path/to/file.txt`
- Can accept input via a pipe: `echo 'foo' | ./myscript.py`
- Will print help and exit if invoked interactively with no arguments: `./myscript.py` (instead of just hanging and waiting for input)

```py
#!/usr/bin/env python3

import argparse
import sys

def main():
    parser = argparse.ArgumentParser(
        description='A well-behaved CLI app which accepts input from stdin or a file'
    )
    parser.add_argument(
        'file',
        nargs='?',
        help='Input file, if empty stdin is used',
        type=argparse.FileType('r'),
        default=sys.stdin,
    )
    args = parser.parse_args()

    if args.file.isatty():
        parser.print_help()
        return 0

    sys.stdout.write(args.file.read())
    return 0

if __name__ == '__main__':
    sys.exit(main())
```
