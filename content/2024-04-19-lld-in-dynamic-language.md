---
title: Object Oriented Design in a Dynamically Typed Language
date: 2024-04-19
categories: [Software Engineering, Low Level Design]
tags: [software-design, low-level-design]
---

Java is often the "go-to" for practicing Low Level Design (LLD) because its strong typing just makes things feel... safer.

I've tried solving LLD problems in both Python and Java, and honestly? Python is *clearly harder*. Contrary to what you might expect, you don't actually save that many lines of code compared to Java. Plus, you lose that sweet type safety and debugging becomes a bit more of a headache. 

That said, Python is versatile. Here are some tricks I've picked up while grinding LLD in Python.

## Thread Safety

Python isn't thread-safe by default. If you've coming from Java and miss the `synchronized` keyword, you'll have to get familiar with the `threading` module.

Here is how I usually implement a `@synchronized` decorator that mimics Java’s keyword using a `threading.Lock`:

```python
import threading

def synchronized(lock):
    def decorator(func):
        def wrapper(*args, **kwargs):
            with lock:
                return func(*args, **kwargs)
        return wrapper
    return decorator

# Usage
lock = threading.Lock()

@synchronized(lock)
def thread_safe_method():
    # Critical section
    pass
```

## Property Decorators for Getters and Setters

Python doesn't have private variables in the way Java does. A single underscore `_` is just a convention saying "hey, don't touch this from outside the class." For LLD interviews, encapsulation is key, so you’ll want to use `@property`.

```python
class Person:
    def __init__(self, first_name, last_name):
        self._first_name = first_name
        self._last_name = last_name
        
    @property
    def first_name(self):
        return self._first_name

    @first_name.setter
    def first_name(self, value):
        self._first_name = value
```

## Custom Exceptions

Generally in Java, you'd extend `Exception`. In Python, it’s pretty much the same.

```python
class ParkingLotException(Exception):
    def __init__(self, message: str):
        self.message = message
        super().__init__(self.message)
```

## Abstract Classes and Interfaces

Interfaces aren't strictly enforced in Python, which can lead to messy code if you're not careful. This is where the `abc` (Abstract Base Classes) module comes in.
