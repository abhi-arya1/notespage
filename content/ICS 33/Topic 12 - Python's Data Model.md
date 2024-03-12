---
title: Topic 12 - Python's Data Model
date: 2024-03-11
---


>Full Notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/PythonDataModel/)

Contents
- [[#Introduction]]
- [[#The Protocols]]
	- [[#Lengths]]
	- [[#Truthiness]]
	- [[#Indexing, Slicing, etc.]]
		- [[#How does the presence of indexing impact other operations?]]
	- [[#Slicing]]
	- [[#Hashing]]
	- [[#Comparisons]]
		- [[#Relational Comparisons]]
	- [[#Arithmetic Operations]]
		- [[#Reflections (Comeback)]]
		- [[#Augmented Arithmetic]]


#### Introduction 
- Collectively, the mechanisms that govern how Python objects interact with one another is known as the *Python Data Model* #PDM
	- We've already seem some of those methods at work: 
		- `__repr__`: Shell Representation of Objects
		- `__init__`: Initialization of Objects
		- `__enter__, __exit__`: Context Management 
		- `__iter__, __next__`: Iterable Management 
	- These are all *protocols*, which rely on the built-in *dunder methods*, which assist with the Data Model. 

- We can learn how to use the #PDM for writing programs that are "Pythonic", which use the features built-in to Python to make our problem-solving easier (as seen in Project 3)
- Let's see what we can learn


### The Protocols
#### Lengths 
- We say that an object is *sized* in Python if it can be asked for a length. We use the `len` function to find that size. 
- The protocol we use to define this would be `__len__(self)`, returning an integer specifying the length of `self`
- But remember that there's a couple issues. 
	- First off, time complexity. We don't want to index through an entire object because that would be increasing to $O(n)$, $O(n^2$), or even more! 
	- We can use `sum`, `max`, or indexing by depth, etc. to find the length in the *optimal* way. 
```python
class MyRange:
    ...

    def __len__(self):
        return max(0, math.ceil((self._stop - self._start) / self._step))
```


#### Truthiness 
- We say that an object is *Truthy* if its `bool(obj)` returns True. 
- For built-ins:
	- `None` is falsy.
	- `True` is truthy, while `False` is falsy.
	- Numbers that are zero are falsy, while others are truthy.
	- Empty strings are falsy, while others are truthy.
	- Empty lists are falsy, while others are truthy.

- Length affects Truthiness --> Length of 0 is False, while any positive length is True.
- We can *override* Truthiness through the `__bool__` protocol, that lets us define the Truthiness of an object in our own way. 

```python 
>>> class Person:
...     def __init__(self, name):
...         self._name = name
...     def __bool__(self):
...         return self._name == 'Boo'
...
>>> p1 = Person('Boo')
>>> bool(p1)
    True          # Boo is truthy
>>> p2 = Person('Alex')
>>> bool(p2)
    False         # Everyone else is falsy
```


#### Indexing, Slicing, etc. 
- As indexing is supported on lists, ranges, strings, and other types, it's possible to also uniquely identify indexing for our own objects.
	- For example, dictionaries have their own indexing protocol through their *hashability* of their keys 
	- Many indexing operations also support deletion and insertion, but that varies from object to object. 
		- `d['B'] = 1` is valid Python, while `range(1, 100)[3] = 10` is not!
- How can we implement this? 
	- We can employ the `__getitem__, __setitem__` and `__delitem__` protocols!
```python
class MyRange:
    ...

    def __getitem__(self, index):
        if type(index) is not int:
            raise TypeError(f'MyRange index must be int, but was {type(index).__name__}')
        elif index < 0 or index >= len(self):
            raise IndexError('MyRange index was out of range')

        return self._start + index * self._step
```
- Again, the best case would be when we can implement this with $O(1)$ time *and* memory complexities. 

##### How does the presence of indexing impact other operations?
- When a class has a `__len__` and a `__getitem__`, it automatically defines its own `__iter__/__next__` protocols!
- Let's take a look at `ThreeSequence`, which has a fixed length of $3$, as well as a `__getitem__` method:
```python 
class ThreeSequence:
    def __len__(self):
         return 3
    def __getitem__(self, index):
        if 0 <= index < len(self):
           return index * 3
        else:
           raise IndexError

        
>>> tt = ThreeSequence()
>>> for i in tt:
>>>     print(i)
... 0
... 3
... 6

# since we have a `__getitem__` that defines how we 
# want to recieve items, and we have a length through 
# loop through, we can do this legally!

# we can also do this with `while`
>>> index = 0
>>> while index < len(s):
...     print(s[index])
...     index += 1
    0
    3
    6
```

- In fact, iteration works in the presence of a `__getitem__` method that accepts non-negative indexes, even in the absence of a `__len__` method, in which case successively larger indexes are passed to `__getitem__` until it raises an `IndexError`, at which point the iteration is considered to have ended! This way, we don't even need `len` defined in the getitem!

- However, if we have `__len__` defined, then we can reliably iterate in *reverse*, because without that, Python wouldn't know for certain the end of the sequence that we want to implement. 

- So, for indexing, there are some iteration methods that we can use to perform the best most efficient iteration for our purposes: 
	- The `__iter__(self)` method we saw before would provide custom iteration, by returning an iterator. (Note that if `__iter__(self)` is written as a generator function, then its result will be an iterator automatically.)
	- The `__reversed__(self)` method would provide reverse iteration. It returns a _reverse iterator_ (i.e., an iterator that produces the values in reverse order).
	- The `__contains__(self, value)` method would determine whether a value is part of the sequence, returning `True` if so or `False` otherwise.

- It's important to know how to implement each of these so we can make the asymptotic decisions to make our code more efficient!

#### Slicing 
- Slicing lets us split a *iterable* into a variety of different manners, such as `"hello"[1:]` giving us `"ello"` for those across the pond. 
- We already get a built-in implementation of slicing through `__getitem__`:
```python
>>> class Thing:
...     def __getitem__(self, index):
...         print(f'type(index) = {type(index)}')
...         print(f'index = {index}')
...         return None
...
>>> t = Thing()
>>> t[4]
    type(index) = <class 'int'>
    index = 4
>>> t[1:17:6]
    type(index) = <class 'slice'>
    index = slice(1, 17, 6)
>>> t[1:17]
    type(index) = <class 'slice'>
    index = slice(1, 17, None)
>>> t[:17]
    type(index) = <class 'slice'>
    index = slice(None, 17, None)
>>> t[::]
    type(index) = <class 'slice'>
    index = slice(None, None, None)
```
- However, there's a better way to actually define the `slice` object protocol inside of our `__getitem__` methods:
```python
class MyRange:
    ...

    def __getitem__(self, index):
        if type(index) is int:
            if 0 <= index < len(self):
                return self._start + index * self._step
            else:
                raise IndexError('MyRange index was out of range')
        elif type(index) is slice:
            start, stop, step = index.indices(len(self))

            start_value = self._start + start * self._step
            stop_value = min(self._start + stop * self._step, self._stop)
            step_value = step * self._step

            return MyRange(start_value, stop_value, step_value)
        else:
            raise TypeError(f'MyRange index must be int or slice, but was {type(index).__name__}')
```

It's also possible to assign to a slice of an object, as well as delete a slice. Implementing support for those operations requires similar modifications to `__setitem__` and `__delitem__`, whose `index` parameter will be a `slice` object in these situations.
- Slicing is *very* similar to indexing, just with a different `type` protocol! 


#### Hashing 
- Python draws a distinction between *hashable* and non-hashable objects. 
- **Hashable** objects have *two properties*:
	1. Hashable objects can be defined as a single integer value called a `hash`, which can be stored in a `hash table` that organizes objects based on their *intended* positions, and we can cheaply index by knowing exactly where they are given hashes. 
	2. Hashable objects are *immutable*, so that if they are hashed, then looking for them again won't cause future issues. 
- We can view an object's hash through Python's `hash()` method, which hashes similar objects similarly. 
- Hashable types include: `int`, `str`, `float`, `tuple`, and some more. If something isn't hashable, it raises a `TypeError`

- We can implement hashability through Python's `__hash__` method like this: 
```python
class MyRange:
    ...

    def __hash__(self):
        return hash((self._start, self._stop, self._step))
```

- But since hashing implies indexing through a hash table, we also need to know how to *compare* types in Python. 

#### Comparisons
##### Identity and Equivalence 
- Python has two different ways to determine *equality* 
	- *Identity* and *Equivalence* 
		- **Identity**: Each object has a unique identity, and that same identity (or memory address), and the `is` operator is used to check equality of `ids`
		- **Equivalence**: Equivalence is used to determine if two objects are directly equivalent, either by attributes, types, or by custom definition. 
	- `is` cannot be overriden, but `==` can with `__eq__`!
- We can override equality with the Data Model such as seen below: 
```python
class MyRange:
    ...

    def __eq__(self, other):
        if type(other) is MyRange:
            return self._start == other._start \
                   and self._stop == other._stop \
                   and self._step == other._step
        else:
            return NotImplemented
```
- Note that the data model *automatically* implements a `!=` check when you implement equality, because its pretty easy to tell when that doesn't return *True*. However, if you wanted to customize that, you could use `__ne__`. 

- But what is the difference between `__hash__` and `__eq__`?
	- If two objects are *equivalent*, they must have the same *hash*
	- If two objects have different *hashes*, they cannot be *equivalent* 
- As a safety mechanism, when we write an `__eq__` method in a class without a `__hash__` method being written in that same class, Python automatically sets the value of `__hash__` in the class dictionary to `None`, specifically to avoid the problem we otherwise would have created: Specifying a way for two objects to be inequivalent without having ensured that their hashes would be different.

##### Relational Comparisons
- Often, we want to compare the *relationship* between two objects, so we can determine a *natural ordering* 
	- These natural orderings are often determined *lexicographically*:
		- Compare the first element of each list. If one is less than the other, that determines how the lists compare.
		- If the first elements were equivalent, compare the second element of each list. If one is less than the other, that determines how the lists compare.
		- Continue until you either find an element that distinguishes one list from the other, or until one list runs out of elements. In that case, if both lists have run out of elements, the lists are equivalent; otherwise, the shorter list is less than the longer one.
- These can be overriden in the data model with 
	- `__lt__`: less than 
	- `__gt__`: greater than
	- `__le__`, `__ge__`: $\leq, \geq$ 
- Fun fact! Implementing `__lt__` and `__eq__` automatically defines all the rest!

- Equality can be implemented with `__eq__(self, other)`.
	- Note that `NotImplemented` in Python is not an error, but a constant with a type of `NotImplementedType`. We return it, we don't *raise* it!
	- This automatically defines `__ne__` (not equal), but you can override it with that method too. 
- Note that **equality and hashing are related**
	- If two objects are equivalent, they must have the same hash.
	- If two objects have different hashes, they must not be equivalent.
		- What these rules don't specify is that *inequivalent objects are allowed to have the same **hash***
```python
>>> class HashableThing:
...     def __hash__(self):
...         return 999
...
>>> hash(HashableThing())
    999
>>> h1 = HashableThing()
>>> h2 = HashableThing()
>>> h1 == h2, hash(h1) == hash(h2)
    (False, True)   
	# Inequivalent objects are allowed to have the same hash.
>>> h1 == h1, hash(h1) == hash(h1)
    (True, True)    
	# Equivalent objects must have the same hash.
```

- Essentially, nothing can go wrong here, as long as our `__hash__` method is written in a way that uses only what's stored within the object, since the default behavior of equality is to use identity (i.e., no two different objects are equal). What we're trying to avoid is equivalent objects having different hashes; that can't happen when we implement `__hash__`, as long as we implement it properly.

- On the other hand, what happens if we implement equality without hashing?
```python
>>> class UnhashableThing:
...     def __init__(self, value):
...         self.value = value
...     def __eq__(self, other):
...         return isinstance(other, UnhashableThing) and self.value == other.value
...
>>> UnhashableThing(13) == UnhashableThing(13)
    True     # Objects storing the same values are equivalent.
>>> UnhashableThing(11) == UnhashableThing(7)
    False    # Objects storing different values are inequivalent.
             # But without having implemented custom hashing behavior,
             # how can we be sure their hashes will be different?
>>> hash(UnhashableThing(11)) == hash(UnhashableThing(7))
    Traceback (most recent call last):
      ...
    TypeError: unhashable type: 'UnhashableThing'
             # Oh!  UnhashableThings aren't hashable at all!
>>> UnhashableThing.__hash__ is None
    True     # It's because UnhashableThing has no __hash__ method.
```

- As a safety mechanism, when we write an `__eq__` method in a class without a `__hash__` method being written in that same class, Python automatically sets the value of `__hash__` in the class dictionary to `None`, specifically to avoid the problem we otherwise would have created: Specifying a way for two objects to be inequivalent without having ensured that their hashes would be different.

- Of course, this mechanism isn't fool-proof, since nothing prevents us from writing `__eq__` and `__hash__` methods that don't fit together properly, but there is at least a sensible default at work here, which prevents us from accidentally introducing a problem we hadn't considered thoroughly.

##### Reflectivity 
- What if we have one object that implements `__eq__` but another one doesn't? If we check equality between these two, we'll actually get the result of the object that *does* implement the `__eq__` class. 
- This is because Python automatically incorporates *reflected* operators. If the class being invoked doesn't have that method, it checks with the other class in a reflected manner, instead of raising an error (that happens when the other class doesn't have it either though)
```python
>>> class B:
...     def __eq__(self, other):
...         print("B.__eq__")
...         return True
...
>>> class C:
...     pass
...
>>> c1 = C()
>>> c1 == b1
    B.__eq__
    True

# nice!
```
- This happens with almost all operations that require an `other`, and can be implemented in a custom manner by doing `__r[NAME]__`. This is more common with [[#Arithmetic Operations]]

#### Arithmetic Operations
- `__pos__(self)` implements the unary plus operator.
- `__neg__(self)` implements the unary minus operator.
- For example:
```python 
>>> i = 40
>>> +i # calls __pos__
	40
>>> -i # calls __neg__
	-40 
```

- For operations that require an `other` attribute, we can do: 
	- `__add__(self, other)` returns the sum of `self` and `other`.
	- `__sub__(self, other)` returns the difference when subtracting `other` from `self`.
	- `__mul__(self, other)` returns the product of `self` and `other`.
	- `__truediv__(self, other)` returns the quotient of `self` and `other`, without taking the floor of the result.
	- `__floordiv__(self, other)` returns the floor of the quotient of `self` and `other`.
	- `__pow__(self, other)` returns the result of raising `self` to the power `other`.

##### Reflections (Comeback)
- When we redefine arithmetic operators, the usual rule is that the operation `x ? y` turns into a call to the equivalent dunder method, with `x` being the first argument (i.e., passed into `self`) and `y` being the second (i.e., passed into `other`). So, for example, when evaluating `x + y`, Python attempts to call `x.__add__(y)`, the result of which becomes the result of the addition.

- However, when `x.__add__(y)` is not supported, Python makes one more attempt to add `x` and `y`, by instead calling into a _reflected_ version of the operator instead. In the case of addition, that reflected operation is implemented by the dunder method `__radd__`, so if `x.__add__(y)` is unsupported, Python attempts to call `y.__radd__(x)` instead. The mechanism it uses to determine whether `x.__add__(y)` is supported is simple: If the `__add__` method doesn't exist, or if it returns `NotImplemented`, it's unsupported.

- There's a reason why the reflected arithmetic operators need to be implemented separately: Not all of these operators are commutative. We would expect `x + y` and `y + x` to return the same result — in many cases, but certainly not all! — but we would not expect the same from `x - y` and `y - x`. So, if Python is going to reverse the order of the arguments, we'll also need to know that we need to perform the reverse of the usual operation, so a different dunder method is called.

	- `__radd__(self, other)` returns the sum of `other` and `self`.
	- `__rsub__(self, other)` returns the difference when subtracting `self` from `other`.
	- `__rmul__(self, other)` returns the product of `other` and `self`.
	- `__rtruediv__(self, other)` returns the quotient of `other` and `self`, without taking the floor of the result.
	- `__rfloordiv__(self, other)` returns the floor of the quotient of `other` and `self`.
	- `__rpow__(self, other)` returns the result of raising `other` to the power of `self`.


##### Augmented Arithmetic 
- Augmented arithmetic is defined when we do something like `+=`, where we edit the value and assign it *back* into the same variable, by either returning a new object, or adding to the original, like so:
```python 
>>> i = 5
>>> old = id(i)
>>> i += 1
>>> old == id(i)
	False
```

- Augmentability is already implemented with the presence of basic [[#Arithmetic Operations]], but the issue is that the objects are by-default immutable. 
	- What this means, is that calling augmented operations returns a new instance of the object with those attributes, as seen with `id(i)` right above. 
- What we can do if we want to make an object mutable ourselves is implement a custom calling mechanic. 
- We can do so with `__i[NAME]__`, where the `i` implies *in-place*
- It can either return `self`, or `NotImplemented`, or a new object if required!