# don't repeat yourself

## challenge

There's many software design principles you should follow in order to write clean code. In this challenge, please make sure you DRY, thank you and good luck!

dry.py
```py
#!/usr/bin/env python3

while True:
    inp = input('>>> ')
    check = 0
    used = set()
    for c in inp:
        check ^= ord(c)
        if c in used:
            print("DRY!")
            exit()
        used |= {c}
    if check:
        print("Non-zero checksum!")
        exit()
    exec(inp)
```

## intended solution

```py
y=input()# 0
eval(y)#E
```

You have access to full builtins, so eval(input()) split across two payloads solves the challenge. You can bypass the checksum with a comment at the end of each line.

## unintended solution

`breakpoint()#1`
