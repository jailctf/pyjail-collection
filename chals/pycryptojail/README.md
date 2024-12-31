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
...   print(i, chr(i))
97 a
98 b
99 c
100 d
101 e
102 f
103 g
104 h
105 i
106 j
107 k
108 l
109 m
110 n
111 o
112 p
113 q
114 r
115 s
116 t
117 u
118 v
119 w
120 x
121 y
122 z
...
(some output omitted for brevity)
...
119834 𝐚
119835 𝐛
119836 𝐜
119837 𝐝
119838 𝐞
119839 𝐟
119840 𝐠
119841 𝐡
119842 𝐢
119843 𝐣
119844 𝐤
119845 𝐥
119846 𝐦
119847 𝐧
119848 𝐨
119849 𝐩
119850 𝐪
119851 𝐫
119852 𝐬
119853 𝐭
119854 𝐮
119855 𝐯
119856 𝐰
119857 𝐱
119858 𝐲
119859 𝐳
...
```

see how the `119834 𝐚` is divisible by 2 but the `97 a` is not? the alphabets are off-sync for their divisibilities, so we can freely use all alphabet chars.

importantly, we can input a singular left parenthesis `(`, but it has an ord of 40 so we cannot use any other divisible by 2 chars. this is fine since `breakpoint()` with nfkc should be sufficient.
but how do we pad our python code with chars to make it have a product `8`?

well, we have access to the comment char, so we can add our padding to manipulate the product to be `8` after the comment

todo ping me on discord if this writeup is not done yet and you want the writeup
