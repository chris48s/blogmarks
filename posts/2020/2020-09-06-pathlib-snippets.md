<!--
.. title: Three useful pathlib snippets
.. slug: pathlib-snippets
.. date: 2020-09-06 00:00:00
.. tags: python
.. category: 
.. link: 
.. description: 
.. type: text
-->

```py
from pathlib import Path
p = Path('/foo/bar')


# Read in a text file
text = (p / 'file.txt').read_text()

# Recursively list all .csv files in directory
csvs = list(p.rglob('*.csv'))

# Iterate files or subdirectories
files = [i for i in p.iterdir() if i.is_file()]
subdirs = [i for i in p.iterdir() if i.is_dir()]
```

Maybe that's three and a half 🙂
