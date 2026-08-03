# Module 4 — Functions, Tuples, Dictionaries & Exceptions

---

## 4.1 Functions — Basics

### 🔹 Why Functions?

- Avoid repeating code in multiple places → easier maintenance, fewer bugs.
- **Decomposition**: break large/complex problems into small, testable pieces.
- Enables teamwork — different developers write separate functions/modules.

### 🔹 Where Functions Come From

| Source | Example |
|---|---|
| Built-in (Python itself) | `print()`, `input()`, `len()` |
| Pre-installed modules | `math.sqrt()` |
| Your own code | user-defined functions |
| Classes | methods (covered later) |

### 🔹 Defining & Calling

```python
def message():          # def + name + () + colon
    print("Enter a value: ")

message()               # must be called explicitly — defining ≠ running
```

### 🔹 Key Rules ⭐

- Python reads top-to-bottom — **you can't call a function before it's defined**.
  ```python
  hi()              # NameError: name 'hi' is not defined
  def hi():
      print("hi")
  ```
- A variable and a function **can't share a name**.
  ```python
  def message(): print("hi")
  message = 1      # function 'message' is now gone
  ```

<details>
<summary>🛡️ <b>SOC Use Case</b> — reusable log parser</summary>

```python
def log_alert(event_id, description):
    print(f"[ALERT] Event {event_id}: {description}")

log_alert(4625, "Failed Login")
log_alert(4720, "New Account Created")
```
One function = one place to fix/update alert formatting instead of editing every `print()` in the script.
</details>

---

## 4.2 How Functions Communicate — Parameters & Arguments

### 🔹 Parameters vs Arguments

- **Parameter** — variable inside `()` in `def`; only exists inside the function.
- **Argument** — the actual value passed in when calling.

```python
def message(number):
    print("Enter a number:", number)

message(1)     # 1 is the argument, number is the parameter
```

### 🔹 Shadowing ⭐

A parameter with the same name as an outside variable is a **completely separate entity** inside the function.

```python
def message(number):
    print("Enter a number:", number)

number = 1234
message(1)        # Enter a number: 1
print(number)      # 1234  (outside variable untouched)
```

### 🔹 Positional vs Keyword Arguments

```python
def introduction(first_name, last_name):
    print("Hello, my name is", first_name, last_name)

introduction("Luke", "Skywalker")                       # positional — order matters
introduction(last_name="Skywalker", first_name="Luke")   # keyword — order doesn't matter
```

> ⚠️ Positional arguments **must** come before keyword ones: `adding(3, c=1, b=2)` ✅ but `adding(a=5, 2)` → `SyntaxError`.

### 🔹 Default Parameter Values ⭐

```python
def introduction(first_name, last_name="Smith"):
    print("Hello, my name is", first_name, last_name)

introduction("Henry")                  # Hello, my name is Henry Smith
```

<details>
<summary>🛡️ <b>SOC Use Case</b> — flexible blocklist checker</summary>

```python
def check_ip(ip, blocklist=("10.0.0.5", "192.168.1.10")):
    if ip in blocklist:
        print(f"🚫 {ip} is blocked")
    else:
        print(f"✅ {ip} is allowed")

check_ip("10.0.0.5")                          # uses default blocklist
check_ip("10.0.0.5", blocklist=("10.0.0.5",))  # custom blocklist override
```
</details>

---

## 4.3 Returning Results

### 🔹 `return` — Two Variants

```python
# Without expression → exits function early
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

> 💡 No `return <expr>` in a function → it **implicitly returns `None`**.

### 🔹 `None` ⭐

```python
print(None + 2)   # TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'

value = None
if value is None:      # safe: comparison, not arithmetic
    print("no value")
```

### 🔹 Lists as Arguments / Return Values

```python
def list_sum(lst):
    s = 0
    for elem in lst:
        s += elem
    return s

print(list_sum([5, 4, 3]))   # 12
print(list_sum(5))            # TypeError: 'int' object is not iterable
```

<details>
<summary>🛡️ <b>SOC Use Case</b> — extracting failed IPs from logs</summary>

```python
def get_failed_ips(logs):
    failed = []
    for entry in logs:
        if entry["status"] == "failed":
            failed.append(entry["ip"])
    return failed

