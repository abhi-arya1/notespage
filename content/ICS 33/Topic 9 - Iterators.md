---
title: Topic 9 - Iterators
date: 2024-03-08
---

> Full Notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/Iteration/)

Contents

- [[#Introduction]]
- [[#Manual Iteration]]
  - [[#Asymptotic Analysis of List Iteration]]
- [[#The Iterable Protocol]]

### Introduction

- Earlier, we introduced _iterables_, which describe Python objects that can be _iterated_. In doing so, it produces a sequence of values, one at a time.
- Python supports iteration in many ways:
  - `for` loop
  - `while` loop
  - comprehensions
  - `list` constructor (iterates the argument)
  - `set` constructor (iterates the argument)
- However, one thing that we're building towards here is the PDM ([[Topic 12 - Python's Data Model]]), so we want to be able to design objects that can be iterated in custom ways, in the most efficient asymptotic manner.

### Manual Iteration

- Python defines iteration protocol as follows:
  - An `iterable` is a sequence of objects that we can `iterate`
  - An `iterator` is an object that manages the process of producing that sequence of elements once.
    - It can keep track of how many objects we've seen, which one we'll see next, and whether we haven't seen something yet.
  - To reduce complexity, the iteration protocol manages iterables, rather than the iterables managing their iteration themselves.
- There's two things to do here:

  - Start a new iteration by asking an iterable object for an iterator
  - Ask an iterator for the next element that hasn't been seen, along with a way for us to tell if we've seen them all
    - _Pop off the stack till its empty_

- Python provides two built-in functions, `iter` and `next`, for these respective purposes.

```python
>>> values = [1, 2, 3]
>>> i = iter(values)
	# This is how we create an iterator for an iterable object.
>>> type(i)
    <class 'list_iterator'>
    # A list_iterator is an iterator that can iterate a list.
>>> next(i)
    1  # This is how we ask an iterator for the next element.
>>> next(i)
    2
>>> i2 = iter(values) # Multiple iterators can exist simultaneously.
>>> next(i2)
    1
>>> next(i)
    3
>>> next(i)
    Traceback (most recent call last):
      ...
    StopIteration
	# A StopIteration exception is raised when there are no more
	# elements.
>>> next(i2)
    2 # When one iteration has finished, others are unaffected
```

- Other types of _iterables_, such as `set`, `string`, or `range` support this protocol in a similar manner!

#### Asymptotic Analysis of List Iteration

- Let's do an example:

```python
>>> values = [1, 2, 3, 4]
>>> i = iter(values)
>>> next(i)
    1

>>> del values[1]  # This is the next element we would have seen.
>>> next(i)
    3 # The removed element was simply skipped, and we got back
      # the next element still in the list, which seems reasonable.

>>> values = [1, 2, 3, 4]
>>> i = iter(values)
>>> next(i)
    1

>>> values.insert(1, 17)
	# Inserting an element after the one we just saw from the iterator.
>>> next(i)
    17 # The inserted element emerged in the proper sequence.

>>> values = [1, 2, 3, 4]
>>> i = iter(values)
>>> next(i)
    1
>>> next(i)
    2

>>> del values[0]  # Removing an element we already saw.
>>> next(i)
    4
    # An element we hadn't seen yet, but is still in the list, was skipped. That seems a little stranger.
```

Mutating a data structure while you're in the midst of iterating it is generally not a good idea, if it can be avoided, but these three examples give us some insight about how `list_iterator` works internally, because there's a straightforward technique that would behave in exactly this way. A `list_iterator` could store two things: a reference to the list being iterated and the index of the next element to be returned. (Since modifications to the list affect iteration, we know the iterator isn't storing a copy of the list.)

- When initially created, it would store a reference to the list, along with the index 0.
- When asked for its next element, it would return the element at the index it's storing, then add 1 to that index.
- When the index it's storing is greater than or equal to the list's length, asking it for its next element would cause it to raise `StopIteration`.

This also gives us some insight about the time and memory cost of iteration.

- A `list_iterator` requires *O*(1) memory, no matter how large the list is, because it always stores exactly two things: a reference to the list and an integer index.
- Asking a `list_iterator` for its next element takes *O*(1) time, because it always obtains an element from a known index. It's no more expensive to do that with the 500th element than it is with the fifth.

So, all in all, we can iterate an entire list of *n* elements in *O*(_n_) time, while using *O*(1) additional memory, which is as cheap asymptotically as you'd ever expect iteration of *n* elements to be.

### The Iterable Protocol

-
