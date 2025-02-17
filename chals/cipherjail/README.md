# cipherjail

## challenge

"You've been cipherjailed, but the guard is nice and will let you say short words, including contractions."

```py
#!/usr/local/bin/python3
from secrets import choice

flag = 'REDACTED'
# keep non-flag var names over 4 chars
attempts = 30
alphabet = 'abcdefghijklmnopqrstuvwxyz\''

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

for _ in range(attempts):
    try:
        print(eval(sanitize(input('> '))))
    except Exception:
        print('NICE GUARD: cool')
```

## solution

intended solution: Send "aa", "bb", etc. until the nice guard doesn't say cool. Then, you know the apostrophe. Use that to send things that cipher+eval to two-character strings, until you have the letters that cipher to "flag". It may take multiple attempts if you are unlucky.
