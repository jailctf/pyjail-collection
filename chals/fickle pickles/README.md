# fickle pickles

author is A~Z, of course (they make good pickle challenges)

source is [here](https://github.com/maple3142/imaginaryCTF-solution/blob/master/2021-08/frikle_pickles)

## challenge

pickles.py
```py
from pickle import _Unpickler as EverSoSecureUnpickler
from base64 import b64decode as extract_base_64


class NotAPickle(EverSoSecureUnpickler):
    from io import BytesIO

    def __init__(self, pickled):
        super().__init__(NotAPickle.BytesIO(pickled))
        self.step = 0

    def find_class(self, module, name):
        self.step += 1

        if module != "__main__" or "ex" in name.lower() or "ev" in name.lower():
            return "Don't even think about it"

        if self.step == 0:
            if name == "flag":
                return "Good boy", parts[2]
            else:
                return "You're bad at this >:("

        if self.step == 1:
            if name == "fleg":
                return ["Is it the flag? Is it not? Is it?", parts[1]]
            else:
                return "Wtf are you trying to hack me?"

        if self.step == 2:
            if name == "flog":
                return {"Can you get the flag out?": parts[0]}
            else:
                return "You should be used to it by now..."

        if self.step == 3:
            # TODO: think about snarky comment
            if name == "flig":
                return (
                    f"This one's gonna be a tad more difficult to extract: {parts[3]}"
                )

        if self.step >= 4:
            # Last but clearly least
            return parts[4]

        # Robustness requires default behavior even though it's impossible to get here
        return super().find_class(module, name)

    # We like light syntax so might as well make good use of it
    def __call__(self):
        return self.load()


def partition_flag():
    with open("flag.txt", "r") as file:
        flag = (
            file.read().strip()
        )  # Since we take security seriously, we partition the flag into 5 parts and only send them one at a time

    return [flag[i : i + 8] for i in range(0, len(flag), 8)]


# I've been told that was good practice
if __name__ == "__main__":
    parts = partition_flag()
    assert len(parts) == 5

    try:
        pickle = NotAPickle(extract_base_64(input("Where are my pickles? ")))()
        assert type(pickle) == str  # Lots of fickle fake pickles out there :c
    except:
        pickle = None

    if "".join(parts) == pickle:
        print(
            "You pickled my pickle! Was it your own doing?\nHere's your reward:",
            "".join(parts),
        )
    else:
        print("Your pickles are fickle! >:~(")
```

## solution

todo
