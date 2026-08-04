# Python Essentials 2 
### Module 3: Object-Oriented Programming (OOP)

---

## 1. Core Concepts

| Term | Meaning |
|---|---|
| **Class** | A blueprint/recipe — defines properties + methods |
| **Object** | An instance created from a class (the actual "thing") |
| **Instantiation** | The act of creating an object from a class |
| **Property / Attribute** | A piece of data belonging to an object (noun/adjective) |
| **Method** | A function that belongs to a class (verb) |
| **Superclass** | The more general, "parent" class |
| **Subclass** | The more specific class, derived from a superclass |
| **Inheritance** | Subclass automatically gets superclass's properties/methods |
| **Encapsulation** | Hiding data so it can't be accessed/modified directly |
| **Polymorphism** | Subclass can redefine (override) inherited behavior |
| **Composition** | Building a class using other objects as building blocks |

```python
class TheSimplestClass:
    pass                       # empty class — no properties/methods

my_object = TheSimplestClass() # instantiation — class used like a function
```

> 💡 Naming hint: **noun** → object name, **adjective** → property, **verb** → method.
> e.g. "A pink Cadillac went quickly" → name=Cadillac, property=color(pink), method=go(quickly)

### 🔐 SOC relevance
```python
class Alert:
    pass

incoming_alert = Alert()   # every SIEM alert, log entry, or asset in your
                            # inventory is naturally an "object" — OOP maps
                            # directly onto how security data is modeled
```

---

## 2. Procedural vs Object Approach (Stack Example)

| | Procedural | Object-Oriented |
|---|---|---|
| Data | global list, unprotected | hidden inside object (`__stack_list`) |
| Reuse | copy-paste for a 2nd stack | just create another object |
| Extending | rewrite functions | inherit + add new methods |

```python
# ---- PROCEDURAL ----
stack = []
def push(val):
    stack.append(val)
def pop():
    val = stack[-1]
    del stack[-1]
    return val
```

```python
# ---- OBJECT-ORIENTED ----
class Stack:
    def __init__(self):        # constructor — runs automatically on creation
        self.__stack_list = [] # __ prefix = private (name-mangled)

    def push(self, val):
        self.__stack_list.append(val)

    def pop(self):
        val = self.__stack_list[-1]
        del self.__stack_list[-1]
        return val

s1 = Stack()   # each object gets its OWN private __stack_list
s2 = Stack()
s1.push(1)
s2.push(99)
print(s1.pop(), s2.pop())   # 1 99 — independent, no interference
```

### 🔐 SOC relevance
```python
class AlertQueue:
    def __init__(self):
        self.__queue = []          # analysts can't corrupt this directly
    def push(self, alert):
        self.__queue.append(alert)
    def pop(self):
        return self.__queue.pop()

soc1_queue = AlertQueue()   # one queue per shift/analyst — no shared-state bugs
soc2_queue = AlertQueue()
```

---

## 3. Constructors, `self`, and Private Members

| Concept | Rule |
|---|---|
| Constructor name | always `__init__` |
| First parameter | always `self` (convention, not enforced by name but required in position) |
| `self` | represents "this object" — Python passes it automatically |
| Calling a method | `obj.method(x)` → Python auto-fills `self`, you only pass `x` |
| `_name` | single underscore = "internal use" convention only (not enforced) |
| `__name` | double underscore = **name-mangled**, becomes `_ClassName__name` |

```python
class Classy:
    def method(self, par):
        print("method:", par)

obj = Classy()
obj.method(1)          # self is filled in automatically -> prints "method: 1"
```

```python
class Classy:
    def __hidden(self):
        print("hidden")

obj = Classy()
# obj.__hidden()             # ❌ AttributeError
obj._Classy__hidden()        # ✅ still reachable via mangled name — "hidden"
```

> ⚠️ Private in Python = **convention + name-mangling**, NOT true access control like Java/C++.

### 🔐 SOC relevance
```python
class Credential:
    def __init__(self, user, secret):
        self.user = user
        self.__secret = secret     # mangled, not printed/logged by accident
    def __str__(self):
        return f"user={self.user}"  # secret never exposed in __str__
```
Name-mangling isn't real security, but it signals intent — same principle as marking a field "sensitive" before it ever reaches a log line.

---

## 4. Instance Variables vs Class Variables

| | Instance Variable | Class Variable |
|---|---|---|
| Belongs to | one specific object | the class itself (shared) |
| Defined | `self.x = val` (usually in `__init__`) | `x = val` directly inside class body |
| Storage | object's own `__dict__` | class's own `__dict__` |
| Changing it | affects only that object | affects **all** objects (single shared copy) |

```python
class ExampleClass:
    counter = 0                 # CLASS variable — one shared copy

    def __init__(self, val=1):
        self.first = val        # INSTANCE variable — unique per object
        ExampleClass.counter += 1   # update the shared counter

a = ExampleClass()
b = ExampleClass(2)
print(a.__dict__)      # {'first': 1}        <- no 'counter' here!
print(a.counter, b.counter)   # 2 2   <- same shared value for both
```