logs = [
    {"ip": "10.0.0.5", "status": "failed"},
    {"ip": "10.0.0.9", "status": "success"}
]
print(get_failed_ips(logs))   # ['10.0.0.5']
```
</details>

---

## 4.4 Scopes

### 🔹 Local vs Global Rule ⭐

> A variable defined outside a function IS visible inside it (read-only) — **unless** the function creates its own variable of the same name, which then **shadows** it.

```python
var = 1
def my_function():
    var = 2                 # creates a NEW local variable
    print("Inside:", var)   # 2
my_function()
print("Outside:", var)       # still 1 — outer var untouched
```

### 🔹 `global` Keyword

Forces the function to use (and modify) the **outer** variable instead of creating a local one.

```python
var = 2
def return_var():
    global var
    var = 5
    return var

print(return_var())   # 5
print(var)              # 5 — outer variable changed
```

### 🔹 Function Arguments: Scalars vs Lists ⭐ (common interview trap)

**Scalars** — reassigning the parameter never affects the caller's variable.
**Lists** — reassigning the parameter also doesn't affect the caller, BUT **mutating it in place** does, because both names point to the same object.

<details>
<summary>🛡️ <b>SOC Use Case</b> — mutating a shared alert list</summary>

```python
def clear_investigated(alerts):
    del alerts[0]        # in-place mutation → affects the original list!

active_alerts = ["malware", "phishing", "brute_force"]
clear_investigated(active_alerts)
print(active_alerts)     # ['phishing', 'brute_force']  ← changed!
```
Be careful passing shared log/alert lists into functions — `.append()`/`.remove()`/`del` mutate the caller's data; reassignment (`alerts = [...]`) does not.
</details>

---

## 4.5 Multi-Parameter Functions & Recursion

### 🔹 Sample: BMI, Unit Conversion

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

> ⚠️ `in` can't be used as a parameter name — it's a Python keyword.

### 🔹 Triangle Checks (compacting a function) ⭐

```python
def is_a_triangle(a, b, c):
    return a + b > c and b + c > a and c + a > b
```

- Right-triangle check applies the Pythagorean theorem to the **longest** side (hypotenuse).
- Floating-point results may be *very close* to but not exactly correct (e.g., `0.49999999999999983` instead of `0.5`) — a precision artifact, not a bug.

### 🔹 Factorial (iterative)

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

### 🔹 Recursion ⭐

A function that calls **itself**, with a **base case** to stop the chain.

```python
def factorial(n):
    if n == 1:          # base case
        return 1
    else:
        return n * factorial(n - 1)

def fib(n):
    if n < 1:
        return None
    if n < 3:
        return 1
    return fib(n - 1) + fib(n - 2)
```

> ⚠️ Missing a base case → infinite recursion. Recursion also uses more memory than an equivalent loop.

<details>
<summary>🛡️ <b>SOC Use Case</b> — recursive directory/file traversal</summary>

```python
def count_suspicious(files):
    if not files:
        return 0
    first, rest = files[0], files[1:]
    flagged = 1 if first.endswith(".exe") else 0
    return flagged + count_suspicious(rest)

print(count_suspicious(["a.txt", "b.exe", "c.exe"]))   # 2
```
Recursive patterns are handy for nested structures (folders-in-folders, threat trees).
</details>

---

## 4.6 Tuples & Dictionaries

### 🔹 Sequence Types & Mutability

| Term | Meaning |
|---|---|
| Sequence type | Can be iterated element-by-element with `for` (lists, tuples, strings) |
| Mutable | Can be changed in place (e.g. `list.append()`) |
| Immutable | Cannot — "changing" it means building a new object (tuples) |

### 🔹 Tuples

```python
tuple_1 = (1, 2, 4, 8)          # parentheses (conventional)
tuple_2 = 1., .5, .25, .125      # commas alone are enough

empty_tuple = ()
one_element_tuple = (1,)          # trailing comma REQUIRED — else it's just an int
```

```python
my_tuple.append(10)   # ❌ AttributeError: 'tuple' object has no attribute 'append'
my_tuple[1] = -10       # ❌ TypeError — tuples can't be modified
```

> 💡 Elegant multi-variable swap: `t1, t2, t3 = t2, t3, t1`

### 🔹 Dictionaries

Key–value pairs, **mutable**, ordered by default (Python 3.6+).

```python
dictionary = {"cat": "chat", "dog": "chien", "horse": "cheval"}

if "cat" in dictionary:            # safe check before access
    print(dictionary["cat"])        # chat
```

### 🔹 Common Dictionary Operations

```python
for key, value in dictionary.items():
    print(key, "->", value)

