---
title: Topic 13 - Inheritance
date: 2024-03-11
---

> Full notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/Inheritance/)

Contents
- [[#Defining Inheritance]]
- [[#Attributes and Classes]]
- [[#Exemplifying Single Inheritance]]
- [[#Multiple Inheritance]]
- [[#Custom Exceptions and Inheritance]]

#### Defining *Inheritance*

- If lots of objects solve a similar problem, and if they all agree on solving that problem in the same *way*, then we can simplify that methodology through ***inheritance***.
	- We want two classes with methods with the same signature and *body*, but the ability to do different *specific things*

- If we say that a class _inherits_ from another, then we're saying that objects of the _derived class_ can do everything that objects of the _base class_ can do, also in the same way. 
	- The base class _is-a_ derived class, but the derived class _is-not-a_ base class. 


#### Attributes and Classes
- Remember that Python follows the LEGB rule. (i.e. Local, Enclosing, Global, Built-In)
- If we don't have an `__eq__` method built into a class, we get the `object` defined `__eq__` class

```python
class Thing:
	pass

# all python objects inherit from "object"

>>> Thing.__bases__
... (<class 'object'>)

>>> object.__eq__ is Thing.__eq__
... True
```

- What we just displayed was *single inheritance*, where a class has *one* base class. 


#### Exemplifying Single Inheritance
- Let's write up an example

```python
class Base: 
	def first(self): 
		return 13

class Derived(Base): 
	def second(self):
		return 17

>>> Derived.__bases__
... (<class '__main__.Base'>,)

>>> d = Derived()
>>> d.second()
... 17
>>> d.first()
... 13

>>> b = Base()
>>> b.first()
... 13
>>> b.second()
... # Failed with an error


# a better convention than "type(item) ==" is to use "isinstance" when we are 
# considering base and sub classes: 
>>> type(d) == Base
... False 
>>> isinstance(d, Derived)
... True
>>> isinstance(d, Base)
... True
>>> isinstance(b, Derived)
... False 
```

- So here, the `Derived` class *singly inherits* from `Base`, which *singly inherits* from `object`. Remember that inheritance goes *upward*, not downward. 

- When a class "Y" singly inherits from a class "X", what can it do?
	- Derived classes can add new functionality not present in their base classes.
	- Derived classes can override methods or attributes from their base classes.
	- Derived classes cannot remove attributes from their base classes.
	- We can replace attributes in a derived class, "bypassing" the LSP (next) either by providing a new definition or by incorporating the functionality of the base class using the super() function. 

> The Liskov Substitution Principle
>      - If Y inherits from X, Y objects must be able to *safely* take the place of X objects. 

- How can we use `super()` in python to bypass or "override" ("overload" is when you change the params)?

```python
class Base:
	def value(self): 
		return 11

class Derived(Base):
	def value(self): 
		base_val = super().value() 
		return base_val ** 2

>>> Derived().value()
... 121 
```

- Why "super"?
	- In the usual obj-oriented design lexicon: 
		- When Y inherits from X: 
			- X is a "superclass" of Y
			- Y is a "subclass" of X
- Overall, inheritance *enriches* a subclass 

- That's a general overview of how to handle basic inheritance of classes in ~~Java~~ Python!


#### Multiple Inheritance
- We say that *multiple inheritance* in Python is the act of a class deriving from **more than one Base**
	- Objects of this "multiple-subclass" can be substituted in place of objects of _either_ of those superclasses. 

```python
class Base1:
    def one_only(self):
        return 'Base1.one_only'


    def both(self):
        return 'Base1.both'



class Base2:
    def two_only(self):
        return 'Base2.two_only'


    def both(self):
        return 'Base2.both'



class Derived12(Base1, Base2):
    def derived_only(self):
        return 'Derived12.derived_only'



class Derived21(Base2, Base1):
    def derived_only(self):
        return 'Derived21.derived_only'


# lets look at some examples with this: 

>>> Derived12().derived_only()
    'Derived12.derived_only'
>>> Derived12().one_only()
    'Base1.one_only' 
>>> Derived12().two_only()
    'Base2.two_only'           
>>> Derived12().both()
    'Base1.both'               

# wait! why did we get Base1? 
# lets look into that: 
```

- **Method Resolution Order (MRO)**
	- When a class is defined in python, then an MRO is automatically determined for it 
	- The MRO of a class for single inheritance goes like `( [the class itself], [the superclass], [the superclasses' superclass], ..., [object], )`
		- As such, the *MRO* of `Derived12` is (`Base1`, `Base2`, `object`)
		- But when we look at *multiple* inheritance, it gets a bit more interesting

	- The MRO is a *linearization* ([Python MRO Linearization Algo](https://www.python.org/download/releases/2.3/mro/))
		- A linearization is a collection of related classes that adheres to two rules 
			- If a class X is a base class of class Y, Y must appear before X
			- If classes X and Y are listed as base classes of *the same class*, then X must appear before Y
		- As such, the MRO of the subclass would be `(X, Y, object)`
	- What about ***multiple multiple inheritance***???

```python
# lets consider whether we had something like this:

class MoreDerived(Derived12, Derived21):
	pass

# TypeError() was Raised!
# we can't even do this type of thing, because a good MRO cannot even be defined! 
```


#### Custom Exceptions and Inheritance

- We've been (for a while) defining our own exceptions in these projects. 
```python
class CustomException(Exception):
	pass
```

- Exceptions in python are *categorized* 
	- For example, for files: 
	- `UnicodeDecodeError` stems from `IOError`
		- i.e. if you catch a "parent" exception, you also catch the "child" exception!

- Exceptions are categorized using inheritance!

```python
class CustomValueError(ValueError):
	pass

try:
	raise CustomValueError
except ValueError:
	print("caught valueerror")
except CustomValueError:
	print("caught customvalueerror")

# the issue here is that excepts are matched *linearly*
# i.e. ValueError will be checked *before* CustomValueError!

# so if we do it like this, we'll always get a print of "caught valueerror"
# but if we do it like this: 

try: 
	raise CustomValueError
except CustomValueError:
	print("caught customvalueerror")

... "caught customvalueerror"

# this worked!
```

- Make sure to *pay attention* to order of exceptions!