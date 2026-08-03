# Module 4 - Functions, Tuples, Dictionaries, Exceptions & Data Processing

---

## 4.1 Functions — Basics

### Why Functions?

- Avoid repeating the same code in multiple places → hard to maintain, error-prone when fixing bugs across copies.
- **Decomposition**: break large/complex problems into smaller, well-isolated, testable pieces.
- Enables teamwork: different developers can write separate functions/modules.

### Where Functions Come From

| Source | Example |
|---|---|
| Built-in (Python itself) | `print()`, `input()`, `len()` |
| Pre-installed modules | `math.sqrt()` |
| Your own code | user-defined functions |
| Classes | methods (covered later) |

### Defining & Calling

```python
def message():          # def keyword + name + () + colon
    print("Enter a value: ")

message()               # must be called explicitly — defining ≠ running
```

### Key Rules ⭐

- Python reads top-to-bottom — **you can't call a function before it's defined**.
  ```python
  hi()              # NameError: name 'hi' is not defined
  def hi():
      print("hi")
  ```
- A variable and a function **can't share a name** — assigning a value to the name overwrites the function.
  ```python
  def message(): print("hi")
  message = 1      # function 'message' is now gone
  ```
- Functions don't have to be at the top of the file — just defined before they're called.

🛡️ **SOC use:** Wrapping repeated log-parsing/alert logic into one function makes fixing detection logic a one-place change instead of hunting through the whole script.

---

## 4.2 How Functions Communicate — Parameters & Arguments

### Parameters vs Arguments

- **Parameter** — variable defined inside `()` in `def`; only exists inside the function.
- **Argument** — the actual value passed in when calling.

```python
def message(number):
    print("Enter a number:", number)

message(1)     # 1 is the argument, number is the parameter
```

- Missing a required argument → `TypeError: missing 1 required positional argument`.

### Shadowing ⭐

A parameter with the same name as an outside variable is a **completely separate entity** inside the function.

```python
def message(number):
    print("Enter a number:", number)

number = 1234
message(1)        # Enter a number: 1
print(number)      # 1234  (outside variable untouched)
```

### Positional vs Keyword Arguments

```python
def introduction(first_name, last_name):
    print("Hello, my name is", first_name, last_name)

introduction("Luke", "Skywalker")                       # positional — order matters
introduction(last_name="Skywalker", first_name="Luke")   # keyword — order doesn't matter
```

- Mixing is allowed, but **positional arguments must come before keyword ones**.
  ```python
  adding(3, c=1, b=2)     # OK
  adding(a=5, 2)           # SyntaxError
  ```
- Passing the same parameter twice → `TypeError: got multiple values for argument`.

### Default Parameter Values ⭐

```python
def introduction(first_name, last_name="Smith"):
    print("Hello, my name is", first_name, last_name)

introduction("Henry")                  # Hello, my name is Henry Smith
introduction(first_name="William")     # Hello, my name is William Smith
```

- If a required (non-default) parameter is placed **after** a default one → `SyntaxError`.

🛡️ **SOC use:** Functions like `check_ip(ip, blocklist=default_list)` let analysts override defaults only when needed.

---

## 4.3 Returning Results

### `return` — Two Variants

```python
# Without expression → just exits the function early
def happy_new_year(wishes=True):
    print("Three...")
    if not wishes:
        return
    print("Happy New Year!")

# With expression → exits AND sends a value back
def boring_function():
    return 123

x = boring_function()   # x = 123
```

- A function's result can be **ignored** entirely (its side effect, like a `print()`, still happens).
- If a function has no `return <expr>`, it **implicitly returns `None`**.

### `None` ⭐

- Represents "no value" — not usable in expressions.
  ```python
  print(None + 2)   # TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'
  ```
- Safe uses: assign it, or compare with `is`.
  ```python
  if value is None:
      print("no value")
  ```

### Lists as Arguments / Return Values

```python
def list_sum(lst):
    s = 0
    for elem in lst:
        s += elem
    return s

print(list_sum([5, 4, 3]))   # 12
print(list_sum(5))            # TypeError: 'int' object is not iterable
```

- A function can also **build and return** a list:
  ```python
  def create_list(n):
      my_list = []
      for i in range(n):
          my_list.append(i)
      return my_list
  ```

🛡️ **SOC use:** A function like `get_failed_ips(logs)` returning a list of IPs lets you chain it directly into filtering/reporting logic.

---

## 4.4 Scopes

### Local vs Global Rule ⭐

> A variable defined outside a function IS visible inside the function (read-only) — **unless** the function defines its own variable of the same name, in which case the local one **shadows** it.

```python
var = 1
def my_function():
    print("Do I know that variable?", var)   # reads outer var → 1
my_function()
print(var)   # 1
```

```python
var = 1
def my_function():
    var = 2                 # creates a NEW local variable
    print("Do I know that variable?", var)   # 2
my_function()
print(var)   # still 1 — outer var untouched
```

