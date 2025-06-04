# Python

## Table Of Contents

- [Descriptors](#descriptors)
- [Property and CachedProperty](#property-and-cachedproperty)
- [Context Manager](#context-manager)
- [Multithreading](#multithreading)


## Descriptors

Descriptors in Python provide a powerful way to customize the behavior of attribute access. A descriptor is simply a class that defines one or more of the special methods: __get__, __set__, and __delete__.

These methods are automatically triggered when the descriptor is used as a class-level attribute, allowing us to control what happens when an attribute is retrieved, set, or deleted on an instance.

Descriptors are commonly used for purposes like data validation, logging, computed attributes, or transformation (e.g., converting a value to uppercase).

Python’s built-in @property decorator is actually a simplified way of using descriptors behind the scenes. If we need more control, we can implement custom descriptors ourselves."**


```
__get__(self, instance, owner)

__set__(self, instance, value)

__delete__(self, instance)

```

 ### Example :

```
class UpperCase:
    def __get__(self, instance, owner):
        print("Getting value...")
        return instance._name

    def __set__(self, instance, value):
        print("Setting value...")
        instance._name = value.upper()

    def __delete__(self, instance):
        print("Deleting value...")
        del instance._name


class Person:
    name = UpperCase()

    def __init__(self, name):
        self.name = name


# Usage
p = Person("naresh")

print(p.name)    # Triggers __get__

p.name = "kakarla"  # Triggers __set__

print(p.name)    # Triggers __get__

del p.name       # Triggers __delete__

# Accessing after deletion would raise an AttributeError
# print(p.name)  # Uncommenting this line will raise an error

```

## Property and CachedProperty

**@property** is used in Python to make a method act like an attribute.
It helps us access private variables in a safe way using getter, setter, and deleter methods.
When we use @property, the value is calculated every time we access it — so it's always fresh.

**@cached_property** is similar, but it stores (or caches) the value after the first time it's calculated.
That means the function runs only once, and later it gives the saved result without recalculating.
It's helpful when the value takes time to calculate and doesn't change often (like a database query or a complex calculation).

Both @property and @cached_property are built using a Python concept called descriptors, which control how getting/setting values works in the background.

```

from functools import cached_property

class Employee:
    def __init__(self, name, salary):
        self._name = name
        self.salary = salary

    # --- @property with getter, setter, deleter ---
    @property
    def name(self):
        print("Getting name...")
        return self._name

    @name.setter
    def name(self, value):
        print("Setting name...")
        self._name = value.strip().title()

    @name.deleter
    def name(self):
        print("Deleting name...")
        del self._name

    # --- @cached_property ---
    @cached_property
    def yearly_bonus(self):
        print("Calculating bonus once...")
        return self.salary * 0.10


emp = Employee("naresh", 50000)

print(emp.name)          # Calls getter
emp.name = " kakarla "   # Calls setter
print(emp.name)          # Getter again

print(emp.yearly_bonus)  # Calculates once
print(emp.yearly_bonus)  # Uses cached value

del emp.name             # Calls deleter

```

## Context Manager

Context Manager in Python is an object that automatically handles setup and cleanup actions around a block of code. It is most often used with the with keyword to ensure resources like files, thread locks, database connections, or network connections are properly opened and then closed or released, even if errors occur.

A context manager class typically implements two special methods:

- __enter__: runs setup code and returns a resource (like a file handle or lock)

- __exit__: runs cleanup code automatically after the with block finishes, handling exceptions if any

Using context managers helps write cleaner and safer code by managing resource lifecycles automatically.

```
class MyContextManager:
    def __enter__(self):
        print("Setup: Resource is ready")
        return self  # Optionally return a resource or self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Cleanup: Resource is released")
        # Handle exceptions if needed, return True to suppress
        if exc_type:
            print(f"Exception caught: {exc_type}")
        return False  # Propagate exceptions if any

# Usage
with MyContextManager() as mgr:
    print("Inside the context block")
    # You can raise an exception here to test __exit__

```

## Multithreading


Multithreading is a way to run multiple threads (smaller units of tasks) concurrently within a single process. This helps to perform multiple operations seemingly at the same time.

There are two types of multithreading approaches:

- `Without synchronization`: Threads access shared data freely, which can cause race conditions.

- `With synchronization (Locks)`: We use locks to ensure only one thread accesses the shared resource at a time, preventing data corruption.

**Python’s Limitation: GIL (Global Interpreter Lock)**

- Python’s standard interpreter CPython has a Global Interpreter Lock (GIL), which allows only one thread to execute Python bytecode at a time.

- This prevents true parallel execution of threads for CPU-bound tasks (like heavy computations).

- But for I/O-bound tasks (like network or file operations), threads spend a lot of time waiting, so Python threads can still improve performance by switching while waiting.

**Example Without Synchronization**
```
import threading
import time

def task(name):
    print(f"{name}")
    time.sleep(2)

thread1 = threading.Thread(target=task, args=('Naresh',))
thread2 = threading.Thread(target=task, args=('Nishi',))

thread1.start()

thread2.start()

thread1.join()
thread2.join()


counter = 0

def increment():
    global counter

    for _ in range(100000):
        counter +=1

threads = [threading.Thread(target=increment) for _ in range(10)]

for thread in threads:
    thread.start()

for thread in threads:
    thread.join()


print("counter", counter)

```

**Example with Lock**
```
import threading

lock = threading.Lock()
counter = 0

def increment():
    global counter

    for _ in range(100000):
        with lock:
            counter +=1

threads = [threading.Thread(target=increment) for _ in range(10)]

for thread in threads:
    thread.start()

for thread in threads:
    thread.join()

print("counter", counter)

```

**Example CPU Intensive**

```
import threading

total = 0

def compute():
    global total
    for _ in range(10**9):
        total +=1

threads = [threading.Thread(target=compute) for _ in range(5)]

for thread in threads: thread.start()
for thread in threads: thread.join()

```




















