# exceptional pyjail

## challenge

```py
import string
import sys

a = input()
if any(i in a for i in ["import", "__", "[", "]", ".", "exec", "eval"]):
  print("not allowed")
  sys.exit()
if any(i not in string.printable for i in a):
  print("no")
  sys.exit()
with open("flag.txt") as file:
  flag = file.read()
sys.stdout = "lol"
del sys
exec(a)
```

## solution

intended is to `raise SystemError(flag)` since stderr is still available

my sol was `int(flag)`
