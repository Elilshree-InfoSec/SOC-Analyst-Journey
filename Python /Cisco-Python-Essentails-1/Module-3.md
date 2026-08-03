# Module 3 : Boolean Values, Conditional Execution, Loops, Lists and List Processing, Logical and Bitwise Operations

---

## 3.1 Making Decisions

### Comparison Operators

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `>` | Greater than |
| `>=` | Greater than or equal to |
| `<` | Less than |
| `<=` | Less than or equal to |

```python
print(5 == 5)   # True
print(5 != 3)   # True
print(10 > 5)   # True
print(10 >= 10) # True
```

> ⭐ Don't confuse `=` (assignment) with `==` (comparison).

### `if` Statement

```python
age = 20
if age >= 18:
    print("Adult")
```

- Python uses **indentation** (4 spaces) to define a block — no braces `{}`.
- Missing indentation → `IndentationError`.

🛡️ **SOC use:** Comparing Event IDs, filtering logs, writing detection rules.
```python
if event_id == 4625:
    print("Failed Login")
```

---

## 3.2 Loops

### `while` Loop

```python
counter = 1
while counter <= 5:
    print(counter)
    counter += 1
```

- Condition checked **before** each iteration.
- If condition is `False` at the start, loop body never runs.
- Loop body must eventually change the condition (or it becomes infinite).

**Infinite loop:**
```python
while True:
    print("Hello")
```

### `for` Loop + `range()`

```python
for i in range(5):        # 0 1 2 3 4  (start=0 default)
    print(i)

for i in range(2, 8):     # 2 3 4 5 6 7 (stop not included)
    print(i)

for i in range(2, 8, 3):  # 2 5 (start, stop, step)
    print(i)
```

- `range(start, stop, step)` → stop value is **never included**.
- `range(5, 1)` produces nothing (no output, no error).

### `break` vs `continue`

| Keyword | Effect |
|---|---|
| `break` | Exits the loop immediately |
| `continue` | Skips current iteration, moves to next |

```python
for i in range(10):
    if i == 5:
        break        # stops loop entirely
    print(i)

for i in range(6):
    if i == 3:
        continue     # skips only i == 3
    print(i)
```

### `while...else` / `for...else`

- The `else` block runs **only if the loop finishes normally** (i.e., not exited via `break`).

```python
for i in range(3):
    print(i)
else:
    print("Finished")   # runs, since no break occurred
```

🛡️ **SOC use:**
```python
while monitoring:
    check_logs()

for log in logs:
    if log == "":
        continue
    if "malware" in log:
        break
```

---

## 3.3 Logic and Bit Operations

### Logical Operators

| Operator | Meaning |
|---|---|
| `and` | True only if both are True |
| `or` | True if at least one is True |
| `not` | Reverses True/False |

```python
print(age >= 18 and has_id)
print(5 > 2 or 10 < 5)
print(not True)   # False
```

**Operator priority:** `not` → `and` → `or` (highest to lowest)
```python
True or False and False   # evaluated as: True or (False and False) → True
```

### De Morgan's Laws ⭐

```
not (p and q)  ==  (not p) or (not q)
not (p or q)   ==  (not p) and (not q)
```

### Truthy/Falsy

- `0` → treated as `False`
- Any non-zero value → treated as `True`

🛡️ **SOC use:** Very high — detection rules, SIEM filters.
```python
if event_id == 4625 and failed_attempts >= 5:
    print("Brute Force Detected")
```

### Bitwise Operators

| Operator | Meaning |
|---|---|
| `&` | Bitwise AND |
| `\|` | Bitwise OR |
| `~` | Bitwise NOT |
| `^` | Bitwise XOR |
| `<<` | Left shift |
| `>>` | Right shift |

```python
print(15 & 16)   # 0
print(15 | 16)   # 31
print(15 ^ 16)   # 31
print(~15)       # -16 (two's complement)
print(17 << 2)   # 68  → 17 * 2**2
print(17 >> 1)   # 8   → 17 // 2
```

### Logical vs Bitwise ⭐

```python
i = 15
j = 22
print(i and j)  # 22  → logical: returns second operand if first is truthy
print(i & j)    # 6   → bitwise: works on individual bits
```

| Logical | Bitwise |
|---|---|
| Works on Boolean truthiness | Works on individual bits |
| Result can be either operand | Result is always an integer |

### Bit Masking

```python
mask = 8

if flag_register & mask:      # check a bit
    print("Bit is set")

flag_register &= ~mask        # reset a bit
flag_register |= mask         # set a bit
flag_register ^= mask         # toggle a bit
```

🛡️ **SOC use:** Permission flags, network protocols, malware analysis (medium importance).

---

## 3.4 Lists — Basics

### Creating & Accessing

```python
numbers = [10, 5, 7, 2, 1]     # ordered, mutable, 0-indexed
my_list = [1, "Hello", True, 3.14]   # can mix types

numbers[0]        # 10   (indexing)
numbers[-1]       # 1    (negative indexing → last element)
len(numbers)      # 5
```

### Modifying

```python
numbers[0] = 111          # update by index
del numbers[1]             # remove by index
numbers.append(4)          # add to end
numbers.insert(1, 100)     # insert at index (shifts elements right)
```

### Functions vs Methods ⭐

| Function | Method |
|---|---|
| Independent — `len(numbers)` | Belongs to an object — `numbers.append(4)` |

### Traversing

```python
# By index
for i in range(len(my_list)):
    total += my_list[i]

# Direct (Pythonic, preferred)
for number in my_list:
    total += number
```

### Swapping (no temp variable) ⭐

```python
a, b = b, a
my_list[0], my_list[4] = my_list[4], my_list[0]
```

### Reversing a List (manual)

