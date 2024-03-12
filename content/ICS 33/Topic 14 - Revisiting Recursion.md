---
title: Topic 14 - Revisiting Recursion
date: 2024-03-11
---

> Full notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/Recursion/)

Contents 
- [[#Introduction]]
- [[#Analyzing the Cost of Recursion]]
- [[#Tail Recursion]]
- [[#Multiple Recursion]]
- [[#Better Multiple Recursion]]
- [[#Memoization]]
- [[#Dynamic Programming (DP)]]

#### Introduction
- We've already visited recursion at a *surface* level, but let's take a deeper dive
	- Let's start with `factorial`

```python
def factorial(n): 
	if n == 0: 
		return 1
	else: 
		return n * factorial(n - 1)

# 0! = 1
# n! = n * (n-1)!
```

- This function exhibits *recursion* because it calls itself recursively
- This function exhibits *direct recursion* because `factorial` itself calls `factorial`
	- As opposed to *indirect/mutual recursion* where `factorial` calls another function `foo` that calls `factorial` again 
	- We also say that `factorial` shows *single recursion* since each call to `factorial` only makes at most one further recursive call 

- Remember that recursion operates via a **call stack**, where functions that call other ones are paused until the function above it on the stack is evaluated. 
	- When this stack can't reach a maximum depth, then we get a `StackOverflowError`, or in the case of Python, a `RecursionError` where the maximum recursion depth gets reached. 

##### Analyzing the Cost of Recursion
- When we have *single* *direct* recursion like this, the memory cost is simply $O(n)$, because we need to assign a return variable `n * factorial(n-1)` in *each* call to the stack, so for $n$ calls, $O(n)$ memory complexity exists. The time would also be $O(n)$, since the number of multiplication operations that take place grows to $n$ from $O(1)$
- What would be a more efficient manner?

##### Tail Recursion 
- A *tail call* is when a called function's result also becomes the calling function's result. 
- Let's consider an example

```python
def f(a, b):
    return (a * a, b * 2)

def g(a, b):
    return f(a, -b)

def h(a, b):
    return list(f(a, b))
```

- Both `g` and `h` call `f` --> 
	- `g` evaluates its arguments and then returns the output of `f`
	- `h` evaluates its arguments and then returns the output of `f` in list form, which is not *exactly* what `f` gave it 
- As such, only `g` *tail-calls* `f`. 
- The reason why *tail calls* are important is because instead of storing something in a variable, we are simply returning an already existing state. 
- As such, the memory complexity remains $O(1)$ overall.

- So, *tail recursion* is the technique of writing a recursive function whose recursive calls are tail calls, meaning that each time the function calls itself, it's done, so that whatever the recursive call returns will be its result, too.
	- There is a tradeoff --> We have to design our function to support this style of recursion. 

- As we've seen, in a recursive function, the communication between calls happens in two ways.
	- Communication while the tide is going out happens through the arguments passed to each call.
	- Communication while the tide is coming back in happens through the results returned to each call.
- So we can write these new functions with *accumulator parameters* which tracks an overall result. 
	- We can write `factorial` like this now:
```python
def factorial(n: int, product: int) -> int:
    if n == 0:
        return product
    else:
        return factorial(n - 1, product * n)
```

- Then, for `factorial(5, 1)`, we would finally get `120` as the final return output. 
- But we've added an extra step to this function, and we could default the product to equal 1, but then it'd be a weird setup. As such, we can rewrite this function with a new enclosing scope function: 
```python 
def factorial(n: int) -> int:
    def factorial_tail(n: int, product: int) -> int:
        if n == 2:
            return product * 2
        else:
            return factorial_tail(n - 1, product * n)

    if n < 0:
        raise ValueError(f'cannot calculate factorial of negative value: {n}')
    elif n == 0 or n == 1:
        return 1
    else:
        return factorial_tail(n, 1)
```

- As such, the parameters of the enclosing scope can store a state, while `factorial_tail` actually performs the tail recursion.

- **REMK:** Tail recursion doesn't actually improve recursion depth due to Python's ability to stack trace your recursion. What it does however is reduce the memory complexity of your function to $O(1)$ even though the time has to remain at $O(n)$.

##### Multiple Recursion 
- We say that a function exhibits *multiple recursion* if one call to a function is responsible for *multiple* further calls to a function.
	- Such a function makes one recursive call, directly or indirectly, pauses until it gets a result, and then, after obtaining that result, makes another recursive call.

- We can't show that with factorial, but we can with the *Fibonacci* sequence.
	- `fib(0) = 0, fib(1) = 1, fib(n) = fib(n-1) + fib(n-2)`
	- We can already see a logical approach to this: 
```python 
def fib(n):
	if n == 0:
		return 0 
	elif n == 1:
		return 1 
	else: 
		return fib(n-1) + fib(n-2) 
```
- But remember, after a few recursive calls here, we'd be reaching a point where many many calls have already been determined, so why do we do them again?
	- i.e. fib(5) requires knowing fib(4.....1), which would be a waste of time to calculate those again, since fib(4) already calculates fib(3), but fib(3) is *still* called again. 
- While multiple recursion isn't necessarily *bad* here, it does raise a question: 
	- Why are we <u>repeatedly</u> solving the same problem, and how can we fix that?
		- This will be answered next. 

###### Better Multiple Recursion 
- A better multiple recursion example is when we sum over a list of either nested or element based lists: 
```python
def nested_sum(nested_list):
    return sum(nested_sum(element) if type(element) is list else element \
               for element in nested_list)
```
- Here, we don't recurse over `len(nested_list)` (that's just one `for` loop comprehension)
	- What we recurse over, rather, is the **depth** of the list. 
	- As such, this function only takes $O(n)$ time, where $n$ is `len(nested_list)`
	- Memory store sums would take $O(d)$ memory complexity, where $d = \text{depth of list}$, since we need to store those sums somehow. 

- But there's a few last ways to truly truly optimize what we're doing here....

#### Memoization 
- The reason why the original `fib` is so bad is because it calculates things that the program already *knows*, just because it **has** to do them again. 
	- We can solve this through *memoization*, since we've already calculated them.

- ***Memoization*** (without the ***r***) is the technique of remembering previous results and using them later in-place rather than recalculation. We can spend memory to *store* results rather than *re-calculate* results, which would take less time!

- Here's a memoized implementation of `fib`:
```python
def fibonacci(n):
    # Since we'll be using n as a list index, best to eliminate negative inputs,
    # since negative indices are legal in Python, but not what we want here.
    if n < 0:
        raise ValueError(f'argument cannot be negative, but was {n}')

    # Every value in results[0:n] starts out as None.  In other words, we don't
    # yet know any results, but we have a place to store them as we determine them.
    # Since we need a place to put the base case values, we'll make sure that the
    # list has at least two elements in it.
    results = [None] * max(n + 1, 2)

    # Store the base case results ahead of time, since we already know what
    # they're going to be.
    results[0] = 0
    results[1] = 1

    # The recursive function is nested, so that the initialization of the results
    # only happens once.

    def fibonacci_memo(n):
        # If we have a result already, use it.  Otherwise, calculate it,
        # but store it before returning it.
        if results[n] is not None:
            return results[n]
        else:
            result = fibonacci_memo(n - 1) + fibonacci_memo(n - 2)
            results[n] = result
            return result

    # Call the memoized recursive function and return its result.  Note that we
    # don't need to pass the results list, since it's in fibonacci_memo's enclosing
    # scope and we always want to use the same one throughout.
    return fibonacci_memo(n)
```
- While we take $O(n)$ time and $O(n)$ memory, its better than the $O(Fib(n))$ time that we had previously, since we now don't need crazy depths of recursion!

- **Memoization** works here because the `fib` function returns the same `n` for all given `n`. Keep that in mind that memos can only be used *where necessary*.

#### Dynamic Programming (DP)
- `fib` as a function has an interesting thing such that the result of `fib(n)` will *always* be the same for any `n`. `fib` is a *pure* function!
	- As such, `fib` has something called *optimal substructure*, where knowing the answer about smaller problems always tells you something definitive about the bigger ones
	
- Then, we can employ the strategy of *dynamic programming*, where instead of starting at the top and working our way down, we can start at the bottom and work our way up. 
	- We break down a large problem into smaller sub problems, the results are saved, and then the sub-problems are optimized to find the overall solution
	
- Let's implement `fib` dynamically:

```python
def fib(n):
	if n == 0:
		return 0
	elif n == 1:
		return 1
	else: 
		results = [0, 1]
		for i in range(2, n+1):
			results.append(results[-2] + results[-1])
		return results[-1] # now the answer sits on the end of the list! 
```
- Now what we did was this:
	- Rather than calculating a result recursively, by having `fibonacci(n)` call `fibonacci(n - 1)` and `fibonacci(n - 2)`, we can instead reverse the order of the calculations altogether, by first determining (and storing) `fibonacci(0)`, then `fibonacci(1)`, and so on. 
	- Each time we need to determine the next value in the sequence, we'll already have the results that allow us to calculate it. 
	- Consequently, there will be no need to make a recursive call at all, which means not only that we'll avoid the costs of making function calls altogether, but also that we'll avoid Python's recursion depth limit.

- As such, we have $O(n)$ memory and $O(n)$ time complexities for the list of size `n` and the `n` addition operations performed. 

- We can make the most *optimal* form of `fib` through DP through this final implementation, because `fib` makes a list but it doesn't necessarily need to. We don't need 1200 answers for `fib(1200)`, we just need one!
- Let's do that: 
```python 
def fib(n):
	if n == 0:
		return 0
	elif n == 1:
		return 1 
	else: 
		prev, cur = 0, 1
		for i in range(2, n+1):
			prev, cur = cur, (prev + cur)
		return cur 
```

- We've now reduced our memory footprint to _O_(1), because all we're storing are two integer variables, no matter how big `n` is. 
	- This, for `fibonacci`, is about as good as it gets for calculating it using a sequence of additions: linear time and constant memory. 

- Note, too, that not all dynamic programming solutions can be optimized this far — our ability to do it here depended on the quirk that each step in `fibonacci` only needed the previous two values, so all the others could be safely forgotten.