```python
# Adding a property on the fly (fully legal in Python)
c = ExampleClass(4)
c.third = 5
print(c.__dict__)      # {'first': 4, 'third': 5}
```

### `hasattr()` — safely check if a property/method exists
```python
class ExampleClass:
    def __init__(self, val):
        if val % 2:
            self.a = 1
        else:
            self.b = 1

obj = ExampleClass(1)
print(hasattr(obj, 'a'))   # True
print(hasattr(obj, 'b'))   # False — avoids AttributeError crash
```

### `getattr()` / `setattr()` — read/write a property by name (string)
```python
val = getattr(obj, 'a')        # same as obj.a
setattr(obj, 'a', val + 1)     # same as obj.a = val + 1
```

### 🔐 SOC relevance
```python
class Host:
    total_hosts = 0             # class variable — global asset counter
    def __init__(self, ip):
        self.ip = ip            # instance variable — unique per host
        Host.total_hosts += 1

h1, h2 = Host("10.0.0.1"), Host("10.0.0.2")
print(Host.total_hosts)         # 2 — auto-tracks inventory size without a separate counter
```
`hasattr()`/`getattr()` are handy when parsing inconsistent alert JSON — some fields may or may not exist depending on the log source.

---

## 5. Methods & Introspection

| Built-in attribute | Type | Exists on | Meaning |
|---|---|---|---|
| `__dict__` | dict | object AND class (separately) | all properties currently set |
| `__name__` | string | **class only** (not object) | class's name |
| `__module__` | string | class | module where class was defined |
| `__bases__` | tuple | class only | direct superclasses |

```python
class Classy:
    pass

print(Classy.__name__)     # 'Classy'
print(Classy.__module__)   # '__main__'
print(Classy.__bases__)    # (<class 'object'>,) — every class implicitly extends object
```

```python
obj = Classy()
print(type(obj))            # <class '__main__.Classy'> — find an object's class
```

- **Introspection** = examining an object's type/properties at runtime.
- **Reflection** = introspection **+** the ability to modify it at runtime (`setattr`).

```python
# example: increment every integer property starting with "i"
def inc_i_ints(obj):
    for name in obj.__dict__.keys():
        if name.startswith('i'):
            val = getattr(obj, name)
            if isinstance(val, int):
                setattr(obj, name, val + 1)
```

### 🔐 SOC relevance
```python
def redact_object(obj):
    for name in list(obj.__dict__.keys()):
        if "password" in name or "secret" in name:
            setattr(obj, name, "***REDACTED***")
```
Reflection lets a logging/sanitization utility scrub sensitive fields off **any** object generically, without hard-coding every class's field names.

---

## 6. Inheritance

```python
class Vehicle:
    pass

class LandVehicle(Vehicle):        # LandVehicle IS-A Vehicle
    pass

class TrackedVehicle(LandVehicle): # inherits from LandVehicle -> also from Vehicle
    pass
```

### Checking relationships
| Function | Checks | Example |
|---|---|---|
| `issubclass(A, B)` | is class A a subclass of B? | `issubclass(TrackedVehicle, Vehicle)` → `True` |
| `isinstance(obj, Class)` | is obj an instance of Class (or its subclass)? | `isinstance(tv, Vehicle)` → `True` |
| `a is b` | do two variables point to the **same object** (not just equal value)? | see below |

```python
x = Classy()
y = Classy()
z = x
print(x is y)   # False — different objects
print(x is z)   # True  — same object, just 2 names for it
print(x == y)   # depends on __eq__; by default same as "is"
```

### Calling the superclass constructor
```python
class Super:
    def __init__(self, name):
        self.name = name
    def __str__(self):
        return "My name is " + self.name

class Sub(Super):
    def __init__(self, name):
        super().__init__(name)      # ✅ preferred — no need to name superclass
        # Super.__init__(self, name)  # also valid, explicit form

s = Sub("Andy")
print(s)   # "My name is Andy" — __str__ inherited from Super
```

### Property/method lookup order
Python searches (in this order):
1. the object itself
2. its class, then superclasses, **bottom → top**
3. left → right if multiple superclasses at the same level
4. not found anywhere → `AttributeError`

### Overriding (polymorphism)
```python
class Vehicle:
    def turn(self):
        self.change_direction()      # "abstract" — no implementation here
    def change_direction(self):
        pass

class TrackedVehicle(Vehicle):
    def change_direction(self):      # overrides the empty method
        print("stopping one track")

class WheeledVehicle(Vehicle):
    def change_direction(self):
        print("turning front wheels")
```
> Same method name, different behavior per subclass = **polymorphism**.
> A redefined method that changes superclass behavior is called **virtual**.

### Multiple inheritance
```python
class SuperA:
    def fun_a(self): return "A"

class SuperB:
    def fun_b(self): return "B"

class Sub(SuperA, SuperB):    # inherits from BOTH
    pass

obj = Sub()
print(obj.fun_a(), obj.fun_b())   # A B
```
- If both superclasses define the **same name**, the **left-most** one wins (MRO).
- **MRO (Method Resolution Order)** = the strategy Python uses to decide which class to check first. Must be *consistent* — Python raises `TypeError` if the inheritance order is contradictory.
- **Diamond problem**: `D(B, C)` where both `B` and `C` inherit from `A` — Python resolves it left-to-right without ambiguity, but keep hierarchies simple to avoid confusion.

