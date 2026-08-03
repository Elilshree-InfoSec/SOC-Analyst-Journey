# 🐍 Python Essentials 1 - Module 2
## Python Data Types, Variables, Operators & Basic I/O

## 1. Introduction

- Created by Guido van Rossum (1989).
- Open-source, easy to learn, and highly readable.
- Python 3 is the current standard.
- Widely used in automation, scripting, log analysis, cybersecurity, and SOC.

### 1.1 Hello World

```python
print("Hello, World!")
```

- `print()` → Built-in function to display output on the screen.
- `"Hello, World!"` → A string (text enclosed in quotes).
- `()` → Parentheses required when calling a function.
- Quotes (`""` or `''`) → Tell Python the content is text, not code.

**Output:**
```
Hello, World!
```

> 💡 **Interview Tip:** `print()` is mainly used for displaying output and debugging/testing code.

### 1.2 Function Syntax

```python
function_name(argument)
```

Example:
```python
print("Hello")
```
- `print` → Function name
- `()` → Parentheses (required)
- `"Hello"` → Argument (String)

### 1.3 Strings

```python
"Hello"
'Hello'
```
- Text enclosed in quotes.
- Can use single (`' '`) or double (`" "`) quotes.

### 1.4 Multiple Arguments

```python
print("Hello", "World")
```
**Output:**
```
Hello World
```
- Separate arguments with commas.
- `print()` automatically adds a space.

### 1.5 New Line (`\n`)

```python
print("Hello\nWorld")
```
**Output:**
```
Hello
World
```
- `\n` = New line (escape character).

### 1.6 Empty print()

```python
print()
```
- Prints a blank line.

### 1.7 `sep` (Separator)

```python
print("A", "B", "C", sep="-")
```
**Output:**
```
A-B-C
```
- Changes the separator between arguments.

### 1.8 `end` (Ending Character)

```python
print("Hello", end=" ")
print("World")
```
**Output:**
```
Hello World
```
- Changes what is printed at the end.
- Default: `end="\n"` (new line).

---

# 2. Data Types

## 2.1 Literals

- **Literal** = fixed value written directly in code.
- Python uses literals to store data.

```python
123        # integer literal
"Hello"    # string literal
True       # boolean literal
```

### Types of Python Literals

## 2.2 Integer (int)

- Whole numbers without decimal.
- Can be positive or negative.

```python
123
-50
1_000_000   # underscore improves readability
```

Other number formats:
```python
0o123   # Octal (base 8)
0x123   # Hexadecimal (base 16)
```

## 2.3 Float

- Numbers with decimal points.
- Used for fractional values.

```python
3.14
-0.5
4.0
```

Scientific notation:
```python
3E8      # 3 × 10⁸
6.626E-34
```

## 2.4 String

- Text data enclosed in quotes.

```python
"Hello"
'Python'
```

Escape characters:
```python
\"   # double quote
\'   # apostrophe
\n   # new line
```

Example:
```python
print("I like \"Python\"")
```
**Output:**
```
I like "Python"
```

## 2.5 Boolean

- Represents `True` or `False` values.
- Only two values: `True`, `False`.
- Used in conditions and comparisons.

```python
10 > 5
```
**Output:**
```
True
```

### Integer vs Float

| Integer | Float |
|---|---|
| Whole number | Decimal number |
| `5` | `5.0` |
| No decimal point | Has decimal point |

### Important Interview Points ⭐

- Python is case-sensitive → `True ≠ true`, `False ≠ false`
- Strings need quotes: `"Hello"`
- Numbers do not need quotes: `100`
- `"100"` is a string, `100` is an integer

### Boolean as Numbers

Python treats:
```python
True = 1
False = 0
```

Example:
```python
print(True > False)
```
**Output:**
```
True
```
(because `1 > 0`)

---

# 3. Operators

- **Operators** = symbols used to perform operations on values.
- Values + Operators = Expressions.

```python
2 + 3
```
**Output:**
```
5
```

## 3.1 Arithmetic Operators

| Operator | Meaning | Example | Result |
|---|---|---|---|
| `+` | Addition | `5 + 2` | `7` |
| `-` | Subtraction | `5 - 2` | `3` |
| `*` | Multiplication | `5 * 2` | `10` |
| `/` | Division | `5 / 2` | `2.5` |
| `//` | Floor division | `5 // 2` | `2` |
| `%` | Remainder (modulo) | `5 % 2` | `1` |
| `**` | Power | `2 ** 3` | `8` |

### Important Rules ⭐

**`/` Division**
- Always returns float.
```python
5 / 2   # 2.5
4 / 2   # 2.0
```

