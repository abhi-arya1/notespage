---
title: Topic 8 - Comprehensions
date: 2024-03-08
---

> Full notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/Comprehensions/)

Contents

- [[#Introduction]]
- [[#List Comprehensions]]
  - [[#Increasing Complexity in Comprehensions]]
- [[#Set Comprehensions]]
  - [[#The meaning of `hashability`]]
- [[#Dictionary Comprehensions]]
- [[#What about Tuple Comps?]]
- [[#Asymptotic Analysis of Comprehensions]]
  - [[#How do ranges work exactly?]]
  - [[#How do lists work exactly?]]

### Introduction

- Writing larger programs in any language requires you to sort a variety of information into the appropriate data structures.
- In Python, we see that the pattern for a lot of these structures is very similar (i.e. lists, tuples, dictionaries)
  - If we consistently follow a _pattern_, why can't we simplify it further?
- Python offers the **comprehension**, a feature for building a variety of common data structs

#### List Comprehensions

- A _list comprehension_ builds a list based on a description of values, rather than writing a loop and adding to an empty list.
- Let's write a few:

```python
>>> [x for x in range(10)]
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> [x * x for x in range(10)]
    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> [x.upper() for x in 'Boo']
    ['B', 'O', 'O']
>>> [x for x in range(10) if x % 2 == 0]
    [0, 2, 4, 6, 8]
```

- Let's break down the syntax so we can apply generalization to further understand this:
  - A comprehension is driven by a _variable_, such as `x`, to describe a sequence of values, one at a time.
  - Within brackets (for lists), or curly braces (for dicts or _sets_), we can define the expression that we want to generate _inside_ of that structure
  - We follow the _variable_ with a `for *variable* in *iterable*` clause, which lets us define the range, other list, or whatever else we need to loop through to generate this new list.
  - Optionally, we can follow this with a conditional "if" to include certain `*variable*` conditions, where only truthy values are added

```python
# in basic terms:
[variable for variable in iterable ?(if truthy(variable))]
```

- List comprehensions aren't a _come one come all_ remedy for all problems though, but when it's possible to create them, then it makes reading list generations (foreshadowing) a whole lot easier.
  - Try writing any of the comprehensions above, and see how much wordier they get.
- One **limitation** to note is that all types must match! Otherwise, the entire comprehension will fail!

##### Increasing Complexity in Comprehensions

- While an initial list comprehension demands the structure that we defined with `[variable for variable in iterable if...]`, we can actually pair this with as many loop `for` and conditional `if` clauses as we would like!

```python
>>> [x * y for x in 'ABC' for y in (1, 2, 3)]
	['A', 'AA', 'AAA', 'B', 'BB', 'BBB', 'C', 'CC', 'CCC']
# We're iterating over every value in 'ABC'.  For each of those, we're
# iterating over every value in (1, 2, 3).  Every combination of these
# values is passed to the initial expression, and we get a list of the results.

>>> [x for x in range(10) if x % 2 == 0 if x != 6]
    [0, 2, 4, 8]
# Multiple if clauses can establish two separate filtering conditions,
# all of which have to be truthy to add to the list!
```

- Additionally, because Python is...well...Python, there's a lot of _ahem_ interesting possibilites:

```python
>>> [[x * x for x in range(4)] for y in range(4)]
    [[0, 1, 4, 9], [0, 1, 4, 9], [0, 1, 4, 9], [0, 1, 4, 9]]

>>> values = [[x * x for x in range(4)] for y in range(4)]
>>> values[0] is values[1]
    False
    # The sublists have equivalent values, but they're separate lists.

>>> values[0][0] = 999
>>> values
    [[999, 1, 4, 9], [0, 1, 4, 9], [0, 1, 4, 9], [0, 1, 4, 9]]
    # So, mutating one sublist leaves the others unchanged.

>>> [[0, 1, 4, 9]] * 4
    [[0, 1, 4, 9], [0, 1, 4, 9], [0, 1, 4, 9], [0, 1, 4, 9]]
     # This looks like the same result.  But let's look more closely.

>>> other_values = [[0, 1, 4, 9]] * 4
>>> other_values[0] is other_values[1]
    True
# All of the sublists are actually the same list (i.e., other_values  contains four references to the same list).

>>> other_values[0][0] = 999
>>> other_values
    [[999, 1, 4, 9], [999, 1, 4, 9], [999, 1, 4, 9], [999, 1, 4, 9]]
    # So, mutating one sublist mutates them all.
```

#### Set Comprehensions

- Sets are very similar to list comprehensions, except we just change the surrounding `[]`with `{}`
  - However, there's two main differences:
  1.  Sets cannot contain duplicate elements, while lists can. Sets automatically enforce this
  2.  Sets cannot contain _mutable_ elements, because if sets could contain those, then elements could be auto-removed through changing them to another existing element.
- Here's how to do set comps:

```python
>>> {x for x in range(10)}
	{1,...,10}
>>> {x for x in range(10) if x % 3 != 0}
    {1, 2, 4, 5, 7, 8}  # Filtering with "if" works, too.

>>> {x.upper() for x in 'Hello Boo!'}
    {'H', 'L', '!', 'E', 'B', ' ', 'O'}
    # Duplicates eliminated, as expected.

>>> {list(range(x)) for x in range(3)}
    Traceback (most recent call last):
      ...
    TypeError: unhashable type: 'list'
    # Mutable elements not permitted, as expected.
```

##### The meaning of `hashability`

- **This is a precursor to [[Topic 12 - Python's Data Model#Hashing]]**
- When we add to a set, there are two things that need to be done:
  - Determine whether the element is already in the set
  - If not, store the element in the set
- The issue is that if we wanted to look for the item in the set _each time_ that we did a set operation, we'd use so much $O(n)$ (worst case) time that we'd be absolutely cooked.

  - Either that or we sort, which is also expensive

- The Computer Science field's workaround to this dilemma is _Hashability_.
  - Hashability is determined by a data structure called a hash table, which we can worry about in ICS 46, but here's an overview:

1. Every element has a unique _hash_ that describes the element, that is obtained through an operation called _hashing_, which boils an immutable element down to its bare minimum essence.
2. Once we have the hash values and hash table, we can just look for those immutable values based on what's given, so we can do all of this in $O(1)$ time!

- A Python object is **Hashable** iff:
  1.  It has a `__hash__` method that determines the objects hash
  2.  It has a `__eq__` method that determines whether it is equivalent to another object
- Python provides a built in `hash()` function that lets you invoke different object's `__hash__` methods!

- Lists don't provide a `__hash__` method, which means they aren't hashable. The underlying reason they shouldn't provide a `__hash__` method is because they're mutable, which would presumably mutate their hash, as well.

#### Dictionary Comprehensions

- Dictionaries are more similar to sets than they are to lists, but they also have differences:
  - The keys of a dictionary must be hashable, just like the elements of a set. This is because they must be unique (and we need to be able to find them quickly), also like the elements of a set.
  - Each key in a dictionary has a value associated with it. The values can be anything you'd like — mutable or hashable, duplicated or not.

```python
# Here are examples of this!

>>> {'A': 1, 'B': 2, 'C': 3}
    {'A': 1, 'B': 2, 'C': 3}
    # A dictionary literal has keys and values separated by colons,
	# surrounded in its entirety by curly braces.

>>> {x: len(x) for x in ['Boo', 'is', 'happy', 'today']}
    {'Boo': 3, 'is': 2, 'happy': 5, 'today': 5}
    # The same notation appears here with the same meaning: curly braces and colons. Notice that duplicate values are fine, as they should be; 5 appears twice.

>>> pairs = [('A', 1), ('B', 2), ('C', 3)]
>>> {key: value for key, value in pairs}
    {'A': 1, 'B': 2, 'C': 3}
	# We're sequence-assigning each element of the tuples within
	# pairs into the two variables key and value here.

>>> {x.upper(): x.lower() for x in 'Hello Boo!'}
    {'H': 'h', 'E': 'e', 'L': 'l', 'O': 'o', ' ': ' ', 'B': 'b', '!': '!'}
    # Duplicate keys are eliminated.

>>> {list(range(x)): len(range(x)) for x in range(3)}
    Traceback (most recent call last):
      ...
    TypeError: unhashable type: 'list'
    # Unhashable keys are disallowed.
```

- It's worth noting that the proper usage of dictionary comps is to take a sequence and use each element to determine _both_ a key and its associated value in a "tuplized" format with a `:`. We'll not be using these often.

#### What about Tuple Comps?

- Let's try a tuple comp:

```python
>>> (x * x for x in range(7))
    <generator object <genexpr> at 0x000002120B11CA50>
```

- This is a precursor to [[Topic 10 & 11 - Generators & Using Generators]], which we'll touch on there. What the issue is, is that parentheses define _generator comprehension_, not tuple comprehension. Generators are useful, but yet to be learned!

- If we _really_ needed the tuple, we can run:

```python
>>> tuple(x * x for x in range(5))
	(0, 1, 4, 9, 16, 25) # done!
```

#### Asymptotic Analysis of Comprehensions

- With more tools requires more analysis to pick the _best candidate for the job_
- Let's consider an example:

```python
def f(n):
	return [x * n for x in range(n)]

>>> f(5)
	[0, 5, 10, 15, 20]
>>> f(10)
	[0, ..., 90]
>>> f(0)
	[]
```

- Now that we have an understanding of what the function does:
  - The list comprehension is driven by `range(n)` containing `n` elements from `0` to `n-1` (inclusive)
  - All elements in the range are used, since there's no `if` clause.
- Since we loop through `n`, but ranges pop off a stack with their `__next__` methods, we don't need to worry about that costing extra time. However, now that we've determined that, we can determine that our final time complexity is $O(n)$ for a loop _n_ times.

  - (Full Derivation _Thornton Style_$^{tm}$ available on lecture notes at the [top](#Introduction))

- What about memory usage? What does `f` use memory for?
  - It builds a range of *n* elements. One open question, then, is how much memory they require to do that.
  - It builds a list, which is initially empty, but eventually contains *n* elements.

##### How do ranges work exactly?

- Ranges are a succinct way of describing sequences of integers. What makes them so succinct is not just the choice of syntax; their succinctness is enabled by the fact that not all sequences of integers can be described by them.

- If we want a range of integers containing 0, 1, 2, 3, and 4, we can say `range(5)` to describe it.
- If we want a range of integers containing -2, -4, and -6, we can say `range(-2, -8, -2)` to describe it.
- But if we want a range of integers containing 0, 1, 4, 9, 16, we're out of luck. Ranges only offer the ability to describe sequences of integers with the same "step" (i.e., where every element differs from the previous element by the same amount).

  - So, ranges are limited in their capabilities, though not so limited that they don't have many realistic uses. But their limitations don't just mean that their syntax is simple; it also means that they're a lot more efficient than they might be if they had a more difficult problem to solve.

  - Ranges are *iterable*, which means that they support a protocol that allows us to ask them for a single element at a time, continuing until they tell us that there are no more elements. We'll discuss the details of the iterable protocol a little later in the course, but let's think about how ranges might be able to solve this kind of problem.

- Ranges know what their `start` value is. So, the first time we ask them for an element, they'll know what to give us: the `start` value.
- Let's assume that ranges remember what they returned the last time we asked them for an element; we'll call that value `last`. If so, how do they know what to return the next time? The answer is simple: `last + step`.
- How do ranges know to tell us when there are no more elements? When `last + step` exceeds `stop`.

- A range is entirely described by its three components: `start`, `stop`, and `step`. While we're iterating a range, the only additional thing a range would need to know is `last`.

- So, all in all, a range requires the same amount of memory, no matter how many elements it has. We would state that asymptotically as $O(1)$

##### How do lists work exactly?

- A Python list is a sequence of elements, where each element has an *index*. The index of the first element is 0, the index of the second element is 1, and the indices increase sequentially from there. (This scheme is often called *zero-based indexing*.)

- Those indices aren't just conceptual; they also provide a powerful but simple way for Python to find its way around the list. Internally, a list contains a reference to the first of a sequence of its elements. The elements are arranged immediately next to one another in memory. Each element is actually a reference to a Python object, which means that each element has the same size in memory — references are the same size, no matter what kind of object they refer to. So, if a list knows where its first element is, it can easily know where the element at any other index is; all it needs to do is some arithmetic.

- **address of element at index #*n* = address of element at index #0 + (the size of a reference \* *n*)**

- In other words, if we know the index of what we want in a list, we can obtain it in a constant amount of time, because all the list needs to do is one multiplication and one addition, no matter how big the list is and no matter what the index is.

```python
>>> def element_at(elements, index):
...     return elements[index]
...              # This function runs in O(1) time.
```

- Determining the cost of adding elements to a list is a little bit trickier, because the time needed is situationally dependent. Understanding why requires us to know two more facts about the internals of a Python list:

- Lists know their length, i.e., they know how many elements they store. They don't have to count them; they keep track of the length separately and adjust it when necessary.
- The address of a list's first element doesn't change over time, but the number of elements in a list can be extended (more or less) without bound and (more or less) without cost.

  - (That second fact turns out to be a half-truth in practice — there's more to the story than this — but is a good enough assumption for now.)

  - If those things are true, then how do elements get added to a list? It depends on where they're being added.
    - If we add an element to the end of the list, the list will know the index in which to store it. If the length of the list is `x`, the element should be stored in index `x`.
    - If we add an element to the beginning of the list, we want it to become the element at index 0. To make that possible, all of the elements in the list will need to be moved forward — the element at index 10 will become the element at index 11, 9 will become 10, and so on — because the address of the first element can't change.

- These two functions solve the same problem in the reverse order. Even though their results are the same, one is wildly more expensive than the other.

```python
>>> def relatively_cheap(n):
...     values = []
...     for i in range(n):
...         values.append(i)
...     return values
...
>>> def relatively_expensive(n):
...     values = []
...     for i in reversed(range(n)):
...         values.insert(0, i)
...     return values
...
```

- The `relatively_cheap` function appends elements to the end of the list. Each of those calls to `append` take *O*(1) time — it's not more expensive to add the 100th element than it is to add the first — so the total time spent is *O*(_n_).

- The `relatively_expensive` function inserts elements at the beginning of the list. Each of those calls to `insert` require shifting all of the elements already in the list, which means they cost more the longer the list gets. The first call to `insert` shifts no elements, then inserts; the second call shifts one element, then inserts; the third call shifts two elements, then inserts; and so on. So, the total time spent looks something like this:
  $$ 1 + 2 + 3 + \ldots + n $$

- That sum appears frequently in computer science, so it's worth knowing that it simplifies to *n*(*n* + 1) / 2, or, when simplified further to an asymptotic notation, *O*(_n_2). So, the time spent in `relatively_expensive` grows in proportion to the square of _n_, which means as *n* gets larger, the impact of the difference between these functions becomes significantly greater. Before long, *n* will be large enough that the time spent will be problematic.

- This is why it's so important for us to know a little bit about how the tools built into our languages and libraries work. We don't need to know every detail, but if we don't know the significant ones — such as the relative costs of manipulating lists in different ways — we won't realize when we've stepped on a performance landmine.

#### Memory Usage of List Comprehensions

- Finally, we can come back to learn the real memory usage of list comprehensions.

  - Building a range of $n$ elements requires $O(1)$ memory, because ranges don't store their elements, they calculate them when asked.
  - Building a list of $n$ elements requires different amounts of memory, but since list comprehensions, by design, add from _beginning-to-end_ order, no shifting is required, no $O(n^2)$ comes up!
    - As such, the total memory required to build a list of $n$ elements is $O(n)$

- As such: $$ O(1) + O(n) = O(n) $$
  - This, the total memory required to run `f(n)` is $O(n)$
