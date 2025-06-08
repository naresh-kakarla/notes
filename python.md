# Python

## Table Of Contents

- [Descriptors](#descriptors)
- [Property and CachedProperty](#property-and-cachedproperty)
- [Context Manager](#context-manager)
- [Multithreading](#multithreading)
- [Thread Deadlock](#thread-deadlock)
- [Closure and Scope](#closure-and-scope)
- [Decorator](#decorator)
- Generators and Iterators(#generators-and-iterators)
  

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

## Thread Deadlock

Thread deadlock is a situation where two or more threads are waiting indefinitely for resources (usually locks) held by each other — causing all of them to halt forever.

**Example**

- Thread A locks resource 1 and waits for resource 2.

- Thread B locks resource 2 and waits for resource 1.
   Neither thread can proceed — they’re stuck forever. That’s a deadlock.


**When Does Deadlock Happen?**

1. Manual Locking Without Release -> Forgetting to call lock.release() after lock.acquire().
   ```
   lock = threading.Lock()

   def task():
    lock.acquire()
    # forgot lock.release()

   ```

2. Multiple Locks in Opposite Order -> One thread acquires lock1 → lock2, another thread acquires lock2 → lock1.

   ```
   lock1 = threading.Lock()
   lock2 = threading.Lock()
   
   def thread1():
       with lock1:
           time.sleep(1)
           with lock2:
               pass
   
   def thread2():
       with lock2:
           time.sleep(1)
           with lock1:
               pass

   ```
   Thread 1 locks lock1 and waits for lock2
   Thread 2 locks lock2 and waits for lock1
   Both wait forever = Deadlock

3. Circular Wait -> Each thread is holding one lock and waiting for another — leading to a wait cycle.
   

**How to Prevent Thread Deadlocks**

**1. Use with Statement (Context Manager)**

Automatically handles acquiring and releasing locks.

```
with lock:
    # safe block

```

**2. Acquire Locks in a Fixed Global Order**

If multiple locks are needed, acquire them in the same order (e.g., lock1 → lock2) across all threads.

**3. Use acquire(timeout=...)**
Avoid waiting forever:

```
if lock.acquire(timeout=1):
    try:
        # safe work
    finally:
        lock.release()

```

**4. Use Higher-Level Tools**
- threading.RLock() – when the same thread may need to acquire the same lock multiple times.

- threading.Semaphore() – limits concurrent access.

- queue.Queue() – thread-safe data sharing without locking.

## Closure and Scope

**Closure** 
- A function is defined inside another function (an inner function).

- The outer function returns the inner function.

- The inner function remembers the variables from the outer function even after the outer function has finished execution.

A closure means a function "remembers its environment" where it was created — not just the code but also the variables around it.

**Example:**

```
def outer(msg):
    def inner():
        print("Message:", msg)  # msg comes from outer
    return inner

func = outer("Hello")
func()  # Output: Message: Hello

```

Even though outer is done, inner() remembers msg = "Hello" — that's a closure.

**Scope (LEGB Rule)**

Python uses LEGB rule to resolve variable names:

|Scope   Level |	Meaning | Example|
|--------------|	------- |--------|
|L|	Local|	Inside current function|
|E| Enclosing	|In enclosing function (if nested)|
|G|Global|	At the top level of the module
|B|Built-in|	Keywords like len, print|


## Decorator

Decorator in Python is a function that takes another function as input, adds some additional behavior before and after the original function, and return the new function - without changing the original function code.

It’s commonly used for logging, authentication, performance tracking, etc.

**Definatio**
```
def my_decorator(func):             # Step 1: Accept a function
    def wrapper(*args, **kwargs):   # Step 2: Define a wrapper
        print("Before Execution")
        result = func(*args, **kwargs)  # Step 3: Call original function
        print("After Execution")
        return result
    return wrapper                  # Step 4: Return the wrapper
```

**Usage**
```
@my_decorator
def original_fun(a, b):
    return a + b

print(original_fun(10, 5))
```
### Decorator with Parameters

Normally, a decorator takes a function as an argument.
But when we want to pass extra arguments to the decorator (like config or flag) we need to add extra outer function layer.

**Structure:**
```
def decorator_with_args(arg1, arg2):
    def actual_decorator(func):
        def wrapper(*args, **kwargs):
            # use arg1, arg2 inside
            return func(*args, **kwargs)
        return wrapper
    return actual_decorator
```

**Example:**

```
def log(level):
    def decorator(func):
        def wrapper(*args, **kwargs):
            print(f"[{level}] - Executing {func.__name__}")
            result = func(*args, **kwargs)
            print(f"[{level}] - Done executing {func.__name__}")
            return result
        return wrapper
    return decorator

@log("INFO")
def say_hello(name):
    print(f"Hello, {name}")

@log("DEBUG")
def add(a, b):
    return a + b

```

### Class-Based Decorator:

A class-based decorator is a class with a __call__ method, which lets it wrap a function and run extra code before or after the function runs. It’s useful when we want to track state or make decorators that need more structure.

```
class MyDecorator:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("Before function call")
        result = self.func(*args, **kwargs)
        print("After function call")
        return result

@MyDecorator
def greet(name):
    print(f"Hello, {name}!")

greet("Name")

```

**Class-Based Decorator with Parameters**
```
class Repeat:
    def __init__(self, times):
        self.times = times

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            for _ in range(self.times):
                func(*args, **kwargs)
        return wrapper

@Repeat(3)
def laugh():
    print("Ha ha!")

laugh()
```

## Generators and Iterators

### Iterators: 
An iterator is an object in Python that implements:
 - __iter__() `->` returns the iterator object itself
 - __next__() `->` returns the next value or raises StopIteration
   
It allows traversing through all the elements of a collection one at a time. Calling next() on an iterator returns the next item, and raises a StopIteration exception when there are no more items. Iterators can be used explicitly or implicitly in loops.

```
my_list = [10, 20, 30]
it = iter(my_list)       # Get iterator from list

print(next(it))          # Output: 10
print(next(it))          # Output: 20
print(next(it))          # Output: 30
# next(it) now would raise StopIteration

```

**Custom Iterator:**
```
class MyCounter:
    def __init__(self, max_value):
        self.max_value = max_value
        self.current = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current < self.max_value:
            value = self.current
            self.current +=1
            return value
        else:
            raise StopIteration

counter = MyCounter(3)

iterator = iter(counter)
print(next(iterator))


for value in MyCounter(3):
    print(value)
```
    

### Generators:

A generator is a simple way to create an iterator using a function with yield. Instead of giving all values at once, it gives one value at a time and remembers where it left off. This saves memory, especially for big data.

When the function runs, it pauses at each yield and resumes next time it’s called.

```
def count_up_to(n):
    i = 1
    while i <= n:
        yield i            # Yield value and pause
        i += 1

gen = count_up_to(3)

print(next(gen))          # Output: 1
print(next(gen))          # Output: 2
print(next(gen))          # Output: 3
# next(gen) now raises StopIteration
```



