```python
length = len(my_list)
for i in range(length // 2):
    my_list[i], my_list[length - i - 1] = my_list[length - i - 1], my_list[i]
```

🛡️ **SOC use:** Very high — storing IPs, usernames, alerts, log entries.

---

## 3.5 Sorting — Bubble Sort

### Concept

- Compares **adjacent elements**; swaps if out of order.
- After each pass, the largest unsorted element "bubbles" to the end.
- Repeat until a full pass makes **no swaps**.

```python
my_list = [8, 10, 6, 2, 4]
swapped = True

while swapped:
    swapped = False
    for i in range(len(my_list) - 1):
        if my_list[i] > my_list[i + 1]:
            swapped = True
            my_list[i], my_list[i + 1] = my_list[i + 1], my_list[i]

print(my_list)   # [2, 4, 6, 8, 10]
```

> `swapped` flag tells the loop whether another pass is needed.

### Built-in Methods (preferred in real code)

```python
my_list.sort()      # sorts ascending, in place
my_list.reverse()   # reverses current order — does NOT sort!
```

> ⚠️ `reverse()` on an unsorted list does **not** produce a sorted list — it just flips the current order.

🛡️ **SOC use:** `sort()` is used for timestamps, IPs, log data (medium importance). Bubble sort itself: low importance (rarely used in practice, mainly for interview/algorithm fundamentals).

---

## 3.6 List Operations

### Reference vs Copy ⭐ (very common interview trap)

```python
list_1 = [1]
list_2 = list_1        # list_2 points to the SAME list
list_1[0] = 2
print(list_2)           # [2]  ← changed too!

list_2 = list_1[:]     # real copy (slicing)
```

### Slicing

```python
my_list = [10, 8, 6, 4, 2]

my_list[1:3]    # [8, 6]      → index 1 up to (not including) 3
my_list[1:-1]   # [8, 6, 4]   → negative index works too
my_list[:3]     # [10, 8, 6]  → from start
my_list[3:]     # [4, 2]      → to end
my_list[:]      # full copy
```

### Deleting

```python
del my_list[1]        # remove one element
del my_list[1:3]       # remove a slice
del my_list[:]          # empty the list (list still exists)
del my_list             # delete the list entirely (variable no longer exists)
```

### Membership Checks

```python
2 in my_list        # True if value exists
5 not in my_list     # True if value does not exist
```

### Common Patterns

```python
# Find largest value
largest = my_list[0]
for i in my_list:
    if i > largest:
        largest = i

# Search with index
for i in range(len(my_list)):
    if my_list[i] == target:
        found = True
        break

# Remove duplicates
new_list = []
for number in numbers:
    if number not in new_list:
        new_list.append(number)
```

🛡️ **SOC use (high importance):**
```python
if user_ip in blocked_ips:      # blacklist check
    print("Blocked")

if "malware" in alerts:          # alert filtering
    print("Investigate")
```

### Quick Reference

| Syntax | Purpose |
|---|---|
| `list[:]` | Copy entire list |
| `list[start:end]` | Slice a section |
| `del list[index]` | Delete an item |
| `del list[:]` | Empty the list |
| `value in list` | Check existence |
| `value not in list` | Check absence |

---

## 3.7 Advanced Lists

### Nested Lists (Lists of Lists)

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

matrix[0][1]     # 2  → row 0, column 1
```

### List Comprehension ⭐

```python
# Basic
row = ["PAWN" for i in range(8)]

# Transform
squares = [x ** 2 for x in range(10)]

# Filter (with condition)
even = [x for x in numbers if x % 2 == 0]
```

Syntax: `[expression for item in sequence if condition]`

### Building a Matrix

```python
# Nested loop version
board = []
for i in range(3):
    row = ["EMPTY" for i in range(3)]
    board.append(row)

# Nested comprehension version (equivalent, more Pythonic)
board = [["EMPTY" for i in range(3)] for j in range(3)]
```

### Processing 2D Lists

```python
for row in matrix:
    for value in row:
        print(value)

# Find max value in matrix
highest = 0
for row in numbers:
    for value in row:
        if value > highest:
            highest = value
```

### 3D Lists

```python
cube = [[[1, 2], [3, 4]], [[5, 6], [7, 8]]]
cube[0][1][0]   # navigate: outer → row → column
```

🛡️ **SOC use (medium–high importance):**
```python
logs = [
    ["user1", "failed_login"],
    ["user2", "success_login"]
]
logs[0][1]     # "failed_login"

alerts = [x for x in events if x == "malware"]   # comprehension for filtering
```

> 💡 For SOC/security roles: **prioritize list comprehension, filtering, and log processing** over deep matrix/3D-array work — those are more general CS fundamentals.

---

## 🎯 Master Interview Checklist

- [ ] `==` vs `=`, indentation rules for `if`
- [ ] `while` checks condition before every loop; `for` + `range(start, stop, step)`
- [ ] `break` exits loop; `continue` skips one iteration; `else` runs only if no `break`
- [ ] `and` / `or` / `not` operator priority + De Morgan's Laws
- [ ] Bitwise (`&ǀ~^<<>>`) vs Logical operators — different results, different use
- [ ] Lists are mutable, ordered, 0-indexed; negative indexing from `-1`
- [ ] `list_2 = list_1` → same reference; `list_2 = list_1[:]` → real copy
- [ ] Slicing: `list[start:end]` — end index excluded
- [ ] `del` variants: single item, slice, empty list, whole list
- [ ] Bubble sort logic + `swapped` flag; `.sort()` vs `.reverse()` (reverse ≠ sort)
- [ ] List comprehension: `[expr for item in seq if cond]`
- [ ] Nested lists / matrices: `list[row][column]`
