---
title: Topic 15 - Functional Programming
date: 2024-03-08
---

> Full notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/FunctionalProgramming/)

Contents

- [[#Introduction]]
- [[#Lambda Expressions]]
- [[#Higher Order Functions]]
- [[#Function Composition]]
- [[#Partially Called Functions]]
- [[#Built-In Higher Order Functions]]
- [[#Implementing Custom Calling Mechanics]]

#### Introduction

- Python is a language based on a strategy called "object oriented programming" (OOP)
  - Everything in the language is based around objects that store data, have data models, and their own functions.
- But _not every language is created equal_

  - In 45C, we'll see that C takes a _functional_ approach, while C++ mixes in some other things

- So what is a _functional_ approach (or a traditional approach really, since OOP is technically "new")
- In a _purely functional_ program, we have a few defining characteristics:

  1.  Programs are organized **predominantly** around _functions_.
  2.  Functions are _pure_, where their results are determined strictly by their arguments.
      - There are no _side effects_ or changes. You take arguments and return an answer.
  3.  Functions are _first class_, which means that they're values just like integers, strings, or anything else, and that they can be passed as arguments and returned as results. This means there will be functions whose role is to operate on other functions; we call these *higher-order functions*.

- What do we get out of sacrificing Objects in favor of Functions?

  1.  _Clarity_, because functions can be understood more easily without regard to the others. Without side effects, a function `f` only impacts the meaning of a function `g` if one calls the other or one is passed as an argument to the other, either directly or indirectly.
  2.  _Testability_, because functions can be tested in isolation without concern about the prior effects of others. (Because of its roots in mathematics, there is also the related benefit of *provability*, in the sense that we might be able to use mathematics to prove that our programs meet their requirements, instead of only being able to test them.)
  3.  _Parallelizability_, meaning that more than one part of a program can be executed simultaneously on hardware that allows it — as most computers allow nowadays, in one form or another. In the absence of side effects, we can be sure that executing two functions simultaneously is safe, unless one of them needs the result of the other.

- Just one thing to keep in mind is the fact that we are speaking outside of real world constraints here.
  - A pure form of functional programming turns out to be relatively impractical for entire programs, for the simple reason that parts of most programs need to have side effects. We need to be able to print output to a shell, write output to a file, create sockets and connect them to a server, interact with a database, or draw graphics within a window, but these are all examples of side effects.
  - Most languages that used to be purely functional or purely OOP let you work on the _other side_ so you can use that necessary functionality
- So why learn this?

  - We can borrow from functional programming strategies when they are useful to us, but we don't need it in any other case.

- Let's take a look at some strategies employed in functional programming then, so we know when to use this strategy.

#### Lambda Expressions

- A *lambda expression* is an expression whose result is what we sometimes call an *anonymous function*.
- When we include the word *expression* in the phrase *lambda expression*, we mean that lambdas are evaluated and return a result, just like other expressions like `i + j` or `list(range(10))` are. We also mean that lambdas must follow Python's syntactic rules regarding expressions, which limits what we can write in the body of the function we're building.
  - We cannot give them a `return` statement, nor can we define them as a proper function with the `:` after the `lambda x:`.
  - All we can do is stuff that can be one-lined
- The word *anonymous* doesn't mean that we can't know what the resulting function does; it just means that the function doesn't have a name.

- A lambda expression is a way to write a function whose body is a single expression and for which we don't need a name.
  - We have all combinations of `*args`, `**kwargs`, etc. and everything else we may potentially need, and we can use them for a variety of things, like shown below:
- For example:

```python
>>> lambda n: n * n
    <function <lambda> at 0x000001812D9FDCF0>
>>> square = lambda n: n * n
>>> type(square)
    <class 'function'>
>>> square(3)
    9
>>> make_reversed_list = lambda *values: list(reversed(values))
>>> make_reversed_list(1, 2, 3, 4, 5)
    [5, 4, 3, 2, 1]
>>> make_dict = lambda **kwargs: dict(kwargs)
>>> make_dict(a = 1, b = 2, c = 3)
    {'a': 1, 'b': 2, 'c': 3}
```

- But _why_ do we want these/_why_ do we need them?
  - Lambdas are good when used with **Higher Order Functions**, which are:

#### Higher Order Functions

- There's two sides to these:

  1.  Functions that accept other functions as arguments
  2.  Functions that return other functions as their result

- Let's take a look at (1)

```python
def square(n):
	return n * n

def transform_all(f, values):
	for value in values:
		yield f(value) # a generator!

# this just makes transform_all a very flexible function!
>>> list(transform_all(square, [1, 2, 3, 4, 5]))
	[1, 4, 9, 16, 25]
>>> list(transform_all(len, ["Boo"]))
		[3]
>>> list(transofmr_all(int, ['1']))
	[1]
... # the possibilites are endless as long as the type of function to transform
	# with is compatible with the items in the list
	# we can even do this:
>>> list(transform_all(lambda n: -n, [1, 2, 3]))
	[-1, -2, -3] # wow! cool!
```

- Let's take a look at (2)

```python
def make_function():
	def another_function():
		return "Hello There"
	return another_function

>>> make_function()()
	"Hello There" # we need to call the returned function with the extra ()!
>>> f1, f2 = make_function(), make_function()
>>> f1 is f2
	False
	# why would we do this then?
```

- If the inner function is being built freshly each time the outer function is called, then why couldn't the inner function be different each time? We'd have something a lot more powerful: a tool that can build many similar functions for us, so we don't have to write them ourselves.

  - A _function_ generator so to speak

- Let's make an example of that
  - Recall `any(args, kwargs)` and `all(args, kwargs)` such that it tells you whether _any_ or _all_ of an iterables value are truthy.
  - Missing from Python is a function called `none`, which is truthy when none of the iterable's values are truthy themselves.

```python
# we could implement "none" in a rather trivial manner, such as:
def none(iterable):
	return not any(iterable) # because none means "not any" trivially


# what if we want a boolean returning function that gives you the negation
# of another function? what if we want to generate many?
# let's do it!
def negate(func):
	def flip(n):
		return not func(n)
	return flip

# now...
none = negate(any)
>>> none([0])
    True

# we don't even have to name it!
>>> negate(any)([0])
	True # same result!

# we can do even better!
def negate(func):
	return lambda n: not func(n)

# now even simpler!
>>> negate(any)([0])
	True # same result!

# we made a *higher order function* because we made a *higher order* function,
# which makes not just one problem go away, but an entire class of problems
# go away!
```

#### Function Composition

- Function composition in Mathematics tells us that $(f \circ g)(n)$ or $f(g(n))$ is a function composition, where the output of one function becomes the input of another, which then gives a final output.
- We can _compose_ Python functions to chain them further too!

```python
# we can do a helper, or we can simplistically do it like this
def compose(f, g):
	return lambda n: f(g(n))

# now, for example
>>> compose(len, str)(123456789)
	9 # the length of "123456789" as a string!
>>> compose(len, str)([1, 2])
	6 # the length of '[1, 2]' as a string!
```

- What if we wanted further composition? We can even make a `bash`-esque pipeline!

```python
def pipeline(first, *rest):
	def execute(n):
		_n = first(n)
		for func in rest:
			_n = func(_n)
		return _n
	return execute

# now, let's try it
square = lambda x: x ** 2
>>> pipeline(str, len, square)(1234)
	16 # str(1234) = '1234', then len('1234') = 4, then 4 ** 2 = 16!

# since `pipeline` accepts parameter packing, we can make *infinite* pipelines!
```

#### Partially Called Functions

- We can define a function such as `multiply(m, n)` that has `return m * n`. As long as we give it two parameters, it'll multiply them. But _what if we don't_?
  - `multiply(3)` would raise a TypeError for a missing argument, but there's a way we can _suspend_ the function call till later.
  - Essentially, we'd do `m2 = multiply(3)` and then `m2(8)` would return 24, but **we can't actually do this**
    - We can, however, write a _higher order function_

```python
def pcall(func, n):
	def hold(m):
		return func(n, m)
	return hold

m2 = pcall(lambda m, n: m * n, 8)
>>> m2(3)
	24 # 8 * 3, but we suspended the multiplication of some "n" by 8 for later!
```

The `partially_call` function could be made more general than this, though, by allowing us to pass any number of positional arguments to it, then returning a partial function that also accepts any number of positional arguments.

```python
>>> def partially_call(f, *args):
...    def complete(*remaining_args):
...        return f(*args, *remaining_args)
...    return complete
...
>>> multiply_by_three = partially_call(multiply, 3)
>>> multiply_by_three(8)
    24
>>> print_boo_description = partially_call(print, 'Boo', 'is')
>>> print_boo_description('happy', 'today')
    "Boo is happy today"
>>> print_boo_description('always', 'totally', 'perfect')
    "Boo is always totally perfect"
```

Partial functions are nonetheless functions, so we can partially call them, as well, with all of the arguments from all of the partial calls being combined as we'd expect.

```python
>>> print_boo_happy_reason = partialy_call(
... 		print_boo_description, 'happy', 'because')
>>> print_boo_happy_reason('of', 'cool', 'weather')
    "Boo is happy today because of cool weather"
```

- There's a lot more ways to continue going deeper and deeper in this manner, with keyword arguments, etc. etc, but Python provides many of its own. As such, let's take a look at some Python built-in functions.

#### Built-In Higher Order Functions

- Python has a variety of built-in _utility_ functions for the purpose of generating things that _higher order_ functions may require.

```python
# let's take a look at a few:

# map ##################################################################

map # --> maps a function onto a list or a combination of lists
>>> list(map(lambda n: n * n, range(5)))
	[0, 1, 4, 9, 16]
>>> list(map(lambda a, b, c: a + b * c, [1, 3, 5], [2, 4, 8], [-1, 1, -1]))
	[-1, 7, -3] # [(1 + 2 * -1), (3 + 4 * 1), (5 + 8 * -1)]
>>> list(map(lambda a, b: a * b, [1, 2, 3], [4, 5]))
	[4, 10] # when the first one runs out, everything ends!

# filter ###############################################################

filter # --> given a function and an iterable, it gives all the items in the
	   #     iterable that are truthy based on the function
>>> list(filter(lambda n: n > 0, [1, 2, -3, -4, 5, -6]))
	[1, 2, 5]


# reduction #############################################################

# we can get this in Python through stdlib `functools`
import functools

reduce # --> combine and generate a *single answer*

>>> functools.reduce(lambda a, b: a + b, [1, 2, 3, 4])
    10  # Adding the elements together yields their sum. (1+2 + 3+4) = 10
>>> functools.reduce(lambda a, b: a + b, [[1, 2], [3, 4], [5, 6]])
    [1, 2, 3, 4, 5, 6]  # Adding lists together flattens them.
>>> functools.reduce(max, [3, 11, 7, 2, 1, 9])
    11  # If each combining operation returns the larger of the new value
        # and the largest so far, we'll have determined their maximum.


# two more things worth knowing for this:

# INITIAL VALUES
>>> functools.reduce(lambda a, b: a + b, [])
	# TypeError! No *initial value* was provided
>>> functools.reduce(lambda a, b: a + b, [], 5)
	5 # we get the initial value because there's nothing left to reduce!
>>> functools.reduce(lambda a, b: a + b, [1, 2, 3], 12)
    18                   # Yep!  12 + 1 + 2 + 3 = 18.

# THE `operation` STANDARD LIBRARY

# if we wanted to do a reduction without a lambda, we could do:
>>> functools.reduce(+, [1, 2, 3])
# we'd expect 6, but we'd rather get an error. What we can do instead is:

import operation
>>> functools.reduce(operation.add, [1, 2, 3])
	6 # nice!
>>> functools.reduce(operation.truth, [0, 1, 2])
	[1, 2] # because 0 is falsy by existence


# one *very* last thing here, the *partial* function is predefined!
>>> print_boo_description = functools.partial(print, 'Boo', 'is')
>>> print_boo_description('happy', 'today')
    Boo is happy today
>>> type(print_boo_description)
    <class 'functools.partial'> # not a function!
```

- Wait a second -- we got a partial, but it was not a function? How could we call it?

#### Implementing Custom Calling Mechanics

- Strangely, even though its not a function, we can call it if we store it into a variable!

  - We've seen that _dunder_ methods let us play with [[Topic 12 - Python's Data Model]]
    - As such, let's implement _call abilities_!

- We can first check whether something is callable with python's builtin `callable(item)`

```python
callable(lambda x: x + 1)
>>> True
```

- Then, we can implement the calling mechanics to our class to directly call it!

```python
class Square:
	def __call__(self, n):
		return n * n

# now...
>>> s = Square()
>>> s(3)
... 9
```

- The parameters of a `__call__` method can use all of the same features as any other function: keyword parameters, positional-only parameters, tuple- and dictionary-packing parameters, and so on. So, theoretically, any function could also be implemented by a class with a corresponding `__call__` method. There would be little reason to do so, since that's just a longer-winded way of saying something that could be said more simply with a `def` statement or `lambda` expression, but that's the mechanism at work. Calls to objects turn into calls to their `__call__` method.

```python-shell
>>> s.__call__(3)
    9           # We can call Square's __call__ method directly.
>>> square.__call__(3)
    9           # Functions also have a __call__ method, so we can call it directly, too.
```

- While we won't find ourselves directly calling the `__call__` method often, implementing a class with a `__call__` method is more common than you might imagine. When we write code that builds code, as we did when we wrote higher-order functions that return functions, we'll sometimes need all of the tools at our disposal, so it's best for us to understand the whole mechanism that makes functions work.
