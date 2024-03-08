---
title: Topic 4 - Context Managers
date: 2024-03-08
---

- Full Notes can be found here: [Context Managers](https://ics.uci.edu/~thornton/ics33/Notes/ContextManagers/)

- There are many situations in Python where we want a function to clean up after itself.
- For example, file line counting.
  - We open and loop through a file and count each line.
  - The issue is that there are _so many_ ways for something to go wrong
  - The usual way to handle issues with that would have to be a _try-except-finally_ loop. For example:

````python
```python
def count_lines_in_file(file_path, encoding = 'utf-8'):
    the_file = open(file_path, 'r', encoding = encoding)

    try:
        lines = 0

        for line in the_file:
            lines += 1

        return lines
    finally:
        the_file.close()
````

- This kind of setup makes sure that if the file can be opened, any future errors will be cleaned up. But _it isn't peak design_. Why? Because what if the file can't be opened.

- How can we make sure everything gets wrapped up properly _regardless_ of error?
- **Context Managers**
  - Context Managers automate a _wrap-up_ process. We control what goes on inside the `with` with the Context Expression that returns a _Context Manager_, that controls the _Context_ of the file.

```python
def count_lines_in_file(file_path, encoding="utf-8"):
	with open(file_path, 'r', encoding=encoding) as the_file:
		lines = 0
		for _ in the_file:
			lines += 1
		return lines

# regardless of how we exit this "the_file" context manager, we will *always* close up the file properly. We leave that context manager or that "with" statement when control slips out of the hands of the context manager.

# this can be either an error, ending of code, etc.

# making sure that stuff cleans up on the *way out* is a great practice of building good and robust software.
```

- We can employ context managers for _more_ than just Files, Sockets, etc.
  - We can even do this in Errors!

```python
import some_module as sm
import unittest

class TestSM(unittest.TestCase):
	def test_sm_function_one(self):
		self.assertEqual(sm.say_hi(), 'Hi')

	# what if we have to test an error?
	def test_sm_function_one_returns_string(self):
		# we can employ a context manager!
		with self.assertRaises(TypeError):
			int(sm.say_hi()) # 'Hi' won't become an 'int', so this works!
```

- We can now also block prints!

```python
import contextlib
import io

def print_hello():
	print("hello")

with contextlib.redirect_stdout(io.StringIO()) as output:
	print_hello()

output.getvalue()
>>> 'hello\n'
```

#### How can we write a Context Manager?

- We can follow the _Context Manager Protocol_
  - Following this protocol, Context Managers can do two things:
    1. They can be _entered_ and told when that happens
    2. They can be _exited_, whether that is with or without failure
       - They will be told what kind of failure (if it happens)
- So we need to have `__enter__(self)` and `__exit__(self, exc_type, exc_value, exc_traceback)`
- Now, let's write a context manager!

```python

# Ctx Mgr that tells us what is going on
class ExampleContextManager:
	def __init__(self, val):
		print('init')
		self._val = val

	def __enter__(self):
		print('entering')
		return self # this is what goes into the "x" in "with CTX as x"
		# you don't always return self, like in contextlib.redirect_stdout

	def __exit__(self, exc_type, exc_value, exc_traceback):
		if exc_type is None:
			reason = 'normal'
		else:
			reason = f'error {exc_type.__name__} was raised'

		print(f'exited with status: {reason}')

		return True # by returning true, we make all output from inside the context manager go away nicely. "return True" just says "exit without panic", even though it might not exit in a *normal* way.
```

- That's it for context management.