**`//` Floor Division**
- Removes decimal part, rounds down.
```python
5 // 2    # 2
-5 // 2   # -3   (floor goes to smaller number)
```

**`%` Modulo**
- Returns remainder after division.
```python
10 % 3   # 10 ÷ 3 = 3 remainder 1 → Output: 1
```

**`**` Exponentiation**
- Power operator: `2 ** 3` means `2 × 2 × 2` → Output: `8`

### Unary vs Binary Operators

**Unary Operator** — Works with one value.
```python
-5   # Changes the sign.
```

**Binary Operator** — Works with two values.
```python
5 - 2
```

## 3.2 Operator Priority

Python follows this order:

| Priority | Operators |
|---|---|
| 1 | `()` Parentheses |
| 2 | `**` Power |
| 3 | Unary `+`, `-` |
| 4 | `*`, `/`, `//`, `%` |
| 5 | `+`, `-` |

Example:
```python
2 + 3 * 5
# 3 * 5 = 15
# 2 + 15 = 17
```
**Output:** `17`

### Parentheses Override Priority

```python
(2 + 3) * 5
# 2 + 3 = 5
# 5 * 5 = 25
```

### Binding Direction

- Most operators calculate **left → right**.
```python
9 % 6 % 2
# 9 % 6 = 3
# 3 % 2 = 1
```
**Output:** `1`

**Exception: Power `**`** — Works **right → left**.
```python
2 ** 3 ** 2
# 2 ** (3 ** 2)
# = 2 ** 9
# = 512
```

## 3.3 Important Rules ⭐

Quick interview revision:
```python
10 / 2     → 5.0
10 // 3    → 3
10 % 3     → 1
2 ** 3     → 8
```

Common mistakes:
- `/` always gives float
- `//` rounds down
- `%` gives remainder
- `**` means power

### 🛡️ SOC Analyst Relevance (Operators)

Medium importance. Used in:
- Calculating values in scripts
- Parsing logs
- Data processing
- Automation scripts

```python
failed_login_percentage = failed / total * 100
```

But for SOC internships, prioritize:
- Variables + Data types
- Conditions (if/else)
- Loops
- Functions
- Files/log parsing
- Regex
- Libraries (`re`, `os`, `json`, `requests`)

---

# 4. Variables

- **Variable** = a named container that stores a value.
- Used to save data and reuse it later.

```python
name = "John"
age = 20
```

A variable has:
- Name
- Value

## 4.1 Creating Variables

- Python does not require declaration. Just assign a value:
```python
x = 10
```
- Python automatically creates the variable.

```python
score = 100
print(score)
```
**Output:**
```
100
```

## 4.2 Variable Naming Rules ⭐

✅ Valid:
```python
age
user_name
student1
_count
```

✅ Can contain: Letters (a-z, A-Z), Numbers (0-9), Underscore (`_`)

✅ Must start with: Letter or Underscore

❌ Cannot start with number:
```python
1name   # wrong
```

❌ Cannot contain spaces:
```python
user name   # wrong
```

❌ Cannot use Python keywords:
```python
class = 10   # wrong
```

## 4.3 Python is Case-Sensitive

Different variables:
```python
age
Age
AGE
```

```python
name = "John"
Name = "Mary"
```
- These are two different variables.

## 4.4 Variable Assignment

- `=` means assign value, not equal.
```python
x = 5   # "Store 5 inside x"
```

Updating variables:
```python
x = 10
x = 20
# Final value: x = 20
```

## 4.5 Using Variables in Calculations

```python
a = 5
b = 3
result = a + b
print(result)
```
**Output:**
```
8
```

## 4.6 Combining Strings + Variables

```python
version = "3.12"
print("Python version: " + version)
```
**Output:**
```
Python version: 3.12
```

## 4.7 Shortcut Operators ⭐

Instead of:
```python
x = x + 5
```
Use:
```python
x += 5
```

| Normal | Shortcut |
|---|---|
| `x = x + 1` | `x += 1` |
| `x = x - 1` | `x -= 1` |
| `x = x * 2` | `x *= 2` |
| `x = x / 2` | `x /= 2` |

Example:
```python
score = 10
score += 5
# Result: 15
```

## 4.8 Dynamic Typing

- Python automatically detects the data type.
```python
x = 10      # int
x = "hello" # string
```
- No need to declare: ~~`int x = 10`~~ ❌

## 4.9 Important Interview Points ⭐

✅ Variable created when assigned: `x = 5`
✅ Python variables can change type:
```python
x = 10
x = "Python"
```
✅ `=` is assignment: `x = 100`
✅ Python is case-sensitive: `name ≠ Name`