### `global` Keyword

Forces the function to use (and modify) the **outer** variable instead of creating a local one.

```python
var = 2
def return_var():
    global var
    var = 5
    return var

print(return_var())   # 5
print(var)             # 5 — outer variable changed
```

### Function Arguments: Scalars vs Lists ⭐ (common interview trap)

**Scalars** — reassigning the parameter never affects the caller's variable:
```python
def my_function(par):
    par = 100
n = 1
my_function(n)
print(n)   # still 1
```

**Lists** — reassigning the parameter (`my_list_1 = [0,1]`) also does NOT affect the caller's list. But **mutating** it in place (`del my_list_1[0]`, `.append()`, etc.) **does** affect the original, because both names point to the same list object.

```python
def my_function(my_list_1):
    del my_list_1[0]     # modifies the actual list object

my_list_2 = [2, 3]
my_function(my_list_2)
print(my_list_2)   # [3]  ← changed!
```

🛡️ **SOC use:** Be careful passing shared log/alert lists into functions — in-place modification (`.append()`, `.remove()`) will affect the caller's data; reassignment won't.

---

## 4.5 Multi-Parameter Functions & Recursion

### Sample: BMI, Unit Conversion

```python
def bmi(weight, height):
    if height < 1.0 or height > 2.5 or weight < 20 or weight > 200:
        return None      # guard against invalid input
    return weight / height ** 2

def lb_to_kg(lb):
    return lb * 0.45359237

def ft_and_inch_to_m(ft, inch=0.0):   # default parameter
    return ft * 0.3048 + inch * 0.0254
```

> ⭐ `in` can't be used as a parameter name — it's a Python keyword.

### Triangle Checks (compacting a function) ⭐

```python
# Verbose → compact, single-expression version
def is_a_triangle(a, b, c):
    return a + b > c and b + c > a and c + a > b
```

- Right-triangle check applies the Pythagorean theorem to whichever side is the **longest** (hypotenuse).
- Triangle area via **Heron's formula** using `** 0.5` for square root.
- Floating-point results may be *very close* to but not exactly the expected value (e.g., `0.49999999999999983` instead of `0.5`) — this is a floating-point precision artifact, not a bug.

### Factorial (iterative)

```python
def factorial_function(n):
    if n < 0:
        return None
    if n < 2:
        return 1
    product = 1
    for i in range(2, n + 1):
        product *= i
    return product
```

### Fibonacci (iterative)

```python
def fib(n):
    if n < 1:
        return None
    if n < 3:
        return 1
    elem_1 = elem_2 = 1
    for i in range(3, n + 1):
        the_sum = elem_1 + elem_2
        elem_1, elem_2 = elem_2, the_sum
    return the_sum
```

### Recursion ⭐

A function that calls **itself**, with a **base case** to stop the chain.

```python
# Recursive factorial
def factorial(n):
    if n == 1:          # base case
        return 1
    else:
        return n * factorial(n - 1)

# Recursive Fibonacci
def fib(n):
    if n < 1:
        return None
    if n < 3:
        return 1
    return fib(n - 1) + fib(n - 2)
```

> ⚠️ Missing a proper base case → infinite recursion. Recursive calls also consume more memory than loops (can be less efficient).

---

## 4.6 Tuples & Dictionaries

### Sequence Types & Mutability

- **Sequence type**: data that can be iterated element-by-element with `for` (lists, tuples, strings).
- **Mutable**: can be changed in place (`list.append()`).
- **Immutable**: cannot (tuples) — any "change" requires building a new object.

### Tuples

```python
tuple_1 = (1, 2, 4, 8)        # parentheses (optional, but conventional)
tuple_2 = 1., .5, .25, .125    # commas alone are enough

empty_tuple = ()
one_element_tuple = (1,)       # trailing comma is REQUIRED — without it, it's just an int
```

- Behaves like a list for reading (`indexing`, slicing, `for`, `len()`, `in`/`not in`, `+`, `*`) — but **cannot be modified**:
  ```python
  my_tuple.append(10)   # AttributeError: 'tuple' object has no attribute 'append'
  my_tuple[1] = -10       # TypeError
  ```
- Tuples support elegant multi-variable swapping:
  ```python
  t1, t2, t3 = t2, t3, t1
  ```

🛡️ **SOC use:** Immutability makes tuples good for fixed records (e.g., a log entry snapshot) you don't want accidentally altered.

### Dictionaries

Key–value pairs, **mutable**, ordered by default (Python 3.6+).

```python
dictionary = {"cat": "chat", "dog": "chien", "horse": "cheval"}
phone_numbers = {'boss': 5551234567, 'Suzy': 22657854310}
empty_dictionary = {}
```

- Keys must be **unique** and **immutable** (string, number — not a list).
- Access: `dictionary['cat']` → error if key doesn't exist. Safer: check with `in`.
  ```python
  if word in dictionary:
      print(dictionary[word])
  ```

