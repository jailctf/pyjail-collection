# pyquinejailgolf 2

## challenge

```
#!/usr/local/bin/python3

import time
from tempfile import TemporaryDirectory

with TemporaryDirectory() as workdir:
    __import__('os').chdir(workdir)
    __import__('os').mkdir('runtime')

    print("Enter a reverse quine. It must print its source code backwards (including any trailing newlines).")
    print("It must be non-empty and contain 0 quotation marks or dunders. All ascii btw. oh and no builtins.")
    print("To prove you wrote it yourself, it must contain the string 'pyquinejailgolf', and be ==343 chars.")
    print("Well...I'm not sure how you're supposed to write a quine without print statements. u can have em.")
    print("~~~~~~~~~~~~~~~~~~~~~~~~~~~ Type <END> to terminate the program input ~~~~~~~~~~~~~~~~~~~~~~~~~~~")

    prog = []
    while (x := input(">>> ")) != "<END>":
        prog.append(x)
    program = "\n".join(i for i in prog)

    assert x == "<END>", "what did you do this time."
    assert all(ord(i) < 128 for i in program), "i don't speak foreign languages."
    assert all(i not in program for i in ['"', "'", '_']), "who uses strings anyway? it's not like quines require strings."
    assert program != "", "scuse me, just cleaning out the garbage."
    assert "pyquinejailgolf" in program, "are you sure you wrote this program yourself?"
    assert len(program) == 343

    goal = time.time_ns() + 5000000000
    try:
        with open('runtime/external_run.py', 'w+') as f:
            f.write(f"""
{program = }
safe_builtins = {{}}
for i in dir(__builtins__):
    if i[0] not in __import__('string').ascii_lowercase + "_":
        safe_builtins[i] = eval(i)
safe_builtins['print'] = print
new_builtins = {{'__builtins__':safe_builtins}}
try:exec(program, new_builtins, new_builtins)
except:pass""")
        handle = __import__('subprocess').run('timeout 2.5 /usr/local/bin/python3 runtime/external_run.py', shell=True, capture_output=True, encoding='utf-8')
    except:
        pass

    time.sleep(max(0, (goal - time.time_ns())/1e9))
  
    if (handle.stdout[::-1]) == program:
        print("good jorb")
    else:
        exit("bad")

    with open('/app/flag.txt') as f:
        print(f.read())
```

## solution

i recommend taking a look at [pyquinejailgolf 1](../pyquinejailgolf) before taking a look at this

let's take a look at what builtins we have access to for this one

```py
['ArithmeticError', 'AssertionError', 'AttributeError', 'BaseException', 'BlockingIOError', 'BrokenPipeError', 'BufferError', 'BytesWarning', 'ChildProcessError', 'ConnectionAbortedError', 'ConnectionError', 'ConnectionRefusedError', 'ConnectionResetError', 'DeprecationWarning', 'EOFError', 'Ellipsis', 'EncodingWarning', 'EnvironmentError', 'Exception', 'False', 'FileExistsError', 'FileNotFoundError', 'FloatingPointError', 'FutureWarning', 'GeneratorExit', 'IOError', 'ImportError', 'ImportWarning', 'IndentationError', 'IndexError', 'InterruptedError', 'IsADirectoryError', 'KeyError', 'KeyboardInterrupt', 'LookupError', 'MemoryError', 'ModuleNotFoundError', 'NameError', 'None', 'NotADirectoryError', 'NotImplemented', 'NotImplementedError', 'OSError', 'OverflowError', 'PendingDeprecationWarning', 'PermissionError', 'ProcessLookupError', 'RecursionError', 'ReferenceError', 'ResourceWarning', 'RuntimeError', 'RuntimeWarning', 'StopAsyncIteration', 'StopIteration', 'SyntaxError', 'SyntaxWarning', 'SystemError', 'SystemExit', 'TabError', 'TimeoutError', 'True', 'TypeError', 'UnboundLocalError', 'UnicodeDecodeError', 'UnicodeEncodeError', 'UnicodeError', 'UnicodeTranslateError', 'UnicodeWarning', 'UserWarning', 'ValueError', 'Warning', 'ZeroDivisionError']
```

we have access to all of the error classes. with these, we can get error objects as well. these error objects have the `args` attribute which contain strings!

so, we can get some strings.

![yayayay](https://github.com/quasar098/pyjail-collection/assets/70716985/cd38798d-800f-47c3-a5c2-137b561121f5)

but also, python 3.10 added the name and obj attributes to python objects, so we can get arbitrary attributes using format strings by having them raise an attribute error from the format string and then getting the object as shown below

```py
try:
    "{0.__getattribute__.argthatdoesnotexist}".format(object)
except Exception as e:
    print(e.obj)  # <slot wrapper '__getattribute__' of 'object' objects>
```

with getattribute, we can get the `__traceback__` attribute of an exception object, and then the `tb_frame` attribute of that. then, we walk up the call stack and print out the program in reverse.

the final payload is below (it was made by RJCyber)

```py
try:{}%9
except(q:=Exception)as e:c=e.args[0][32:39:6]
try:QQgetattributeQQZQQtracebackQQZtbQframeZfQbackZfQglobalsZprogram
except q as e:s=e.name.replace(c%81,c%95).split(c%90)
*pyquinejailgolf,o=q.mro()
try:((c*3+s[0]+c*3)%(123,48,46,46,65,125)).format(o)
except q as e:f=e.obj;print(f(f(f(f(e,s[1]),s[2]),s[3]),s[4])[s[5]][::-1],end=c[1:1])
```