⚠️ **Guidance**: prefer **single inheritance**. Multiple inheritance is powerful but risky (ambiguous overrides, harder to maintain) — **composition** is often a safer alternative.

### Composition (alternative to inheritance)
```python
class Tracks:
    def turn(self): print("turn via tracks")

class Wheels:
    def turn(self): print("turn via wheels")

class Vehicle:
    def __init__(self, controller):
        self.controller = controller   # "has-a" relationship, not "is-a"
    def turn(self):
        self.controller.turn()

car = Vehicle(Wheels())
tank = Vehicle(Tracks())
car.turn()    # turn via wheels
tank.turn()   # turn via tracks
```

### 🔐 SOC relevance
```python
class LogSource:
    def parse(self, raw):
        raise NotImplementedError   # abstract method — must be overridden

class FirewallLog(LogSource):
    def parse(self, raw):
        return {"src_ip": raw.split()[0]}   # concrete implementation

class WebLog(LogSource):
    def parse(self, raw):
        return {"status": raw.split()[-1]}

def process(source: LogSource, raw):
    return source.parse(raw)   # works identically for ANY subclass — polymorphism
```
This is exactly how real SOC/SIEM pipelines are built: one common interface (`LogSource`), a different subclass per log vendor — new log sources just plug in as new subclasses, no changes to `process()`.

---

## 7. Exceptions as Objects (OOP view)

### `try / except / else / finally`
```python
try:
    result = 1 / x
except ZeroDivisionError:
    print("division failed")
else:
    print("no error happened")   # runs ONLY if try succeeded, no exception
finally:
    print("always runs")         # runs no matter what (even after return/exception)
```

| Clause | Runs when |
|---|---|
| `except` | an exception was raised & matched |
| `else` | **no** exception was raised |
| `finally` | **always** — success, failure, or even a `return` |

### Catching the exception object
```python
try:
    raise ValueError("bad input")
except ValueError as e:
    print(e)          # "bad input" — uses the exception's __str__()
    print(e.args)      # ('bad input',) — tuple of constructor args
```

### Custom exceptions
```python
class PizzaError(Exception):                  # derive from a built-in exception
    def __init__(self, pizza, message):
        super().__init__(message)              # pass message up to Exception
        self.pizza = pizza                     # store your own extra data

class TooMuchCheeseError(PizzaError):           # can build a sub-hierarchy too
    def __init__(self, pizza, cheese, message):
        super().__init__(pizza, message)
        self.cheese = cheese

try:
    raise TooMuchCheeseError("mafia", 120, "too much cheese!")
except TooMuchCheeseError as e:
    print(e, ":", e.cheese)   # too much cheese! : 120
```
> Rule: catch **specific** exceptions before **general** ones (same as Module 2) — order still matters here.

### 🔐 SOC relevance
```python
class SOCError(Exception):
    """Base class for all custom SOC pipeline errors."""
    pass

class InvalidIOCError(SOCError):
    def __init__(self, ioc, reason):
        super().__init__(reason)
        self.ioc = ioc

class RateLimitError(SOCError):
    pass

def check_ioc(ioc):
    if not ioc.count(".") == 3:
        raise InvalidIOCError(ioc, "not a valid IPv4 format")

try:
    check_ioc("999.999.999")
except InvalidIOCError as e:
    print(f"Rejected IOC {e.ioc}: {e}")
except SOCError as e:
    print(f"Pipeline error: {e}")   # catches any other custom SOC error
```
Building a custom exception hierarchy (`SOCError` as base) lets a detection pipeline distinguish "bad IOC format" from "API rate-limited" from "auth failed" — while still letting a catch-all `except SOCError` handle anything you didn't specifically anticipate.

---

## Quick-Reference Summary

| Concept | Key Syntax |
|---|---|
| Define class | `class Name:` |
| Constructor | `def __init__(self, ...):` |
| Create object | `obj = Name(...)` |
| Instance var | `self.x = val` (per-object) |
| Class var | `x = val` inside class body, outside methods (shared) |
| Private | `self.__x` → mangled to `_ClassName__x` |
| Check existence | `hasattr(obj, 'x')` |
| Get/set dynamically | `getattr(obj,'x')` / `setattr(obj,'x',val)` |
| Introspect | `__dict__`, `__name__`, `__module__`, `__bases__`, `type(obj)` |
| Inherit | `class Sub(Super):` |
| Call super | `super().__init__(...)` |
| Check relationship | `issubclass(A,B)`, `isinstance(obj,Class)`, `a is b` |
| Override method | redefine same method name in subclass (polymorphism) |
| Multiple inherit | `class Sub(A, B):` — left-to-right priority, avoid if possible |
| Composition | store other objects as properties instead of inheriting |
| Custom string | `def __str__(self): return "..."` |
| Custom exception | `class MyError(Exception):` |
| try extras | `try / except X as e / else / finally` |
