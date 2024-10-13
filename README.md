# pyjail collection

## why

i love pyjails

## pyjails

|title|source|description/tldr|
|-|-|-|
|[minijail](./chals/minijail)|ImaginaryCTF Round 9|eval with only the print builtin in 37 bytes|
|[tax evasion](./chals/tax-evasion)|ImaginaryCTF Round 11|python audit hook function overwrite|
|how r u|ImaginaryCTF Round 12|todo|
|[Fickle Pickles](./chals/fickle-pickles)|ImaginaryCTF Round 13|weird pickle challenge|
|[prison](./chals/prison)|ImaginaryCTF Round 18|standard no builtins escape but with blacklist|
|[Stackless Jail](./chals/stackless-jail)|ImaginaryCTF Round 23|read sys environ with only stack size 1 from dis.code\_info|
|[Nameless Jail](./chals/nameless-jail)|ImaginaryCTF Round 24|use dictionary instead of multiple variables|
|[\_](./chals/_)|ImaginaryCTF Round 25|use `*` for unpacking a file object|
|[dont-repeat-yourself](./chals/dont-repeat-yourself)|ImaginaryCTF Round 28|only use each char once|
|No Comment|ImaginaryCTF Round 32|todo|
|[decorated](./chals/decorated)|ImaginaryCTF Round 33|use decorators on functions|
|Breaking in the jail|ImaginaryCTF Round 34|pickle jail, chain builtin modules with dependencies to get to os|
|Revenge is best served pickled|ImaginaryCTF Round 34|pickle rev if i recall|
|My Little Jail|ImaginaryCTF Round 35|todo|
|PyCryptoJail|ImaginaryCTF Round 36|discrete logarithm and nfkc normalization abuse|
|Exceptional Pyjail|ImaginaryCTF Round 37|todo|
|Safe Pickle|ImaginaryCTF Round 38|vulnerability in [picklescan](https://github.com/mmaitre314/picklescan)|
|[pickle-madness](./chals/pickle-madness)|ImaginaryCTF Round 43|bash jail disguised as pickle jail|
|[Low Security Jail](./chals/low-security-jail)|ImaginaryCTF Round 44|eval blacklist overwrite|
|[pygolf](./chals/pygolf)|ImaginaryCTF Round 44|NFKC normalization op|
|Completely new challenge|ImaginaryCTF Round 47|todo|
|pickle overflow|ImaginaryCTF Round 49|utf8 str encoded can be longer than 2 bytes per char|
|pygolf 2|ImaginaryCTF Round 49|todo|
|[You shall not call!](./chals/you-shall-not-call)|ImaginaryCTF 2023|BUILD opcode abuse into unpickler attr overwrite with cheese-ish|
|[You shall not call Revenge](./chals/you-shall-not-call-revenge)|ImaginaryCTF 2023|BUILD opcode abuse into unpickler attr overwrite|
|[Get and set](./chals/get-and-set)|ImaginaryCTF 2023|pydash (almost) arbitrary get and set|
|[pyquinejailgolf](./chals/pyquinejailgolf)|amateursCTF 2024|todo|
|[pyquinejailgolf 2](./chals/pyquinejailgolf-2)|amateursCTF 2024|todo|
|[Just Another Pickle Jail](./chals/just-another-pickle-jail)|Sekai CTF 2023|todo|
|[introspection](./chals/introspection)|angstromCTF 2024|find discrepancies between pickle c impl and python impl (python2)|
|[llama jail](./chals/llama-jail)|vsCTF 2024|bypass ast visitor when ast.iter_child_nodes|
|[llama jail revenge](./chals/llama-jail-revenge)|vsCTF 2024|complicated|
|calc|ImaginaryCTF 2024|bypass audit hook with signal|
|ok-nice|ImaginaryCTF 2024|division by 0 side channel|
|ASTea|UIUCTF 2024|bypass simple ast thingy|
|prison|ifCTF 2023 finals|todo|
|repickle|CyberSpace CTF|abuse bug in pickler|
|parseltongue|jailCTF 2024|todo|
|smiley-faiss|jailCTF 2024|todo|
|parity 1|jailCTF 2024|todo|
|parity 2|jailCTF 2024|todo|
|MMM|jailCTF 2024|todo|
|filter'd|jailCTF 2024|todo|
|charredcoal|jailCTF 2024|todo|
|what numbers?|jailCTF 2024|todo|
|polyglo7quine|jailCTF 2024|todo|
|what flag?|jailCTF 2024|todo|
|respy evil challenge|jailCTF 2024|todo|
|void|jailCTF 2024|todo|
|respy nice challenge|jailCTF 2024|todo|
|functional programming|jailCTF 2024|todo|
|jellyjail|jailCTF 2024|todo|
|lost in transit|jailCTF 2024|todo|
|computer-monitor|jailCTF 2024|todo|
|pickled magic|jailCTF 2024|todo|
|stupid crypto chall|jailCTF 2024|todo|
|get and call|jailCTF 2024|todo|
|no nonsense|jailCTF 2024|todo|
|last message|jailCTF 2024|todo|
|axed|TCP1P CTF 2024|python2 `__metaclass__` abuse|
|sym|TCP1P CTF 2024|todo|
|typically not a revenge|TCP1P CTF 2024|sys & builtins nuked, `a-zA-Z._[]` allowed|
|functional|TCP1P CTF 2024|challenge copied from jailctf but with rce|