### Quick Revision Table

| Concept | Example |
|---|---|
| Create variable | `x = 10` |
| String variable | `name = "Ali"` |
| Update value | `x = 20` |
| Add value | `x += 1` |
| Case-sensitive | `Age ≠ age` |
| Keyword not allowed | `for = 5` |

### 🛡️ SOC Analyst Relevance (Variables)

High importance. Used everywhere in security scripts:
```python
log_file = "auth.log"
failed_attempts = 10
ip_address = "192.168.1.5"
```

---

# 5. Comments

- **Comments** = notes written for humans.
- Python ignores comments during execution.
- Used to explain code, improve readability, and document logic.

```python
# This prints a message
print("Hello")
```
**Output:**
```
Hello
```

## 5.1 Single-Line Comments

Use `#`:
```python
# Store user age
age = 20
```
- Everything after `#` on that line is ignored.

```python
x = 10  # Assign value 10 to x
```

## 5.2 Multi-Line Comments

- Python does not have a special multi-line comment syntax.
- Use multiple `#`:
```python
# This program calculates
# the user's score
score = 100
```

## 5.3 Why Use Comments?

✅ Explain complex code
✅ Help other developers understand
✅ Remember your own logic later
✅ Temporarily disable code during testing

```python
# print("Debug message")
print("Program running")
```

## 5.4 Good Comments Practice ⭐

Use meaningful variable names instead of unnecessary comments.

Bad:
```python
a = 500  # stores area
```

Better:
```python
area = 500
```
- The variable name explains itself.

## 5.5 Comment Shortcut

Quickly comment/uncomment lines:
- **Windows:** `CTRL + /`
- **Mac:** `CMD + /`

---

# 6. Basic Input & Output

## 6.1 `input()` Function

- Used to get data from the user.
- Opposite of `print()`.

| Function | Purpose |
|---|---|
| `print()` | Output data |
| `input()` | Receive data |

```python
name = input("Enter name: ")
print(name)
```

## 6.2 Important Rule ⭐

`input()` always returns a string.

```python
age = input("Age: ")
print(type(age))
```
User enters: `20`

**Output:**
```
<class 'str'>
```
- Even numbers are stored as strings.

## 6.3 Type Conversion (Casting)

Convert input into numbers using:

**`int()`** — Converts to integer
```python
age = int(input("Age: "))
# Now age + 5 works.
```

**`float()`** — Converts to decimal number
```python
price = float(input("Price: "))
```
Input `5.5` → Stored as `float`.

**`str()`** — Converts value into string
```python
age = 20
text = str(age)
# Result: "20"
```

## 6.4 Common Mistake ⭐

Wrong:
```python
num = input("Number: ")
print(num + 5)
```
Error: `TypeError` — because `input` → string, `5` → integer.

Correct:
```python
num = int(input("Number: "))
print(num + 5)
```

## 6.5 String Operators

**String Concatenation `+`** — Joins strings together.
```python
first = "Cyber"
second = "Security"
print(first + second)
```
**Output:**
```
CyberSecurity
```
With space:
```python
print(first + " " + second)
```
**Output:**
```
Cyber Security
```

**String Replication `*`** — Repeats strings.
```python
print("Hi" * 3)
```
**Output:**
```
HiHiHi
```
Another example:
```python
print("5" * 3)
```
**Output:**
```
555
```
(Not `15`)

## 6.6 Example Program

```python
name = input("Name: ")
age = int(input("Age: "))

print("Hello " + name)
print(age + 1)
```

## 6.7 Quick Revision Table

| Function | Purpose | Example |
|---|---|---|
| `input()` | Get user input | `input("Name")` |
| `int()` | Convert to integer | `int("10")` |
| `float()` | Convert to float | `float("3.5")` |
| `str()` | Convert to string | `str(100)` |
| `+` | Join strings | `"A"+"B"` |
| `*` | Repeat strings | `"Hi"*3` |

### Interview Points ⭐

✅ `input()` always returns string
✅ Convert numbers before calculations
✅ `+` joins strings
✅ `*` repeats strings
✅ `int()` removes decimal part
✅ `float()` keeps decimal values

---

# 🛡️ SOC Analyst Relevance

Medium importance. Used for:
- Small security scripts
- Taking user parameters
- Creating CLI tools

```python
ip = input("Enter IP address: ")
print("Scanning " + ip)
```

Later in SOC automation, you'll mostly use:
- Reading files
- Parsing logs
- Processing JSON
- APIs

rather than manual `input()`.
