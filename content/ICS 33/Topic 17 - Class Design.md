---
title: Topic 17 - Class Design
date: 2024-03-08
---

> Full notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/ClassDesign/)

### Introduction

- When we write something in python, there's a couple considerations to make

  1.  What do I want to be able to say?
  2.  What do I want to _prevent myself_ from being able to say?

- So let's consider a problem with a set of guidelines:
  1.  We want to be able to store two pieces of information about people: Their names and birthdates
  2.  There are many people, and we'd like to be able to distinguish between them.
  3.  We don't want the information about a person to be changed once it's been specified.
- What's the simplest thing that could possibly work?
  - `Person = namedtuple('Person', ['name', 'birthdate']`
  - This is truly the simplest way to achieve this, but how do we _feel_ about it?
- Let's see:

```python
import datetime

p = Person('Alex', datetime.date(2005, 11, 1))

```