### Common Dictionary Operations

```python
dictionary.keys()          # iterable of keys
dictionary.values()        # iterable of values
dictionary.items()          # iterable of (key, value) tuples

for key, value in dictionary.items():
    print(key, "->", value)

dictionary['cat'] = 'minou'         # modify existing key
dictionary['swan'] = 'cygne'         # add new key (no error, unlike lists)
dictionary.update({"duck": "canard"})  # alternate way to add/merge

del dictionary['dog']        # remove a key (error if key doesn't exist)
dictionary.popitem()          # removes the last inserted item
dictionary.clear()            # removes all items
dictionary.copy()             # shallow copy
```

🛡️ **SOC use (high):**
```python
# Tuple + dictionary combo — e.g., mapping usernames to a tuple of failed-login timestamps
login_attempts = {}
login_attempts["user1"] = login_attempts.get("user1", ()) + (timestamp,)
```

### Quick Reference

| Structure | Ordered | Mutable | Access by |
|---|---|---|---|
| List | ✅ | ✅ | index |
| Tuple | ✅ | ❌ | index |
| Dictionary | ✅ (3.6+) | ✅ | key |

---

## 4.7 Exceptions

### Errors vs Exceptions ⭐

| Type | When it happens |
|---|---|
| **Syntax error** | Code violates Python grammar — caught at parse time |
| **Exception** | Code is syntactically valid but fails at *runtime* (e.g., bad input, division by zero) |

### `try` / `except`

Python philosophy: *"It's easier to ask forgiveness than permission"* — handle errors when they happen rather than pre-validating everything.

```python
try:
    value = int(input("Enter a value: "))
    print(1 / value)
except ValueError:
    print("Wrong value.")
except ZeroDivisionError:
    print("Cannot divide by zero.")
except:                          # generic/default — must be LAST
    print("Something else went wrong.")
```

- Control jumps straight to `except` the moment an error occurs inside `try` — remaining `try` code is skipped.
- Multiple `except` blocks can catch different exceptions (only one branch executes).
- You can also group exceptions in one line:
  ```python
  except (ValueError, ZeroDivisionError):
      print("Invalid input or division by zero.")
  ```
- The default `except:` (no name) must always be the **last** branch.

### Common Built-in Exceptions

| Exception | Cause |
|---|---|
| `ZeroDivisionError` | Division/modulo by zero (`/`, `//`, `%`) |
| `ValueError` | Right type, wrong/inappropriate value (e.g., `int("abc")`) |
| `TypeError` | Wrong type used in an operation (e.g., float as list index) |
| `AttributeError` | Calling a method that doesn't exist on that object |
| `SyntaxError` | Code violates Python grammar (don't try to catch this — fix the code) |
| `KeyboardInterrupt` | User hits Ctrl+C |

🛡️ **SOC use (very high):** Automation scripts parsing untrusted log data must handle malformed entries gracefully instead of crashing mid-run.
```python
for entry in logs:
    try:
        event_id = int(entry["id"])
    except (ValueError, KeyError):
        continue     # skip malformed entries, keep processing
```

### Testing & Debugging (concepts)

- **Execution paths**: test data should cover every branch (`if`/`elif`/`else`, loops, `try`/`except`).
- Python is interpreted — it won't catch errors in code paths that never execute (e.g., a typo inside an untested `if` branch stays hidden until that branch runs).
- **Print debugging**: insert `print()` statements to trace variable states and execution flow.
- **Rubber duck debugging**: explain your code line-by-line to someone (or something) — often reveals the bug yourself.
- **Unit testing**: writing predictable test cases alongside your code (Python's `unittest` module) so any future change can be verified automatically.

---

## 🎯 Master Interview Checklist — Module 4

- [ ] Functions reduce repetition and enable decomposition/teamwork
- [ ] Must define a function **before** calling it; function name ≠ variable name
- [ ] Parameter vs argument; shadowing outer variables
- [ ] Positional args before keyword args; default parameter values
- [ ] `return` with/without expression; missing `return` → implicit `None`
- [ ] `None` can't be used in expressions — only assigned or compared with `is`
- [ ] Local scope shadows global; `global` keyword needed to modify outer variable from inside a function
- [ ] Passing scalars → changes don't propagate out; passing lists → **reassignment** doesn't propagate, but **in-place mutation** does
- [ ] Recursion needs a base case; risk of infinite recursion / high memory use
- [ ] Tuples: immutable, `()` syntax, trailing comma for single-element tuples
- [ ] Dictionaries: mutable key-value store; `.keys()` / `.values()` / `.items()`; check with `in` before indexing
- [ ] Syntax errors vs exceptions — exceptions are catchable at runtime
- [ ] `try`/`except` order: specific exceptions first, generic `except:` last
- [ ] Common exceptions: `ZeroDivisionError`, `ValueError`, `TypeError`, `AttributeError`
