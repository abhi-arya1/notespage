---
title: Topic 6 - Searching
date: 2024-03-08
---

> The problem of finding an object in a data structure is known as **_searching_**
> -> The object that is being looked for is called a **_search key_**
> -> The algorithm that solves such a problem is a **_search algorithm_**

- Let's dive into the semantics of this:

  - A list in python has a reference to the block of memory that stores references to each list element. $n$ objects in the list implies $n$ references.
  - A integer indicating the list's _size_
  - A integer indicating how many elements can be stored in the list
    - If the list needs to grow beyond this, at least in python, memory will be allocated behind the scenes

- So when we make `x = [13, 18, 11, 7]`, we'll get this:
  ![[Screenshot 2024-01-23 at 5.35.09 PM.png]]

- Note that list elements are stored _contiguously_, such that each reference to an element follows a reference to a previous one.
- All references in Python are the same size when stored in memory. (Objects are not, but references are.) Consequently, Python can multiply the index *i* by the size of a reference, then add that to the address where the list's elements begin. That will always be the address containing a reference to the desired element. This, too, can always be done in constant time (i.e., without searching). No matter what the index is, it's a matter of a multiplication and an addition; no more, no less.
- Negative indexing doesn't change this outcome all that much, because Python can convert a negative index like `-3` into a non-negative one by adding it to the list's size, which can be done in constant time, as well.
- The primary consequence of this design is that we can always find an element in constant time given its index (i.e., as long as we know where to look). Sadly, we won't always be so fortunate; sometimes it's not clear where we should be looking, which is why we need a search algorithm.

> So, let's go through search algorithms and dive deep into the semantics of them

### Sequential (or Linear) Search

- We can implement this in a very simple manner:

```python
def sequential_search(items: list[any], key: any) -> int | None:
	for index in range(len(items)):
		if items[index] == key:
			return items[index]
	return None
```

- So what does this algorithm _cost_?
  - There are two ways to measure: Memory and Time

> Memory

- In terms of memory, the list is already a given, and we can assume for now that the time taken is a given, so what is the memory cost?
  - `index`, and `range --> start, stop, step`
    - Range generates 3 variables and makes them on the fly, which will come up later this quarter.
  - As such, we really only have 4 variables with $O(1)$ Memory Complexity.
- As such, the memory usage of this algorithm is simply $O(1)$, or _Constant Memory Complexity_

> Time

- The time is a bit harder to determine than the complexity.
- We could say that the problem is $O(n)$, but it's a general estimate. By this logic, $O(n^2)$ is also a valid complexity!

- Lets look at cases:

  1.  Best Case: The first element in the list is the `key`: $O(1)$
  2.  Worst Case: The last element in the list is the `key`: $O(n)$
      - Since the last element is found, we used all $n$ "iterations" to find the element.

  - Average-Case Time?
    - _Technically_ the average case is $O(\frac{n}{2})$, but constant factors are not permissible in asymptotic analysis, because it allows us to disregard them.
    - So our average-case time is of $O(n)$ complexity
      - Remember, this is not the same as the Worst Case because these are two _different_ functions of the same _linear-time_ shape.
      - It's essentially a linear-time algorithm until you get lucky.

- Based on this, what if there was a _faster_ way to search for something? Something that doesn't consistently fall in a shape of $O(n)$?

### Binary Search

- What if we have a list `[1, 2, 5, 6, 10, 20, 23, 1761, ...]`
  - We must assume that it is sorted
- In sequential search, we search all elements.
- In binary search, we can pick a _pivot_ and search the left and right and keep cutting that into smaller and smaller pieces until we get the key itself.

- Let's analyze Binary Search

> Time

- To consider the worst-case time required to perform a binary search, let's think about how the time required changes as *n* grows.

  - When there is one element, we'll only ever need to visit the one element.
  - When there are two elements, we might need to visit both of them.
  - When there are four elements, we'll visit one element, and then have no more than two of them left to search. We know already that, at worst, we'll visit both of those. So, in total, we might visit three elements.
  - When there are eight elements, we'll visit one element, and then have no more than four of them left to search. We know already that, at worst, we'll need to visit three elements when searching four elements. So, in total, we might visit four of them.
  - When there are sixteen elements, we'll visit one element, and then have no more than eight of them left to search. We know already that, at worst, we'll need to visit four elements when searching eight elements. So, in total, we might look at five of them.

- So we consider the number of visited elements $v$ in relation to the length of the list $n$.
  - We'd find that it follows a formula similar to this, since visiting one additional element doubles the number of elements we can search:
    - $n = 2^v$
  - We can then get $log_2n = v$
- Thus, at worst, if we have $n$ elements to search, we'll visit $log_2n$ of them. Since this takes a constant amount of time, the O-notation becomes:

- $O(log(n))$

- For Logarithmic Memory Analysis, go here: https://ics.uci.edu/~thornton/ics33/Notes/Searching/

- At the end of the day, we don't always use Binary Search because sometimes Sequential Search fits our use-case better. There is a finale in the notes about this.
