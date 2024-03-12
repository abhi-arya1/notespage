---
title: Topic 2 and 3 - Classes, Objects, Functions, Parameters
date: 2024-03-08
---

Detailed Topic 2 Notes found at [Classes and Objects](https://ics.uci.edu/~thornton/ics33/Notes/ClassesAndObjects/)
Detailed Topic 3 Notes found at [Functions and Parameters](https://ics.uci.edu/~thornton/ics33/Notes/Parameters/)

# Topic 2

#### Attributes of Objects and Classes

- Classes allow us to create blueprints and interfaces for custom objects for our purposes.
- How does this tie into [[Topic 1 - Modules and Namespaces]]?
  - Modules in Python contain `dict` types that contain the _attributes_ and _properties_ of themselves. If we try to access a class element that isn't inside of the attributes dictionary, then an `AttributeError` is raised.
    - Theres no "Not Found" error, but its an Attribute Error! Python at its core looks for _attributes_
  - To access those attributes, we can instantiate a class and then call `__dict__` on that instance!

```python
class Thing:
	pass

t = Thing()
t.__dict__
>>> {} # since the "Thing" class is currently empty (or has no attr's)


# what if we add attributes to "Thing"
t.name = 'Boo'
t.age = 13
t.__dict__
>>> {'name': 'Boo', 'age': 13}
t.__dict__['age'] = 18
t.age
>>> 18 # it updated!

# we can employ the Mechanics from Topic 1 to control the mechanics
# of an Object as well!
```

- How about reworking this with a class with functions?

```python

class Squarer:
	def square(self, n):
		return n*n

s = Squarer()
s.square(3)
>>> 9 # we would *assume* that methods are stored in class attr's too

# but...
s.__dict__
>>> {} # it doesnt!?

# so where is the "square" function?
# lets take a trip back:

# from 32A: there are two ways to call methods on Objects in Python
>>> s.square(3)
# OR
>>> Squarer.square(s, 3) # we need to pass in the "self" param

# these two syntaxes are identical, and the first one gets
# turned into the second by CPython compilers

# so the fact of the matter is that classes in Python
# store methods in their *own* dictionary
Squarer.__dict__
>>> mappingproxy({..., 'square': <function Squarer.square at MEM_ADDR>, ...})
# note that mappingproxy helps the interpreter assure that the keys for class-level attributes and methods can only be strings.

```

- **Recap:** Instances store _variables_, Class Names store _functions_ in their respective `__dict__` attributes.

- Now there's a variety of ways to go about where object attributes are:

```python
class Person:
	MAX_NAME_LEN = 30
	def __init__(self, name, age):
		self.name = name[:self.MAX_NAME_LEN]
		self.age = age
	def describe(self):
		return f'{self.name}, ange {self.age}'

# where can we ask it for certain things?
p1.MAX_NAME_LEN
>>> 30 # interesting...
Person.name
>>> AttributeError()
```

- **If we ask an object for an attribute...**
  1.  First, Python checks the _instance_ of the object to see if the attribute exists. If so, that's what we get
  2.  Otherwise, Python checks the class to see if the attribute exists. If so, that's what we get
  3.  Otherwise, an `AttributeError` is raised
- **ICA Rule --> Instance, Class, AttributeError** for Class scoping in Python

- What if the _Class_ has a variable with the same name as the _Object_ that gets instantiated?
  - The _object_ variable will be returned, which follows the order above.

#### Static and Class Methods

- Let's define a class where it's job doesn't matter, but we want to know how many there are at a given time with a `@staticmethod`. Here is an example of how it works:

```python
class Widget:
	# time for the class attribute to track the count
	_count = 0

	def __init__(self, id):
		self._id = id
		Widget._count += 1

	def id(self):
		return self._id

	# time for the StaticMethod that returns counts
	# we add a @staticmethod "Decorator!"
	@staticmethod
	def count():
		return Widget._count

w1 = Widget(13)
w2 = Widget(18)

w1.id(), w2.id()
>>> 13, 18 # different

w1.count()
>>> 2 # same
w2.count()
>>> 2 # same
Widget.count()
>>> 2 # SAME!


Widget.id()
>>> AttributeError()
```

- Now lets define a `@classmethod`.
  - "When this method gets called, do something special to it"
    - Instead of sticking the arguments into self, shift the arguments over by 1 and make the first parameter the _entire class_
  - The class becomes a parameter to that method, similar to how normal methods transform the target object into a `self` parameter, which is useful if we'll need to use the class for something.
  - One example where class methods are useful is for creating what are sometimes called *factory methods*, which are methods whose job is to create objects of a type, but that have names that make clearer the way that job is being done, which can be especially useful if there's more than one of them in the same class, though can still make for a more readable syntax even if there's only one of them.
- One such example is in `Point`, below:

```python
import math

class Point:
	@classmethod
	def from_cartesian(cls, x, y): # cant use "class", its "cls"
		return cls(x, y)

	@classmethod
	def from_polar(cls, r, theta):
		return cls(r * math.cos(theta), r * math.sin(theta))

	def __init__(self, x, y):
		self._x = x
		self._y = y

	def x(self):
		return self._x

	def y(self):
		return self._y

# how can we work with this?
p1 = Point.from_cartesian(3, 5)
p1.x(), p1.y()
>>> 3, 5

p2 = Point.from_polar(1, math.radians(30))
p2.x(), p2.y()
>>> 0.866..., 0.5
```

# Topic 3

- The way we've learned to write functions previously is functions where:
  - You have to nail down parameters, names, etc.
- How can we make functions much more _dynamic_?
  - Python stdlib already does this: `len` can handle lists, strings, ranges, etc.
  - Same with `max`: All "iterables" are handled!
  - Even `print`:

```python
print('test', 'test', sep='!', end=' woah')
>>> 'test!test woah'
```

- What is the strategy for writing a function that can handle a variety of data types, customizable arguments, and _know_ what to do with them?

- First lets take a look at what we can do with functions:

```python
def subtract(n, m):
	return n - m

subtract(18, 7)
>>> 11
# lets add keyword args
subtract(18, m=7)
>>> 11
# we can flip ordering
subtract(18, n=7)
>>> -11
# or even both
subtract(n=18, m=7)
>>> 11

# but there are rules:
subtract(m=18, 7)
>>> SyntaxError() # positional args cant follow keyword args
```

- Lets understand how Python goes about this, but first, definitions:
  - **`Parameters` are the places/variables you pass `Arguments` to**
    - They are different words. Parameters are in the function context and Arguments are in the global context. _Scoping strikes again_
- There are two types of arguments

  - Positional Arguments: Interpreted in the order that you pass them through (`foo(5)`)
  - Keyword Arguments: Matched by specified name (`foo(a=5)`)

- `*` or `**`, the _unpacking_ operator(s)
  - Similar to how we can do `values = (1, 2)` and `a, b = values` to get `a=1, b=2`, we can unpack directly by using the `*` operator.

```python
# using "subtract(n, m)" from above
things = [18, 7]
subtract(*things)
>>> 11
# but in this case, you need it to be *exactly* the number of vars
# len(things) != 2 --> TypeError since args aren't matched

# you can also do interesting things
few = [11]
subtract(*few, *few)
>>> 0 # hmm...
```

- For _dictionaries_, the `*` unpacks `dict.keys()` by default--even with loops this is true. To get the full dictionary and be able to access the _values_, we have to use `**` operators.

##### Tuple-Packing Parameters

- This is how we define an indeterminate number of positional arguments in Python to access them in a function!
- If you put unpacking parameters while passing in argument in, then you say unpack before it goes in, but here, you say unpack once it is in!

```python
def maximum(first, *rest):
	if rest: # is there more than one arg?
		largest = first
	else:
		largest = None
		rest = first

	for value in rest:
		if largest is None or value > largest:
			largest = value

	return largest

maximum(1, 2, 3, 4, 5, 6)
# "first" will be 1, and "rest" will be (2, ..., 6)
>>> 6

maximum([1, 2, 3, 4, 5, 6])
# "first" is now the list, and "rest" is just ()
>>> 6 # we now *only* check "first"
```

- Note that a `*` argument can only appear once, because everything _else_ thats _positional_ that hasn't been accounted for needs to be put into the first `*` argument, so another one would never be defined!
- Var-Positional Arguments _cannot_ have Default Values.

- How can we specify where we want to _transition_ from Positional to Keyword arguments?

```python
# the * specifies that everything after MUST be a keyword argument.
def sub(n, m, *, minimum=None):
	diff = n - m
	if minimum:
		diff = max(diff, minimum)
	return diff

subtract(5, 6, minimum=4)
>>> 4

# we can also do this with a "/"
def sub(n, m, /, minimum=None):
	diff = n - m
	if minimum:
		diff = max(diff, minimum)
	return diff

subtract(5, 6, minimum=4)
>>> 4

# the only difference is now everything on the left of a "/" must be positional, and on the right it can be either positional or keyword.
```

##### Dictionary Packing Parameters

- We can have an unspecified number of _keyword_ arguments simply by doing `**` as mentioned above

```python
def print_all(**kwargs):
	for key, value in kwargs.items():
		print(key, value)

# this follows the same policies as others, but we can now specify by *name* rather than by *position*!!
```

##### Default parameters:

```python
# if we want a function to default to a parameter, some programming
# languages, like Java (for example), might resort to *overloading*

# in python, we can easily make a default parameter *within*
# a parameter area itself
def read_integer(prompt="Enter a number: "):
	return int(input(f'{prompt}'))

read_integer()
>>> "Enter a number: " 13 # returns 13

read_integer('Num Here: ')
>>> 'Num Here: ' 13 # returns 13

# we now have a default for when there are no params!
```

- Note that when a parameter _without a default_ follows a parameter _with a default_, then we get a SyntaxError

- Also, note that _default parameters_ are built on time of initialization. Editing a default list will edit that default built-in list!

```python
def add_to_end(val, x=[]):
	x.append(val)
	return x

add_to_end('there', ['hi'])
>>> ['hi', 'there']
add_to_end('hey')
>>> ['hey']
add_to_end('there')
>>> ['hey', 'there']

# We edited the default *x=[]* list that was built ON DEFINING the function!
# Never do this, or it will end in weird situations like this!
```

- **NEVER** use Mutable Values as Default Arguments!!
