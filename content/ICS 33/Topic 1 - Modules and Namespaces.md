---
title: Topic 1 - Modules and Namespaces
date: 2024-03-08
---

Detailed Notes can be found at: [Modules and Namespaces Notes](https://ics.uci.edu/~thornton/ics33/Notes/ModulesAndNamespaces/)

#### Python's Built-Ins

- Python has an inherent set of "built-ins" that allow interfacing with the language.
- You can find out what's available by calling the `dir()` function (short for directory), with the ability to use the elements of `dir` dynamically in the Python shell.

  - When calling `dir()` we find `__builtins__: dict` to be an element, but what is it?
    - The keys of that dictionary are all _names_ that are _built-in_ to Python in some way or another.
    - We can even directly refer to these names through the python shell, where `__builtins__['False'] is False` returns `True`, signifying that they are in fact the same object!

- Considering dictionaries are mutable objects (Remember ICS32A), we can do more with it.
  - We can change what is "built-in" to python by redefining elements, adding new functions, etc.

```python
__builtins__["test"] = lambda x: f'Testing {x}"
test('hello')
>>> "Testing hello"
del __builtins__['abs']
abs(-3) # now this will raise a NameError!
```

- This is a characterization of the dynamicism within Python!

#### Scopes, Namespaces, Functions

- When we define a new variable, function, class, etc., they are all put into the `dir()`. Deleting a variable removes it from the directory as well!

  - Even for functions, `def` gives a function a specific name, while `lambda` keeps it as a `lambda` type.

- "Resolving an Identifier" (or "deciding what value is being talked about") is a thing that python does, and lets understand _how_
  - First, look at `globals()` and `locals()` in Python to figure out scopes.

This example script:

```python
print('In example module')
print(f'  globals: {sorted(globals().keys())}')
print(f'   locals: {sorted(locals().keys())}')


def foo(n):
    def bar(m):
        print('In bar function')
        print(f'  globals: {sorted(globals().keys())}')
        print(f'   locals: {sorted(locals().keys())}')
        return n + m

    print('In foo function')
    print(f'  globals: {sorted(globals().keys())}')
    print(f'   locals: {sorted(locals().keys())}')
    return bar(4)


print('Preparing to call foo function')
print(f'  globals: {sorted(globals().keys())}')
print(f'   locals: {sorted(locals().keys())}')
print('Calling foo function')
print(f'foo function returned {foo(2)}')
```

has this output:

```python
In example module
  globals: ['__annotations__', '__builtins__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__']
   locals: ['__annotations__', '__builtins__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__']
Preparing to call foo function
  globals: ['__annotations__', '__builtins__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'foo']
   locals: ['__annotations__', '__builtins__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'foo']
Calling foo function
In foo function
  globals: ['__annotations__', '__builtins__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'foo']
   locals: ['bar', 'n']
In bar function
  globals: ['__annotations__', '__builtins__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'foo']
   locals: ['m', 'n']
foo function returned 6
```

This is a natural example of LEGB rule in Python, and the fact that if I defined a new variable "n" inside of a local scope in the `bar(m)` function, it would overshadow `n` from the enclosing scope.

#### Modules and Importation

- What happens when we import a module?
  - `import math` adds to the namespace assosciated with the current scope, so we saw `math` appear in the results of `dir()`
  - `from math import (func)` brings `func()` into `dir()` directly.
  - `from math import *` brings EVERYTHING from math into `dir()` directly. This shows why we shouldn't be doing that in any way.
- If you import in scopes that aren't global, then you get interesting results where the `locals()` contain the module but `globals()` dont.
  - If you have a function that relies on one module, you can try to strictly add it to that function's local namespace.
- That's what namespaces are after all. Just the outputs of `dir()` in a certain _scope_.

- If we import a module twice with the same name, all we are doing is adding two names pointing to the same module.

```python
import math as m1
import math as m2

m1.sqrt is m2.sqrt
>>> True # because they are the same module!
```

This takes a minute to wrap your head around, so run the examples in the course notes!
