---
title: Topic 17 - Class Design
date: 2024-03-11
---

> Full notes can be found [here](https://ics.uci.edu/~thornton/ics33/Notes/ClassDesign/)

Contents
- [[#Introduction]]
- [[#The Problem]]
	- [[#What's the simplest thing that could possibly work?]]
	- [[#Refactor 1 -> NamedTuple to Class]]
	- [[#Refactor 2 -> Return to Immutability]]
### Introduction 
- When we write something in python, there's a couple considerations to make 
	1. What do I want to be able to say?
	2. What do I want to *prevent myself* from being able to say?
- New developers struggle with both of these, because its harder to tell when an implemented solution is "sub-optimal" so to speak. 
- But as long as we make ourselves aware of questions such as an analysis of the readability and structure of our code, we can gain experience and in turn, the quality of our decisions will naturally improve. 
- So, let's go on a journey where we start with a basic class, and work toward gradual redefinitions of its design until we optimize, keeping the two mantras in mind: 
	- What do we want to say?
	- What do we prevent ourselves from saying?

### The Problem 
- So let's consider a problem with a set of guidelines: 
	1. We want to be able to store two pieces of information about people: Their names and birthdates
	2. There are many people, and we'd like to be able to distinguish between them.
	3. We don't want the information about a person to be changed once it's been specified. 
	
##### What's the simplest thing that could possibly work?
- Let's see:
```python 
from datetime import date
from collections import namedtuple 

Person = 	- `Person = namedtuple('Person', ['name', 'birthdate']`
p = Person('Alex', date(2005, 11, 1))
```
- Great! We can create and do basic interactions with Person objects. 
- This is truly the simplest way to achieve this, but how do we *feel* about it?
	- Since these are named tuples, we can obtain but can't change their attributes, and we can compare for equivalence of each element through the base class. 
	- We can also hash people, since we need some distinguishability!

- Issues that arise now: 
	1. We got what we wanted, but now we can't add methods 
	2. We can't specify whether we want a string and a date, because namedtuples don't restrict inputs, so we didn't answer *what do we prevent ourselves from saying*

##### Refactor 1 -> NamedTuple to Class 
```python
class Person:
    def __init__(self, name, birthdate):
        self.name = name
        self.birthdate = birthdate


    def age(self, as_of_date):
        if self.birthdate > as_of_date:
            raise ValueError(f'Person was not born yet on {as_of_date}')

        years_old = as_of_date.year - self.birthdate.year

        if (self.birthdate.month, self.birthdate.day) >= (as_of_date.month, as_of_date.day):
            years_old -= 1

        return years_old


    def __eq__(self, other):
        return isinstance(other, Person) and \
               (self.name, self.birthdate) == (other.name, other.birthdate)


    def __hash__(self):
        return hash((self.name, self.birthdate))
```

- Oh no! We lost immutability, because we can directly access class attributes and change them! Namedtuples enforce that. Classes, on the other hand, don't!

##### Refactor 2 -> Return to Immutability 



