> Full Notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/Iteration/)

Contents 
- [[#Introduction]]
- [[#Manual Iteration]]
	- [[#Asymptotic Analysis of List Iteration]]
- [[#The Iterable Protocol, Exemplified]]
- [[#Asymptotic Analysis of MyRange]]

### Introduction 
- Earlier, we introduced *iterables*, which describe Python objects that can be *iterated*. In doing so, it produces a sequence of values, one at a time. 
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
		- *Pop off the stack till its empty*

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

- Other types of *iterables*, such as `set`, `string`, or `range` support this protocol in a similar manner!

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

- A `list_iterator` requires _O_(1) memory, no matter how large the list is, because it always stores exactly two things: a reference to the list and an integer index.
- Asking a `list_iterator` for its next element takes _O_(1) time, because it always obtains an element from a known index. It's no more expensive to do that with the 500th element than it is with the fifth.

- So, all in all, we can iterate an entire list of _n_ elements in _O_(_n_) time, while using _O_(1) additional memory, which is as cheap asymptotically as you'd ever expect iteration of _n_ elements to be.

- We've essentially reached the end of as far as we can go in theory, so now let's implement an iterator to see how they actually work 


### The Iterable Protocol, Exemplified

```python
class MyRangeIterator:
    def __init__(self, myrange):
        'Initializes a MyRangeIterator, given a MyRange to iterate.'
        self._myrange = myrange
        self._next = myrange.start()


    def __iter__(self):
        return self # return itself because 
			        # we define a custom __next__ method 


    def __next__(self):
        if self._next >= self._myrange.stop():
            raise StopIteration # if out of range, stop 
        else:
            result = self._next # get old to return  
            self._next += self._myrange.step() # add new to next
            return result # return old 
            # saves us looping, saves us memory! $O(1)$ time!



class MyRange:
    def __init__(self, start, stop = None, step = None, /):
        '''
        Initializes a MyRange, given either a stop value or start, stop, and
        (optionally) step values.
        '''
        if stop is None:
            stop = start
            start = 0

        if step is None:
            step = 1

        self._require_int(start, 'start')
        self._require_int(stop, 'stop')
        self._require_int(step, 'step')

        self._start = start
        self._stop = stop
        self._step = step


    @staticmethod
    def _require_int(value, name):
        if type(value) is not int:
            raise TypeError(f'{name} must be an integer, but was {value}')


    def start(self):
        'Returns the start value associated with a MyRange.'
        return self._start


    def stop(self):
        'Returns the stop value associated with a MyRange.'
        return self._stop


    def step(self):
        'Returns the step value associated with a MyRange.'
        return self._step


    def __iter__(self):
        return MyRangeIterator(self) # get the iterator given attrs


    def __repr__(self):
        step_repr = f'' if self._step == 1 else f', {self._step}'
        return f'MyRange({self._start}, {self._stop}{step_repr})'
        # get the shell representation 
```
- Here's a link to a [complete set of unittests](https://ics.uci.edu/~thornton/ics33/Notes/Iteration/test_myrange.py) for MyRange in case it helps with understanding, though after understanding the protocol it's pretty clear what's going on. 
- Do read the [full notes](https://ics.uci.edu/~thornton/ics33/Notes/Iteration/test_myrange.py) for a proper overview though!

#### Asymptotic Analysis of MyRange 
- Let's start with memory. 
	- Regardless of how large our ranges are, they have a `_start, _stop` and `_step` attribute each. $O(1)$ there. 
	- When in Iteration, a MyRangeIterator uses a `_next` variable to store states. Again, $O(1)$, which is only added to. 
- Overall, the total memory complexity is $O(1)$. 
	- To be fair, that's because `MyRange` is so limited in terms of what it can store, because its entire sequence of elements can be boiled down to a formula involved `start`, `stop`, and `step`. This wasn't magic; it was simply taking advantage of the simplifying properties of the problem we were solving.

- How about time? 
	- Initialization is just three values, so $O(1)$ time, regardless of the shuffling and comparisons in the `__init__`, because no loops are really used. 
	- Iterating a `MyRange` first creates an iterator then calls `__next__` on it $n$ times. 
		- Creation is $O(1)$, 
		- `__next__` calls are each $O(1)$, so $an \implies O(n)$ time complexity for the entire `MyRange`
- Solid! 

- Thus far though, `MyRange` objects can't index, get lengths, check boolean equality, or slice, so we'll implement those once we learn the #PDM ([[Topic 12 - Python's Data Model]])