dictionary['cat'] = 'minou'             # modify existing key
dictionary['swan'] = 'cygne'             # add new key (no error, unlike lists)
dictionary.update({"duck": "canard"})   # add/merge

del dictionary['dog']        # remove a key
dictionary.popitem()          # removes the last inserted item
dictionary.clear()            # removes all items
dictionary.copy()             # shallow copy
```

<details>
<summary>🛡️ <b>SOC Use Case</b> — tracking failed login attempts per user</summary>

```python
login_attempts = {}

def log_attempt(user, timestamp):
    login_attempts[user] = login_attempts.get(user, ()) + (timestamp,)

log_attempt("alice", "09:01")
log_attempt("alice", "09:03")
log_attempt("bob", "09:10")

for user, times in login_attempts.items():
    if len(times) >= 2:
        print(f"⚠️ {user} has {len(times)} attempts: {times}")
```
</details>

### 🔹 Quick Reference

| Structure | Ordered | Mutable | Access by |
|---|:---:|:---:|---|
| List | ✅ | ✅ | index |
| Tuple | ✅ | ❌ | index |
| Dictionary | ✅ (3.6+) | ✅ | key |

---

## 4.7 Exceptions

### 🔹 Errors vs Exceptions ⭐

| Type | When it happens |
|---|---|
| **Syntax error** | Code violates Python grammar — caught at parse time |
| **Exception** | Code is valid but fails at *runtime* (bad input, division by zero, etc.) |

### 🔹 `try` / `except`

> Python philosophy: *"easier to ask forgiveness than permission"* — handle errors when they happen instead of pre-validating everything.

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

- Only **one** matching `except` branch executes.
- Group multiple exceptions in one line: `except (ValueError, ZeroDivisionError):`
- The bare `except:` (no name) must always come **last**.

### 🔹 Common Built-in Exceptions

| Exception | Cause |
|---|---|
| `ZeroDivisionError` | Division/modulo by zero (`/`, `//`, `%`) |
| `ValueError` | Right type, wrong value (e.g. `int("abc")`) |
| `TypeError` | Wrong type used in an operation |
| `AttributeError` | Calling a method that doesn't exist on that object |
| `SyntaxError` | Grammar violation — fix the code, don't catch it |
| `KeyboardInterrupt` | User hits Ctrl+C |

<details>
<summary>🛡️ <b>SOC Use Case</b> — resilient log parser</summary>

```python
logs = [{"id": "4625"}, {"id": "oops"}, {}]

for entry in logs:
    try:
        event_id = int(entry["id"])
        print("Processing event:", event_id)
    except (ValueError, KeyError):
        print("⚠️ Skipping malformed entry:", entry)
        continue
```
Very high importance — automation scripts parsing untrusted log data must handle malformed entries gracefully instead of crashing mid-run.
</details>

### 🔹 Testing & Debugging (concepts)

- **Execution paths**: test data should cover every branch (`if`/`elif`/`else`, loops, `try`/`except`).
- **Print debugging**: insert `print()` statements to trace variable states and flow.
- **Rubber duck debugging**: explain your code line-by-line to someone (or something) — often reveals the bug yourself.
- **Unit testing**: predictable test cases alongside your code (`unittest` module) so future changes can be verified automatically.

---

## 🎯 Master Interview Checklist — Module 4

- [ ] Functions reduce repetition and enable decomposition/teamwork
- [ ] Must define a function **before** calling it; function name ≠ variable name
- [ ] Parameter vs argument; shadowing outer variables
- [ ] Positional args before keyword args; default parameter values
- [ ] `return` with/without expression; missing `return` → implicit `None`
- [ ] `None` can't be used in expressions — only assigned or compared with `is`
- [ ] Local scope shadows global; `global` keyword needed to modify outer variable
- [ ] Scalars: changes don't propagate out. Lists: reassignment doesn't propagate, but **in-place mutation** does
- [ ] Recursion needs a base case; risk of infinite recursion / high memory use
- [ ] Tuples: immutable, `()` syntax, trailing comma for single-element tuples
- [ ] Dictionaries: mutable key-value store; `.keys()` / `.values()` / `.items()`; check with `in` before indexing
- [ ] Syntax errors vs exceptions — exceptions are catchable at runtime
- [ ] `try`/`except` order: specific exceptions first, generic `except:` last
- [ ] Common exceptions: `ZeroDivisionError`, `ValueError`, `TypeError`, `AttributeError`
