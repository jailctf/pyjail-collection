# pycryptojail

## challenge

```py
#!/usr/bin/env python3

from numpy import prod
from sympy import divisor_count
fun = lambda *x: divisor_count(prod(x))

print('''Welcome to PyCryptoJail!
Please enter a fun(1337) command.
The flag is in ./flag.txt.''')

command = input("> ").encode()
if fun(*command) == fun(1337):
    exec(command)
else:
    print('Invalid command')
```

## solution

`fun(1337)` is `4` (it is a constant)

when the bytes of our input are product'd, the divisor count has to be 4. this looks impossible, but we can abuse numpy using int64 in prod

```
>>> prod([2**32,2**31])
-9223372036854775808
```

when we add a char, our previous product is multiplied mod `2**64`. we can just say that negative numbers have `2**64` added onto them (the highest bit in the actual number is used for the sign, so the number is negative). for example,

```
>>> list(b'abcdefghijk')
[97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108]
>>> prod(list(b'abcdefghijk'))
8811889446083713280
>>> ord('l')
108
>>> (8811889446083713280 * 108)%2**64
10900112417853901824
>>> ((8811889446083713280 * 108)%2**64) - 2**64  # but the highest bit is used for the sign, so we have to subtract by 2**64
-7546631655855649792
>>> prod(list(b'abcdefghijkl'))  # it is the same!!!
-7546631655855649792
```

one important thing is that because this is all done `2**64`, if we multiply by `2`, we cannot recover from that by multiplying more things. the `0` bit at the end of all subsequent products will remain there.

remember that we have to make it have 4 divisors exactly. `8` has 4 divisors, and so if our product result is `8`, then that means there are 3 factors of 2. we have some room here to input some divisible by 2 chars.

we have to be careful about what chars we use. we would want to avoid chars divisible by two. unfortunately, we cant do much here if we use just regular ascii chars.
we will have to use nfkc normalized chars because they might have different divisibilities. nfkc chars work because python treats them as regular chars.
you might often see some of these chars from that stupid lingojam text generator [here](https://lingojam.com/FancyTextGenerator) or maybe somewhere else idk

i used a script to find these.

```py
>>> for i in range(0x110000):
...  if normalize('NFKC', chr(i)) in alpha:
...   if prod(list(chr(i).encode())) % 2 == 1:
...    print(chr(i), i)
...
a 97
c 99
e 101
g 103
i 105
k 107
m 109
o 111
q 113
s 115
u 117
w 119
y 121
ſ 383
ˡ 737
ˣ 739
ᵃ 7491
ᵇ 7495
ᵉ 7497
ᵍ 7501
ᵏ 7503
ᵗ 7511
ᵛ 7515
ᵣ 7523
ᵥ 7525
ａ 65345
ｃ 65347
ｅ 65349
ｇ 65351
ｉ 65353
ｋ 65355
ｍ 65357
ｏ 65359
ｑ 65361
ｓ 65363
ｕ 65365
ｗ 65367
ｙ 65369
```

luckily with nfkc, we have unlocked some new chars such as `x`. with this, we can get the builtin `exec`

importantly, we can input a singular left parenthesis `(`, but it has an ord of 40 so we cannot use any other divisible by 2 chars. `)` does not have any factors of two, so we can do one function call.

**todo finish string manipulation section**

but how do we pad our python code with chars to make it have a product `8`?

well, we have access to the comment char since it is not divisible by 2, so we can add our padding to manipulate the product to be `8` after the comment. we can find the correct thing by using discrete logs.

**todo finish dlog using lattices ?? idk what i am doing**

ping me on discord if you want the writeup
