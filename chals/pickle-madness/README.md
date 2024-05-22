# pickle-madness

only slightly related to pickle jails

## challenge

main.py
```py
#!/usr/bin/env python3

import pickle
import string

inp = input("enter your pickle> ").split()

assert not "h" in "".join(inp)
# assert not "." in "".join(inp)

for substring in inp:
  for i in range(len(substring)):
    if i % 2 == 0:
      assert substring[i] in string.printable.lower(), i
    else:
      assert substring[i] in string.printable.upper() + "osystem", i # yw

pickle.loads("\n".join(inp).encode())
```

## solution

we can get a shell by putting a `GLOBAL + b"os system " + MARK + UNICODE + b'$0 ' + TUPLE + REDUCE'`

we use `" "` instead of `"\n"` because it will get split later

the `$0` var is a string with (`"sh"`) i think for some reason
