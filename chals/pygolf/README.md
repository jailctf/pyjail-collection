# pygolf

made by lydxn

## challenge

main.py
```py
#!/usr/bin/python3 -S
flag = 'ictf{redacted}'
eval(input()[:8])
```

## my solution (unintended)

`{}[flag]` leaks flag through stderr

## solution

Python performs NFKC normalization on identifiers before parsing them. Use this to write flag in three chars via the ﬂ ligature. int() leaks the flag through stderr.
