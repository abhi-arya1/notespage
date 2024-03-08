---
title: Topic 5 - Asymptotic Analysis
date: 2024-03-08
---

> Full Notes (and Graphs) can be found here: [Asymptotic Analysis](https://ics.uci.edu/~thornton/ics33/Notes/AsymptoticAnalysis/)

Contents

- [[#Introduction]]
- [[#Big-O Notation]]
- [[#Let's set a standard: Closest Fit O-Notation]]
- [[#What about Measuring Memory?]]

### Introduction

> **Question** How do we build _insight_ into thinking about the complexity of code?

> **Answer** Asymptotic Analysis: "Thinking about how long it would take to solve a problem as it gets larger and larger"

---

Let's consider an "_algorithm_":

- `start` with your back against one side wall, facing the opposite wall.
- `while` you haven't reached the opposite wall:
  - take one `step` forward
- `end`

- We can make this a linear graph where the `x` axis is the _width of the room_ while the `y` axis is the _time it takes to cross_
  - This is a very very common shape of an algorithm, that increases _linearly_ in time based on a variable increasing

> This is called a **_Linear Time Algorithm_** (or _Linear Time Complexity_)

---

Let's consider another algorithm:

- `start` with your back against one side wall, facing the opposite wall.

  - `teleport` to the other side of the wall.
  - `end`

- If we graph this algorithm, it's a straight line where regardless of how far you go, it is just the time it takes to teleport.

> This is called a **_Constant Time Algorithm_**

So, in comparing these two algorithms, depending on the size of the room, we can pick the algorithm that works.

---

> Algorithm Time Complexities begin to show a pattern. **_They fit mathematical functions_**

- But there are a lot of small details here, so let's introduce a convention to fix that issue.
- There is more information on this section in the Detailed Notes at the Top: [[Topic 5 - Asymptotic Analysis and O-Notation#Introduction]]

### Big-O Notation

> **Definition:** We say that $f(n)$ is $O(g(n))$ if and only if there are two positive constants $c$ and $n_0$ such that $f(n) \leq cg(n)$ for all $n \geq n_0$

- Let's break that down

  - $f(n)$ is the _real_ function --> "How long does it take to sort", etc.
  - $g(n)$ is a _simpler_ function --> Simpler function with the same shape
  - The two conditions
    - $f(n) \leq cg(n)$ --> Constant factors don't matter!
      - The functions $n, 3n, 1234n$ all grow at the same rate, so it doesn't change.
    - $n \geq n_0$ --> Small problems don't matter!
      - In practice, it doesn't matter how you solve a small problem because it won't have much effect.

- So, let's fit a algorithm to a function:

  - Is $3n + 4$ fit to $O(n)$? (remember, $g(n)$ is now just $n$)
    - $f(n) \leq cg(n)$ for all $n \geq n_0$
    - Let the $c$ be 4, so now we get:
    - $3n+4 \leq 4n$
    - Simplify to get:
    - $4 \leq n$
    - Let $n_0=4$
    - $\forall_{n\geq4}(4 \leq n)$
      - This is true, so we just proved that the function does in fact fit $O(n)$!

- What if we had a case where $n=500$ and we had to pick between $O(n^2)$ and $O(n)$?
  - We don't know, because O-notation only measures _asymptotic_ behavior (i.e. growth toward infinity)
- What if we had two different algorithms with both $O(n^2)$? Again, we wouldn't know!

- What if we can establish that $3n$ is $O(n^2)$
  - $\forall_{n\geq n_0}(3n \leq cn^2)$
  - Let $c=3$ and $n_0=1$
  - Then we get:
  - $\forall_{n\geq 1}(3n \leq 3n^2)$
    - This _is_ a `True` statement, but does that make it _right_?
    - No.
    - Short version --> $O$ notation is about establishing an _upper bound shape_. It basically will say that it will _never_ get worse than $O(n^2)$
    - So, $3n$ also falls under $O(n_3), O(n_4), O(2_n)$, and $O(n!)$

#### Let's set a standard: Closest Fit O-Notation

- "Closest Fit O-Notation"
- If we had a function $3n^2 + 4n + 5$, there are 10+ choices (again, check the website for full story)

- Let's be _explicit_ about functions that are _correct_ for Closest-Fit O-Notation though:
- _Here are our 3 conditions_:

- **It has no constant coefficients (e.g., we'll never say $2n$ when we could instead say $n$\_).**
- **It has no lower-order terms (e.g., we'll never say $n^2 + n$ when we could instead say $n^2$).**
- \*\*Of the remaining functions with the first two characteristics, it is the slowest growing (e.g., we'll never say $n^3$ when we could instead say $n^2$).
- **If something doesn't grow, then we deem it as $O(1)$ complexity**

### What about Measuring Memory?

- Once time is used, then time is lost. Memory isn't lost, but its still important to allocate it properly and find the _cost_ of memory.

- Let's step back and ask a question
  > In a list `[1, 2, 3, 4]`, how are items actually _stored_?
- Why can we ask for negative indices, grow without restriction, etc?
  - In Python, each object is stored as a _reference_
  - As such, each reference points to many more references. i.e. a list of _n_ objects has _n_ references.

> Let's answer the question: "How much memory is required to store a list of $n$ 3-digit integers?"

- Let's deem the length of as a space in memory of a list as $a$
- Let's deem the reference to the list's list of references as $b$
- And finally, let's deem the contents of the list as $n$ references from the $b$ reference, so we get $bn$
- As such, we sum those to get the Memory Complexity of a list as

  - $a+b+bn$

- We can _wipe out_ constants and coefficients in closest-fit O-notation as mentioned above, so $a+b+bn$ turns into $O(n)$

- What is a constant amount of memory added to a linear amount of memory? *O*(1) + *O*(_n_) = *O*(_n_). (Why? Because whatever function *O*(1) represents, it's a lower-order term in whatever function *O*(_n_) represents.)

- That's that for $O$ notation. Watch a couple youtube videos if you are still confused.
