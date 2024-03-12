---
title: Topic 16 - Decorators
date: 2024-03-11
---


> Full notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/Decorators/)

Contents:
- [[#Introduction]]
- [[#What are decorators?]]
- [[#Implementing Modifier Functionality]]
- [[#Multiple Decorations (interior design)]]
- [[#Argumentation in Decoration (angry interior design)]]
- [[#Characterizing how decorators (could) transform functions]]
- [[#Implementing Decorators with Classes (design school)]]
- [[#Decorators within Classes]]
	- [[#Decorating classes in their entirety]]
	- [[#Decorating Methods in Classes]]
- [[#Using Attribute Descriptors to Decorate Methods]] **(Super Important)**
- [[#Standard Library Decorators]]

#### Introduction
- We found a lot of value in the idea that we might like to combine functions in an automated way, and that higher-order functions presented a nice way to do that, so that we could call them, ask them to build new functions for us, and pass them as arguments to still other higher-order functions.
- But what do we do when we want a function in a module to be a combination of existing functions?
	- What about the methods of a class? 
	- What if we have ten functions in a module that all need to share a common characteristic? 
		- For these kinds of purposes, our best bet is another Python feature called a **_decorator_**, which, like a lot of Python features, is a simple idea with a rich and complex set of uses

#### What are decorators?
- A _decorator_, in its simplest form, is a function that accepts a callable argument and returns a function as its result.
	- Decorators are *not* special! They are just basic functions that are typed in a syntactically different way!

```python
>>> def unchanged(func):
...     return func
...
```

- Being a function, this decorator can be called like one.

```python
>>> def square(n):
...     return n * n
...
>>> another_square = unchanged(square)
>>> another_square(3)
    9
```

- But the name "decorator" implies that they're somehow used to _decorate_ a function, which gives us an idea that there's more to this feature than we've seen so far. How do we decorate functions? What happens when we do?
	- We decorate a function by adding a _decorator expression_ above it, in which we specify the name of the decorator preceded by the `@` symbol.

```python
>>> @unchanged
... def square(n):
...     return n * n
...
>>> square(3)
    9
```

- When a decorator expression appears above a function, it's as though we wrote something a little more long-winded.

```python
>>> def square(n):
...     return n * n
...
>>> square = unchanged(square)
>>> square(3)
    9
```

- In other words, when we decorate a function, we've asked Python to transform it just after defining it. In this case, the transformation was unimpressive, because our decorator returned the function as-is; that's why calling our decorated `square` function did the same thing an undecorated version of `square` would have done.


#### Implementing Modifier Functionality 
- Let's transform a function to give it a custom greeting!
```python 
def with_greeting(func):
	def greet_and_execute(n):
		print("Hello")
		return func(n)
	return greet_and_execute

@with_greeting
def square(n):
	return n ** 2

>>> square(4)
... "Hello"
... 16 

# what about something more interesting? 

@with_greeting
def mul(m, n):
	return m * n

>>> mul(5, 6)
... TypeError --> greet_and_execute takes 1 positional arg but 2 were given

# we only want *one* output from the definition of `greet_and_execute`, since 
# we specified only the argument `n`
```
- So, how can we customize decorators to get a variety of arguments?
	- `*args` and `**kwargs`, the *return*
- We want to make our decorator functions more robust, so in the case of `greet and execute`, it has to do two major things
	- The `greet_and_execute` function needs a signature that accepts any combination of positional and/or keyword arguments.
	- When `greet_and_execute` calls `func`, it needs to pass along whatever arguments it was passed, regardless of what kind of arguments they are.

```python
# let's write it:

def with_greeting(func):
	def greet_and_execute(*args, **kwargs):
		print("Hello")
		return func(*args, **kwargs)
	return greet_and_execute

@with_greeting
def distance_from_origin(x, y, z = 0):
	return math.hypot(x, y, z)

>>> distance_from_origin(11, 18)
    "Hello!"
    21.095023109728988        # That's more like it!
>>> distance_from_origin(x = 1, y = 17, z = 6)
    "Hello!"
    18.05547008526779         # Keyword arguments work, too!
```

#### Multiple Decorations (*interior design*)
- What if we want multiple decorations on a certain function?
```python
>>> from datetime import datetime
>>> def with_time_display(func):
...     def show_time_and_execute(*args, **kwargs):
...         print(f'Current Time: {datetime.now():%I:%M:%S %p}')
...         return func(*args, **kwargs)
...     return show_time_and_execute
...
>>> @with_greeting
... @with_time_display
... def square(n):
...     return n * n
...                 # We can decorate a function with more than one decorator.
>>> square(3)
    Hello!
    Current Time: 11:07:18 PM
    9
```
- When we run in this manner, Python automatically defines them in the *top-to-bottom* order from the call stack. 
	- What this means, essentially, is that first `@with_greeting` is executed, returning `"Hello!"`
	- Then, `@with_time_display` gets executed
	- Finally, the result of `square` is returned!


#### Argumentation in Decoration (*angry interior design*)
- What if we want to pass through specific parameters to the function being decorated?
	- This way, we can even change the functionality of the decorators themselves, without having to redefine them!

- Let's start with this initial function: 
```python
# we define this function that retries input 4 times, but we can simplify and *generalize* it 
def read_int():
    for _ in range(4):
        try:
            return int(input('Enter an integer: '))
        except Exception:
            pass

    return int(input('Enter an integer: '))
```

- Let's now decorate it with a parametrized decorator:
```python
def retry_on_failure(times): # define the wrapper
    def decorate(func): # define the *decorator* itself
        def run(*args, **kwargs): # define the run function that modifies the func 
            for _ in range(times - 1): # check errors "times" times 
                try:
                    return func(*args, **kwargs)
                except Exception:
                    pass

            return func(*args, **kwargs) # return func output

        return run # return run func

    return decorate # return decorator
```

- Now let's try it:

```python 
@retry_on_failure(3)
def read_int():
    return int(input('Enter an integer: '))

>>> read_int()
	Enter an integer: "hello"
	Enter an integer: "goodbye"
	Enter an integer: "see ya"
	TypeError --> Raised Type Error, we retried 3 times!
```


#### Characterizing how decorators (could) transform functions 
- Decorators can transform functions into new ones, but those transformations aren't entirely arbitrary; they automate certain patterns beautifully, while being entirely unable to automate others. So, we should understand where those limits are.

- We have *4 possible options* to make modifications with a Decorator:
	1. Adding code that runs <u>before</u> the original function's body.
	2. Adding code that runs <u>after</u> the original function's body.
	3. Surround the function with a *before* and an *after* addition, so we get modifications to the arguments and return value <u>around</u> the original function's body. 
	4. Replace the return value and the arguments of the original function's body entirely, so we get something <u>entirely different</u> from the original function!
		- This would not realistically be used often in practice

- Note that we **cannot** edit the original function's body *ever* through a decorator.
- Another thing to note is that decorators are defined at *definition* time, not runtime. Once that *def* statement is loaded, the decorator is already evaluated, so we cannot make custom modifications at runtime. 
	- Let's see an example of this:
```python
def bad_idea(func):
	return 'Boo'

@bad_idea
def square(n):
	return n * n

>>> square(5)
... "TypeError, since square is no longer callable (we defined it at compile time to be a string itself)"

>>> square
... "Boo" # damn. 

>>> type(square)
	<class 'str'> # yikes.
```


#### Implementing Decorators with Classes (*design school*)
- We saw in [[Topic 15 - Functional Programming#Implementing Custom Calling Mechanics]], we can call classes in customized way through a `__call__` method, which implements callability for classes that aren't functions. 

- Let's re-implement `bad_idea` through a *class*, with *expected* `callability`!
	- An `__init__` method taking a function as a parameter (in addition to `self`), which would make `InterestingIdea(func)` legal for any function. Since `InterestingIdea` is a class, `InterestingIdea(func)` is an attempt to construct an `InterestingIdea` object, so the result would be an `InterestingIdea` object.
	- A `__call__` method, so that `InterestingIdea` objects are callable, meaning that we can call them as though they're functions.

- If we did that, what would we expect to happen if we used `InterestingIdea` as a decorator?
```python 
>>> class InterestingIdea:
...     def __init__(self, func):
...         pass
...     def __call__(self, *args, **kwargs):
...         return 'Boo!'
...
>>> @InterestingIdea
... def square(n):
...     return n * n
...
>>> type(square)
    <class '__main__.InterestingIdea'>
>>> square(3)
    'Boo!'
```

- Nice! Everything worked as expected, but *why* would we want to do this? 
	- We can give multiple attributes to a *function* in itself, so it gains access to a *variety* of utilities
	- Let's exemplify this with `WithCallsCounted`

```python 
class WithCallsCounted:
    def __init__(self, func):
        self._func = func
        self._count = 0

    def __call__(self, *args, **kwargs):
        self._count += 1
        return self._func(*args, **kwargs)

    def count(self):
        return self._count

# Despite the fact that we wrote it as a class, `WithCallsCounted` is also a decorator, because it conforms to the same requirements that decorators must conform to: `WithCallsCounted` (the class) can be called like a function and pass it the original function, and the resulting object acts like a transformed version of the original function.

# now let's use it!

>>> @WithCallsCounted
... def square(n):
...     return n * n
...
>>> square(3)
    9          # The decorated square function acts like the original one when it's called.
>>> square.count()
    1          # But square also knows how many times it's been called.
>>> square(5)
    25         # Does it really?
>>> square.count()
    2          # Yes, it does!
>>> type(square)
    <class 'call_counting.WithCallsCounted'>
               # And this is why we want to do this! 

```

- So now, by definition, a Decorator is something that is *given* something `callable`, and gives you back something `callable`!
	- In this specific example, we want the `WithCallsCounted` to retain the original mechanics of the function
- The sky's the limit, when it comes to the amount of complexity one could write in a decorator implemented as a class. And while we generally want to build simple solutions to simple problems, more complex problems require more complex solutions, so it's best for us to have a way to organize that complexity when the need arises.


#### Decorators within Classes
- Previously, we saw that we can write classes that act as decorators, with their objects taking the place of the functions they decorate. While we're on the subject of classes though, let's try decorating functions *within* classes, and the entire class itself!

##### Decorating classes in their entirety
- We can expect that the entire class is passed to the decorator as an argument, then replacing the class with the decorator function's result:
```python 
def unchanged(cls):
	print(f'decorating {cls.__name__}')
	return cls 

@unchanged
class Thing:
	pass
>>> 'decorating Thing' # instant! since it's evaluated on class define!

# if we swap it with something without calling mechanics or return
def hello(cls):
	return 'hello'

@hello
class Another:
	pass

>>> Another 
	'Boo' # great, Another is now a string, not a class!
```

- When we decorate a class, we aim to modify it in some manner
-  How do we modify a class *after* it's been built?
- Let's take a look:

```python 
class Person: 
	def __init__(self, name, age):
		self._name = name 
		self._age = age 

def name(self):
	return self._name 

>>> Person.__init__ # we can read class attributes!
	<function Person.__init__ at MEM_ADDR>

# ok, let's put a function into the class attribute
# make sure that it is shaped reasonably:

>>> Person.name = name # woah...
>>> p = Person('Boo', 13)
>>> p.name()
	'Boo' # that worked!

def age(self):
	return self._age 

>>> Person.age = age
>>> p.age()
	13 # it even came back properly, though p was initialzed before 
	   # the function was added!
```
- What does this mean?
	- We can edit the class's attributes after its been initialized. 
- Now we're fully equipped to write a class decorator. **It adds, updates, or removes attributes from a decorated class**, which we can use to automate formulaic code to write in a class!
	- Let's go back to Person 

```python 
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age

    def name(self):
        return self._name

    def age(self):
        return self._age
```
- What if we wanted to add a third value, such as `height` to the `Person` class, the change would follow a formula:
	- Add `height` to `__init__`
	- In the body of `__init__`, store the `height` into `self._height`
	- Add a `height()` method that returns `self._height` 
- This is so common in classes, to set a variety of fields that also have a variety of getters!

```python 
# let's write something that lets us do this:

@fields(('name', 'age', 'height'))
class Person: 
	pass

# if we have a @fields decorator, we can add a new attribute *whenever* 
# we need it! 

# so let's write the *fields* decorator!
from typing import Any 

def fields(field_names: tuple[Any]):
	
	field_names = list(field_names) 

	# we need to put "getter" methods into the class to read values
	def make_field_getter(field_name):
		def get_field(self): 
			# getattr(classname, item) ~> classname.item 
			# so we can create the "getters" 
			return getattr(self, f'_{field_name}')
			
		return get_field 

	def decorate(cls): 
		for field_name in field_names: 
			# setattr is a Python built-in that lets us 
			# set an attribute without knowing the name 
			# until the program runs!
			setattr(cls, field_name, make_field_getter(field_name))
			# we now set the getter functions to the class 
			# with their definition!

		# now we have to define the __init__ method of the class with
		# the possible attributes
		def __init__(self, *args):
			# zip is a utility that makes a list of tuples out of the 
			# two lists in order
			for field_name, arg in zip(field_names, args):
				setattr(cls, f'_{field_name}', arg)
			# now we set all those variables

		cls.__init__ = __init__ # now we set the init method!
		return cls # now we return the class as expected

	return decorate # and finally, we decorate


# LETS TRY IT!

@fields(('name', 'age'))
class Person:
	pass

>>> p = Person('Boo', 13)
>>> p.name
	<function fields.<make_field_getter>.get_field at MEM_ADDR>
	# it defined it as the `fields` attribute!
>>> p.name()
	'Boo' # nice!
>>> p.age()
	13 # NICE!
```

##### Decorating Methods in Classes
- Let's get a brief introduction:
```python 
class Thing:
	def value(self):
		return 13

t = Thing() 
# we can call methods the long way:
>>> Thing.value(t) 
	13 # this is the proper way to do it, but as seen below, 
	   # python does it for you too!
>>> t.value()
	13

# let's try this with @WithCallsCounted from abv and see what happens:
class Thing:
	@WithCallsCounted
	def value(self):
		return 13 

>>> t = Thing()
>>> t.value()
#	Traceback (most recent call last):
#     ...
#     File "C:/Examples/call_counting.py", line 37, in __call__
#      return self._func(*args, **kwargs)
#    TypeError: Thing.value() missing 1 required positional argument:
#               'self'
```
- We lost the `self` argument even though we called `value` on `t`.
- The specifics of what went wrong revolve around how functions follow rules automatically
	- However, we lost the functional abilities of `value`, because `WithCallsCounted` is a **class**. 
	- So let's look at how to solve this issue


#### Using Attribute Descriptors to Decorate Methods 
- We've looked previously at the rules around attribute access in Python, which spell out what happens when we ask an object for one of its attributes. The following example summarizes those rules succinctly.
```python
>>> class Something:
...     c = 0  # Classes can have attributes.
...
>>> s = Something()
>>> s.b = 13   # Objects can also have attributes.
>>> s.b
    13  # We can access the b attribute we just assigned a value into.
>>> s.__dict__['b']
    13   # The value of the b attribute is stored in the dictionary within s.
>>> s.c
    0   # We can access the c attribute, since it was initialized in the class.
>>> 'c' in s.__dict__
    False # But we won't find c in the object's dictionary, since it
          # belongs to the class rather than each object separately.
>>> Something.__dict__['c']
    0     # On the other hand, we will find c in the class' dictionary.
```

- These rules can also be customized when the need arises, *because of course they can* 
	- When we call methods, they need an extra parameter whose name is normally `self`, but what gives the methods of a class their special ability to be called in a way that doesn't require us to pass `self`, but to instead have another part of the expression become a `self` instead, which is built-in to functions. How?

- An <u><b><i>ATTRIBUTE DESCRIPTOR</b></i></u>, lovingly called a *descriptor* is an object that can <u>customize its own value</u> when obtained from an attribute within a class. 
	- The presence of the `__get__` method is what makes an object a descriptor: 
	- `__get__(self, obj, objtype)`, which is given the object from which the attribute is being obtained, as well as the objects class (`objtype`). Whatever `__get__` returns will be the attribute we get back when accessing the attribute

- Lets take a look at an example: 
```python
>>> class AlwaysBoo:
...     def __get__(self, obj, objtype):
			# if we call __get__ from HasBoo, then 
			# `obj` is the *instance* name of HasBoo
			# and `objtype` is "HasBoo"
			print(obj, objtype)
...         return 'Boo!'
...
>>> class HasBoo:
...     boo = AlwaysBoo()          
	# It looks like boo is an AlwaysBoo object.
	
>>> x = HasBoo()
>>> x.boo
	<__main__.HasBoo object at MEM_ADDR>, <class '__main__.HasBoo'>
    'Boo!'                         
    # But when we ask for its value, we get a string, and a print!
    # This is because AlwaysBoo is a *descriptor* 
    
>>> type(x.boo)
    <class 'str'>                  
    # And when we ask for its type, we see it's a string, too.
    
>>> type(HasBoo.__dict__['boo'])
    <class '__main__.AlwaysBoo'>   
    # But in the class' dictionary, it's an AlwaysBoo object, after all.
```

- This seems like a crazy trick, but it's exactly how functions become *bound methods*, which are a bit like partial functions we saw previously. A bound method is one in which the `self` parameter's value has already been determined, so it can be called by passing all remaining args to it.
	- For example:

```python 
>>> class Whatever:
...     def do_something(self):
...         return 13
...
>>> w = Whatever()
>>> w
    <__main__.Whatever object at 0x0000028862F46F80>
>>> Whatever.do_something
    <function Whatever.do_something at 0x000002886305D7E0>
    # When we ask the class for its do_something attribute, we get back
    # a function.  To call it, we'll need to pass all of the necessary
    # arguments, including self.
    
>>> w.do_something
    <bound method Whatever.do_something of <__main__.Whatever object at
		0x0000028862F46F80>>
    # Notice that the address of the Whatever object listed here is
    # the same as the address of w.  That's because w is the object
    # to which the method has been bound.  w will be its self.
```

- Ultimately, this is exactly what is missing from `WithCallsCounted`, so that way we can also add it to classes, because then we can let the function transform into a bound method! 
```python
class WithCallsCounted:
    def __init__(self, func):
        self._func = func
        self._count = 0

    def _call_original(self, func, *args, **kwargs):
        self._count += 1
        return func(*args, **kwargs)

    def __call__(self, *args, **kwargs): # as normal, for `def` direct
        return self._call_original(self._func, *args, **kwargs)

    def __get__(self, obj, objtype):
        def execute(*args, **kwargs):
            original_func = self._func.__get__(obj, objtype) # BIND THE METHOD!
            return self._call_original(original_func, *args, **kwargs)

        execute.count = self.count # give it a count() attribute as well 
        return execute # return the final function so it can be executed! 

    def count(self):
        return self._count
```

- Now that we've done this, we can test it with `self` binded: 
```python
>>> @WithCallsCounted
... def square(n):
...     return n * n
...
>>> class Thing:
...     @WithCallsCounted
...     def value(self):
...         return 13
...
>>> square(3)
    9            
    # We can call a decorated function.
>>> square.count()
    1            
    # Decorated functions have counts.
>>> t1 = Thing()
>>> t2 = Thing()
>>> t1.value()
    13           
    # We can call the value method with self bound already.
>>> t1.value.count()
    1
>>> Thing.value(t1)
    13           
    # We can call the value method with self unbound.
>>> t2.value.count()
    2            
    # Methods have counts, which are independent of which object they're called on.
>>> Thing.value.count()
    2            
    # Method counts can be obtained either through classes or their objects.
```
- What's cool as well is that we can chain `__get__` calls together, to get a variety of *daisy-chained* objects so to speak. 
- Note that the `__get__` descriptor implements a custom access mechanic that can only be allowed at the *class* level, **not** at the *instance* level.
	- At an instance level, we can invoke it by doing `instance.Class.__get__(instance, typeof instance)`, but `__get__` is strictly automatically invoked at the *Class* level. 


#### Standard Library Decorators 
- Not necessary, but a couple useful ones are available on the [notes](https://ics.uci.edu/~thornton/ics33/Notes/Decorators/), such as `functools.cache`, `functools.lru_cache` (least recently used) (both for memoization), and `functools.total_ordering`, which automates a pattern of all orderings of attributes as long as an `__eq__` and `__lt__` are defined. 

- fin. 
