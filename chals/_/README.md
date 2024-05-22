# _

## challenge

main.py
```py
from builtins import open as _, input as _a, print as _b, eval as _c
from unicodedata import normalize as _d
from re import search as _e

if _e("[\\101-\\132\\141-\\172]", __:=_d("\116\106\113\104", _a(_("_").read()))):
    _b("|\\|0 3|\\|+|2`/")
    ______________________
_b(_c(__))
```

## solution

You can use open, because it's imported as _. However, you can't call read, because read has letters. You can bypass this because file objects are iterators over the lines of the file. Using * to iterate over the file and cast the result to the tuple yields the flag.

```
[*_("\146\154\141\147\056\164\170\164")]
```

