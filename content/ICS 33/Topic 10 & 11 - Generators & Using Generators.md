Contents
- [[#Topic 10 - Generators]]
	- [[#Generators in Python]]
		- [[#Q Why not just use iterators?]]
	- [[#Applying Generators]]
- [[#Topic 11 - Using/Combining Generators]]
	- [[#Generator Comprehensions]]
	- [[#Q How are Generator Comprehensions Better]]
	- [[#Infinite Generators]]
	- [[#Combining Generators]]
	- [[#Generators in the Standard Library]]

## Topic 10 - Generators 

> Full Topic 10 Notes [here](https://ics.uci.edu/~thornton/ics33/Notes/Generators/)

Contents
- [[#Introduction (Generators)]]
- [[#Generators in Python]]
### Introduction (Generators)
- We saw previously that the #PDM includes two related concepts that address sequences, from [[Topic 9 - Iterators]]: 
	- *Iterable* is an object that produces a seq. of values, one at a time 
	- *Iterator* is an object that manages the process of *Iteration* through an *Iterable*

- Data structures aren't entirely analogous with iterables though. `range` is a formulaic object, not a data structure, but it still has the same possibilities as a list, because of its nature. 

- But sometimes, what's the solution that we can find? Let's consider reading from a file in Python
	- We'd first load the file, then call `read()` on the file to initialize a `contents` variable, then `splitlines()` to get a list of strings, then iterate through that very last list. 
		- This isn't *wrong*, but consider a file of 10_000_000 lines, each with 1_000 characters. We'd read 10_000_000_000 characters into one string called `contents` (10gb), then split it into a list which requires another 10gb because we can't toss `contents` till it's done splitting. 
	- We need to reduce this impractical $O(n)$ to an $O(1)$, and we can do so with the magic of *generators* 
- Files offer a `readlines()` method, which lets us loop through their result one at a time. We'd only need 10gb now, because we're storing all the lines in the result of the `readlines()`, and no longer would we have to split it. We could just directly loop. 
	- Still $O(n)$, but faster memory wise. What's better?

- Well, Python's files are natively iterable, so if we defined something called `process()` somewhere, we could effectively do: 
```python
with open(file_path, 'r') as the_file:
    for line in the_file:
        process(line)
```
- Now there's no extra saving, and we made a <u>pipeline</u>, which waits until the `process` is finished before hopping to the next line. We just got our memory complexity from 20gb to effectively $O(1)$, even though time is $O(n)$, but that really can't be helped. 

- But, not everything is great -- we've sacrificed customizability and future-proofing for an efficient bit of code. Thankfully, Python has a little *fix* to this, so to speak. 
	- Let's figure out how to obtain a sequence of results one at a time in a customized manner, with *generators*

#### Generators in Python
- A generator is a function that returns a sequence of results rather than just a single one. 
- Not only that, but a generator doesn't return all of its results at once, but rather *one at a time*. Each time a generator function is called, it `yields` you a value, then calculates the next step, for asymptotic optimization. 

- Let's view an example: 
```python
>>> def int_sequence(start, end):
...     current = start
...     while current < end:
...         yield current
...         current += 1
...
>>> int_sequence(3, 8)
    <generator object int_sequence at MEM_ADDR>
```
- Functions that return generators are known as *generator functions*, that have at least one `yield` statement!
- Generators can be iterated with `for`, and they can be constructed into a `list()` to get all of their values (remember, list constructors iterate their objects!) 
- How does it work:
	- When a generator function is first called, a generator object is returned, but none of the function's body is executed, until it is *iterated* 
	- When the generator is first asked for a value, the generator function's body begins executing, continuing until the first time it reaches a `yield` statement. At that point, the yielded value is returned, then the generator function's body stops its execution, but remembers where it left off — including the values stored in its local variables.
	- The next time the generator is asked for a value, the generator function's body picks up where it left off, continuing until the next time it reaches a `yield` statement, at which point it's returned and execution is paused again.
	- This process continues until the generator function is exited (e.g., we fall off the end of it, we encounter a `return` statement, or an exception is raised). At that point, the generator will produce no more values.
- So, the mechanics of that behavior follow the following set of steps. 
	- Initialize but don't return -> next(i) -> yield first value stop execution -> next(i) -> start execution from the point where it left off at the last `yield`, and keep going until iteration ends and we can't satisfy the condition to yield any longer.

- What if we include a `return` block in a Generator?
	- An empty `return` block (which, by default, returns `None`), will simply stop iteration *instantly*
	- If a return statement returns a value, then the generator raises `StopIteration` with a value, that we can access through `except StopIteration as s: s.value`!
```python
def return_gen():
    yield 1
    return 13
    yield 5

    
try:
    i = return_gen()
    print(next(i))
    print(next(i))
except StopIteration as s:
    print(s, s.value)
>>> 1
>>> 13 13

```
- If a Generator raises it's own exception, that exception gets raised as normal with the expected attributes. 

##### Q: Why not just use iterators?
- A: Simplicity! 
- Let's take a look at `int_sequence`, our generator from above, implemented as a class:

```python 
# AS A GENERATOR 
def int_sequence(start, end):
    current = start
    while current < end:
        yield current
        current += 1


# AS A CLASS
class IntSequence:
    def __init__(self, start, end):
        self._start = start
        self._end = end
        self._current = start


    def __iter__(self):
        return self


    def __next__(self):
        if self._current >= self._end:
            raise StopIteration
        else:
            result = self._current
            self._current += 1
            return result
```
- We had to add an extra layer of complexity, storing attributes, and defining a manual `__iter__` and `__next__` protocol. 
- This is not to say that we never want to write a manual iterator, and there are definitely problems that arise with generators that are solved by iterators, but we are adding *another* tool to help us out!

#### Applying Generators 
- Let's rewrite the original `process` lines problem through a generator protocol, and let's also chain generators together!
```python 
def readlines(file_path):
	with open(file_path, 'r') as f: 
		for line in f:
			yield line 

def process(line) ... # implementation hidden 

def process_lines(lines): 
	for line in lines: 
		yield process(line)

if __name__ == "__main__": 
	for line in process_lines(readlines('C:/../')):
		print(line)

# Nice!

# We could even create a shell input function 
# to optimize: 
def read_lines_from_shell():
    while True:
        next_line = input('Next line: ')

        if next_line == '':
            break

        yield next_line
    # Cool!
```




## Topic 11 - Using/Combining Generators 

> Full Topic 11 Notes [here](https://ics.uci.edu/~thornton/ics33/Notes/UsingGenerators/)

Contents
- [[#Introduction (Using Generators)]]
### Introduction (Using Generators) 
- Great! We've defined what a generator is, and the lexical semantics for how to use them, as well as how Python views and rules over them. 
- *Now what?*
	- Time to work with them! 
- But first, let's make sure we fully understand them, so a quick recap
	- Generators give us a natural way to separate the means by which we generate a sequence of values from what we might want to do with them. One of the hallmarks of good software design is what you might call "keeping separate things separate," and generators certainly help with that. Generators are focused on what values to produce in a sequence, but are entirely unconcerned about what will be done with them.
	- Because they're _lazy evaluated_ — doing their work piecemeal, one value at a time — they are able to perform a potentially large number of tasks while avoiding the need to store a similarly large number of results. Even if you ask a generator to produce 100,000,000 elements, it'll only ever remember one at a time.
	- The fact that they are lazy evaluated also allows them to stop working for arbitrary reasons, without having been designed for it ahead of time. They stop working when we stop asking them to, as opposed to them having to anticipate the conditions under which they might need to stop.
- Now, applications time 

#### Generator Comprehensions
- When we were taking a look at [[Topic 8 - Comprehensions]], we saw that a comprehension statement in between parentheses `()` was a generator object, so let's write one to see how it works:
```python 
>>> (x * x for x in range(3))
    <generator object <genexpr> at 0x000002CEA10F4A50>
 # Generator comprehensions return generators, just like
 # generator functions do.
>>> i = (x * x for x in range(3))
>>> next(i)
    0 
    # Just like any other generator, these generators can be
    # iterated.
>>> next(i)
    1
>>> next(i)
    4
>>> next(i)
    Traceback (most recent call last):
      ...
    StopIteration
    # Just like any other generator, these generators raise
    # StopIteration when there are no more values in the
    # sequence.
```

#### Q: How are Generator Comprehensions Better?
- **(A: *Lazy Evaluation*)**
- Generators imploy *lazy evaluation*, a technique that is positive, such that it computes only what is necessary while computing the least amount of resources. Only calculate a result when <u>you're sure you'll need it</u>
	- As such, a generator comprehension is iterating its inputs. When you write `[x ** 2 for x in range(1000000)]`, it gives you everything all at once, but `(x ** 2 for x in range(1000000)` only gives you a value *when you ask for it* 
	- The generator that is built here is iterated somewhere else!
- As such, generators don't *materialize* their sequence, but rather calculate it in $O(1)$ time intervals, for optimization!
- Now that we understand the impact of lazy evaluation, though, we can use iterators, generators, and generator comprehensions to enable some patterns that might otherwise seem a little strange.


#### Infinite Generators 
- We can write infinite generators so long as they continually loop:
```python
def seq(start=0):
	curr = start
	while True: 
		yield curr
		curr += 1 

# this generator would run infintely
```
- We can't ever materialize this, because `list(seq(0))` would take $O(\infty)$ time to run, but we shouldn't avoid this, because it *can* be useful. 


#### Combining Generators 
- Let's write an example called *take*, which takes any iterable, such as a generator, list, etc. and takes a number of variables from it. 
```python 
def take(items: int, seq: _iterable): 
	i = iter(seq) 
	for _ in range(items):
		try: 
			yield next(i)
		except StopIteration: 
			break 
```
- This will take the first `items` amount of items from `seq`, and if it can't anymore, then it ends. This lets us create a pipeline of items from sequence to take, which is a generator in itself, that we can then materialize with `list()` or something, which is another generator comprehension! 
- Finally, we should take note of the fact that generator functions and generator comprehensions both build generators, so we would reasonably expect to be able to combine them in ways similar to what we've seen. For the purposes of clarity, we can introduce names for our intermediate sequences — notably without burdening our performance, since storing generators in variables doesn't cause them to be materialized.

```python
>>> from_zero = sequence(0)
>>> squares = (x * x for x in from_zero)
>>> list(take(10, squares))
    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
# Only ten values from sequence are generated, only ten squares
               # are calculated, and only ten elements are appended to the list.
>>> list(take(10, (x * x for x in sequence(0))))
    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
# We could also have written this in one expression, though these become more difficult to read as they consist of more moving parts and the parentheses nest more deeply.
```


#### Forwarding an entire generator via another 
- Let's take the idea of combining two generated lists. 
- The simplest design is this:
```python
>>> list(take(10, sequence(0))) + list(take(5, sequence(20)))
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 20, 21, 22, 23, 24]
```
- Asymptotically though, we build 3 lists. The first, second, and their concatenation, which is a $O(n^3)$ time complexity! 
- How can we reduce this?
	- We could define a function that appends them all to a list, but then looping that is $O(n^2)$. How can we do it in $O(n)$?
- Let's combine generators!
```python 
>>> def concatenate(first, second):
...     for value in first:
...         yield value
...     for value in second:
...         yield value
...
>>> list(concatenate(take(10, sequence(0)), take(5, sequence(20))))
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 20, 21, 22, 23, 24]
```
- Woohoo! We have a tool that we can use wherever we need a concatenated iterator, that also runs in an optimal manner. 

##### `yield from`
- In the previous example, we had to use loops, but since iterables are *generable* in their own way, since generators are just a *one-at-a-time* form of iterables, we can use Python's special `yield from` keyword. 
```python 
>>> def concatenate(first, second):
...     yield from first
...     yield from second
...
>>> list(concatenate(take(10, sequence(0)), take(5, sequence(20))))
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 20, 21, 22, 23, 24]
```
- In Python, the `yield from` keyword yields every element from an iterable or another generator, one at a time, as though we'd written a for loop like shown above. 

- Another thing to note is that unpacking can be used in any case that we want, so `*generator` is as valid as `*list`, because all iterables can be unpacked in Python. 
- Let's use that to create the *ultimate* concatenation generator:
```python
>>> def concatenate(*sequences):
...     for sequence in sequences:
...         yield from sequence
...
>>> list(concatenate(take(7, sequence(0)), take(5, sequence(11)), take(2, sequence(17))))
    [0, 1, 2, 3, 4, 5, 6, 11, 12, 13, 14, 15, 17, 18]
>>> list(concatenate())
    []
```
- Nice! 

#### Generators in the Standard Library 
- Generators that we wrote above follow a standard structure that can be made common. Examples include: 
	- For each value produced by an iterable, pass it to a function, generating a sequence of the results.
	- Generate a sequence of consecutive integers, starting with a specified value.
	- Take the first _n_ elements from an iterable.
	- Concatenate multiple iterables together into one, with all of the values of the first followed by all of the values of the second and so on.
- Python has some built-in utilities to make that easier: 
```python 
list(map(func, _iterable)) 
# maps a function to each element of the iterable 

list(filter(func, _iterable)) 
# returns a list where each item is truthy under func(item)

any(iterable) 
# returns True if any of the elements are truthy 

all(iterable (OR GENERATOR COMPREHENSION)) 
# returns True if all of the elements are truthy 

enumerate(iterable) 
# returns a list of values of the format (idx, element) 
# for more advanced looping

zip(iter1, iter2, ...)
# returns an iterator that zips the iter1 and iter2 into two lists
# format is (iter1_i, iter2_i, iterN_i) (for any number of iterables)
```
- Functions that return iterators calculate their result one element at a time (just like generators do). Functions that can process iterables lazily — like `any` or `all`, which might know their answer after processing their first input element — will generally do so.

- The `itertools` standard library utility also provides a variety of utilities for iterables, but if they come up on the final, they'll be mentioned