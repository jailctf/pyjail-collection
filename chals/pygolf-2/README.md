# pygolf-2

## challenge

```py
#!/usr/bin/python -S
code = input()
for omegalul in ['breakpoint', 'input', 'open', '__import__']:
    del __builtins__.__dict__[omegalul]
sum(not c.isspace() for c in code) < 25 != eval(code)
```

## solution

i figured that using eval on a sliced repr'd stupid whitespace string would be the answer. my first attempt was `eval(eval(repr("stuff here"[::9])))` but that didn't work

also important to note ligatures aren't possible here since it triggers an import, and `__import__` was deleted


todo finish
