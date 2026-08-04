# Python Essentials 2 
### Module 2: Strings, Strings & List Method and Exceptions

---

## 1. Characters, Encoding & I18N

# Character Representation & Code Standards

| Category | Key Concept / Standard | Core Details & Characteristics | Technical Specifications / Examples |
| :--- | :--- | :--- | :--- |
| **Foundations** | **Numbers Only** | Computers exclusively store numbers. Every character maps to a unique ID. | **Code Point**: The numeric mapping assigned to a specific character. |
| **ASCII** | **Standard English** | Covers the basic Latin alphabet, control codes, and whitespace characters. | <ul><li>**128 code points** (7 bits)</li><li>`Space` = 32 \| `A` = 65 \| `a` = 97</li><li>Upper vs. Lowercase delta = **32**</li><li>Alphabetical ordering: `A < B < C`</li></ul> |
| **I18N** | **Internationalization** | Shorthand for "I" + 18 letters + "N". Highlights that ASCII lacks space for world languages. | ASCII's 128 extended slots (128–255) are mathematically insufficient. |
| **Code Pages** | **Regional Fixes** | Standards assigning custom meanings to the upper 128 slots (128–255) per region. | **Ambiguous**: Code point `200` represents different characters across different pages. |
| **Unicode** | **Universal Solution** | Eliminates ambiguity by assigning a unique code point to almost every character globally. | <ul><li>**> 1,000,000 code points**</li><li>First 128 = Identical to ASCII</li><li>First 256 = Identical to ISO/IEC 8859-1</li></ul> |
| **Encodings** | **UCS-4** | Fixed-width encoding scheme where every single character takes up the exact same memory space. | <ul><li>**4 bytes** (32 bits) per character</li><li>Simple processing but wastes significant space</li><li>May use a **BOM** (Byte Order Mark)</li></ul> |
| **Encodings** | **UTF-8** | Variable-width encoding scheme that dynamically adjusts storage based on the character type. | <ul><li>**8 bits**: ASCII chars \| **16 bits**: Other Latin/non-Latin</li><li>**24 bits**: CJK (Chinese/Japanese/Korean)</li><li>Most common on web; no BOM required</li></ul> |
| **Modern Dev** | **Python 3 Native** | Python 3 architecture treats Unicode/UTF-8 as the baseline standard across the ecosystem. | Variable names, string primitives, and File I/O all support Unicode natively. |


---

## 2. String Fundamentals

- Strings are **immutable sequences** (like a read-only list of characters).
- `len(s)` → number of characters. **Escape chars (`\n`, `\t`) count as 1**, not more.
  ```python
  len("\n\n")   # 2, not 4
  ```
- **Multiline strings** — use triple quotes `'''...'''` or `"""..."""`. The line break itself becomes an invisible `\n` character and **counts toward length**.
  ```python
  s = """Line1
  Line2"""
  ```

### Operators (overloaded for strings)
| Operator | Meaning | Example | Result |
|---|---|---|---|
| `+` | concatenation (order matters — not commutative) | `'a' + 'b'` | `'ab'` |
| `*` | replication (needs string + number) | `'ab' * 2` | `'abab'` |
| `+=` `*=` | shortcut versions | | |

### `ord()` / `chr()`
- `ord(char)` → code point of a **single** character (else `TypeError`).
- `chr(number)` → character for that code point (else `ValueError`/`TypeError`).
- Always true: `chr(ord(x)) == x` and `ord(chr(x)) == x`.

### Strings as sequences
- **Indexing**: `s[0]`, negative indices work (`s[-1]` = last char). Out-of-range → `IndexError`.
- **Iterating**: `for ch in s:` works directly.
- **Slicing**: `s[start:stop:step]` — same rules as lists.
- **`in` / `not in`**: check substring presence → returns `True`/`False`.

### Immutability — what you CANNOT do
```python
s = "abc"
del s[0]        # ❌ Error — can only del the whole variable
s.append("d")   # ❌ no append() method exists for strings
s.insert(0,"a") # ❌ no insert() either
```
✅ Instead, always build a **new** string:
```python
s = s + "d"     # allowed — creates a new string object
```

### Sequence functions/methods usable on strings
| Name | What it does | Example | Result |
|---|---|---|---|
| `min(s)` | smallest code point char | `min("aAbB")` | `'A'` |
| `max(s)` | largest code point char | `max("aAbB")` | `'b'` |
| `s.index(sub)` | index of first match, **error if missing** | `"abcabc".index('c')` | `2` |
| `list(s)` | list of individual characters | `list("abc")` | `['a','b','c']` |
| `s.count(sub)` | counts non-overlapping occurrences | `"abcabc".count('a')` | `2` |

> ⚠️ `.index()` raises `ValueError` if not found. `.find()` (below) does not — it returns `-1`.

---

## 3. String Methods — Full Reference Table

All methods return a **new** string (or value) — the original is never modified.

| Method | What it does | Example | Result |
|---|---|---|---|
| `capitalize()` | 1st char upper, rest lower | `"pyTHON".capitalize()` | `'Python'` |
| `center(w)` | centers string in width `w` (spaces) | `"hi".center(6)` | `'  hi  '` |
| `center(w, ch)` | centers using fill char `ch` | `"hi".center(6,'*')` | `'**hi**'` |
| `count(sub)` | counts occurrences | `"abcabc".count("a")` | `2` |
| `endswith(sub)` | True if string ends with `sub` | `"file.py".endswith(".py")` | `True` |
| `find(sub)` | index of first match, **-1 if not found** (safe) | `"kappa".find("z")` | `-1` |
| `find(sub, start)` | search from index `start` | `"kappa".find("a",2)` | `4` |
| `find(sub, start, end)` | search within `[start:end)` | `"kappa".find("a",0,2)` | `1` |
| `isalnum()` | True if only letters + digits (no spaces) | `"abc123".isalnum()` | `True` |
| `isalpha()` | True if only letters | `"abc".isalpha()` | `True` |
| `isdigit()` | True if only digits | `"123".isdigit()` | `True` |
| `islower()` | True if all letters lower-case | `"abc".islower()` | `True` |
| `isspace()` | True if only whitespace | `"  ".isspace()` | `True` |
| `isupper()` | True if all letters upper-case | `"ABC".isupper()` | `True` |
| `join(list)` | joins list elements using string as separator | `",".join(["a","b"])` | `'a,b'` |
| `lower()` | all lower-case | `"ABC".lower()` | `'abc'` |
| `lstrip()` | removes leading whitespace | `"  hi".lstrip()` | `'hi'` |
| `lstrip(chars)` | removes leading chars listed | `"xxhi".lstrip("x")` | `'hi'` |
| `replace(old,new)` | replaces all occurrences | `"aaa".replace("a","b")` | `'bbb'` |
| `replace(old,new,n)` | replaces first `n` occurrences only | `"aaa".replace("a","b",1)` | `'baa'` |
| `rfind(sub)` | like `find()` but searches from the **right** | `"abcabc".rfind("a")` | `3` |
| `rstrip()` | removes trailing whitespace | `"hi  ".rstrip()` | `'hi'` |
| `split()` | splits on whitespace → list | `"a b c".split()` | `['a','b','c']` |
| `startswith(sub)` | True if string starts with `sub` | `"file.py".startswith("file")` | `True` |
| `strip()` | removes leading + trailing whitespace | `"  hi  ".strip()` | `'hi'` |
| `swapcase()` | flips upper↔lower | `"AbC".swapcase()` | `'aBc'` |
| `title()` | capitalizes first letter of every word | `"hi there".title()` | `'Hi There'` |
| `upper()` | all upper-case | `"abc".upper()` | `'ABC'` |

> 💡 `join()` is the **reverse** of `split()`.
> 💡 Prefer `in` over `find()` if you only need a True/False check — it's faster.

### 🔐 SOC relevance
```python
log = "  2024-01-01 10:00:00 FAILED_LOGIN user=admin src=203.0.113.5  "
log = log.strip()
if "FAILED_LOGIN" in log:
    ip = log.split("src=")[1]
    print(ip)                     # extract attacker IP from raw log line
```
`split()`, `strip()`, `startswith()`/`endswith()`, and `in` are the everyday toolkit for parsing raw log lines, filenames, and alert payloads before feeding them into detection logic.

---

## 4. Strings in Action

### Comparing strings
- Same operators as numbers work: `== != > >= < <=`
- Comparison is **character-by-character using code points** — not "smart"/linguistic.
- Case-sensitive: upper-case < lower-case (`'Beta' < 'beta'` → `True`).
- Shorter string that matches the start of a longer one is considered **smaller**: `'alpha' < 'alphabet'` → `True`.
- **Strings vs numbers**: `==` and `!=` always work (`False`/`True` resp.), but `<`, `>`, `<=`, `>=` between a string and a number raise **`TypeError`**.
- A digit-only string (`"123"`) is still a string — **not** treated as a number in comparisons.

### Sorting
| Function/Method | Behavior |
|---|---|
| `sorted(list)` | returns a **new** sorted list, original untouched |
| `list.sort()` | sorts **in place**, returns `None` |

```python
g = ['omega','alpha','pi']
sorted(g)   # ['alpha','omega','pi'] — new list
g.sort()    # g itself is now sorted
```

### Converting strings ↔ numbers
| Function | Direction | Notes |
|---|---|---|
| `str(x)` | number → string | always works |
| `int(s)` | string → int | `ValueError` if not a valid integer |
| `float(s)` | string → float | `ValueError` if not a valid number |

### 🔐 SOC relevance
```python
severities = ["Medium", "Critical", "Low", "High"]
order = {"Low":0, "Medium":1, "High":2, "Critical":3}
severities.sort(key=lambda s: order[s])   # custom sort — not alphabetical
```
Plain `sorted()` would put "Critical" before "Low" alphabetically — wrong for alert triage. Always check whether default (lexicographic) sort actually reflects priority.

---

## 5. Worked String Programs (concepts to remember)

- **Caesar Cipher**: shift each letter's code point by N, wrap around past `Z`/`z` using modulo-like correction. Core pattern: `ord()` → shift → bounds-check → `chr()`.
- **IBAN validator**: move first 4 chars to the end, convert letters→digits (`A=10...Z=35`), treat result as one big integer, valid if `% 97 == 1`. Shows string manipulation + big-integer math combined.
- Common pattern across all four programs: **loop through characters, classify each one (`isalpha`/`isdigit`), build a new string/number incrementally** — since strings can't be modified in place.

### 🔐 SOC relevance
The Caesar cipher pattern (shift/substitute characters) is the same logic behind simple obfuscation techniques (ROT13, XOR string obfuscation) that malware sometimes uses to hide strings/IOCs — recognizing "shift by N" logic helps when reverse-engineering a decoder script.

---

## 6. Errors & Exceptions

- **Exception** = an object Python creates when something goes wrong; it stops normal flow unless handled.
- If unhandled → program terminates + Python prints a **traceback** + exception name.

### Basic syntax
```python
try:
    risky_code()
except:
    print("something went wrong")
```
- Code in `try` runs first; if it fails, execution **jumps immediately** into `except` (remaining `try` lines are skipped).
- If `try` succeeds fully, `except` is skipped entirely.

### Handling multiple exception types
```python
try:
    ...
except ValueError:
    print("bad value")
except ZeroDivisionError:
    print("div by zero")
except:
    print("something else")
```
- Branches checked **top to bottom** — first match wins, rest are skipped.
- Only **one** unnamed `except:` allowed, and it **must be last**.
- Grouping multiple exceptions in one branch: `except (ValueError, TypeError):`

### Key rule: exception order matters (hierarchy)
Python's exceptions form a tree — general → specific:
```
BaseException → Exception → ArithmeticError → ZeroDivisionError
BaseException → Exception → LookupError → IndexError
BaseException → Exception → LookupError → KeyError
```
- A branch catching a **general** exception (`ArithmeticError`) also catches its specific children (`ZeroDivisionError`).
- ⚠️ **Put specific exceptions BEFORE general ones**, or the specific branch becomes unreachable (Python won't warn you):
```python
# ❌ wrong order — ZeroDivisionError branch never runs
except ArithmeticError:
    ...
except ZeroDivisionError:
    ...
```

### `raise`
| Form | Use |
|---|---|
| `raise ExceptionName` | manually trigger an exception |
| `raise` (no name) | re-raise the **currently handled** exception — **only legal inside an `except` block** |

### `assert`
```python
assert expression
```
- If `expression` is `False` / `0` / `""` / `None` → raises `AssertionError` automatically.
- Used as a safety net/sanity check (like an airbag), **not** a replacement for real input validation.

---

## 7. Built-in Exception Cheat-Sheet

| Exception | Tree location | Raised when |
|---|---|---|
| `BaseException` | root | most general — catches everything |
| `Exception` | ← BaseException | general parent of nearly all normal exceptions |
| `ArithmeticError` | ← Exception | abstract parent of math errors |
| `ZeroDivisionError` | ← ArithmeticError | division by zero |
| `OverflowError` | ← ArithmeticError | number too large to store |
| `LookupError` | ← Exception | abstract parent of collection lookup errors |
| `IndexError` | ← LookupError | invalid sequence index (e.g. list out of range) |
| `KeyError` | ← LookupError | invalid dictionary key |
| `AssertionError` | ← Exception | `assert` condition failed |
| `ImportError` | ← Exception | module import failed |
| `MemoryError` | ← Exception | out of memory |
| `KeyboardInterrupt` | ← BaseException (NOT Exception!) | user pressed Ctrl-C |

> ⚠️ `KeyboardInterrupt` inherits directly from `BaseException`, **not** `Exception` — an `except Exception:` block will **not** catch it.

### 🔐 SOC relevance
```python
import json

def parse_alert(raw):
    try:
        data = json.loads(raw)
        return data["src_ip"], data["severity"]
    except (json.JSONDecodeError, KeyError) as e:
        print(f"Malformed alert, skipping: {e}")
        return None
```
Feeds/APIs send malformed or incomplete data constantly — a SIEM/automation script that dies on the first bad record is useless. Wrapping parsing logic in `try/except` (catching `KeyError` for missing fields, `ValueError`/`JSONDecodeError` for bad data) is standard practice, not optional.

---

## Quick-Reference Summary

- **Encoding**: ASCII (128 codes) → code pages (ambiguous, region-specific) → Unicode (unambiguous, global) stored as UCS-4 (fixed 4 bytes) or UTF-8 (variable width, most common).
- **Strings are immutable sequences** — indexable, sliceable, iterable, support `in`/`not in`, but no `del`, `append()`, or `insert()`.
- **`ord()`/`chr()`** convert between character ↔ code point.
- **String methods** (table above) all return new strings — `find()` is the safe version of `index()`; `split()`/`join()` are reverse operations.
- **Comparisons** are code-point-based, case-sensitive; string vs number only supports `==`/`!=`.
- **`sorted()`** returns a new list, **`.sort()`** sorts in place.
- **`str()` / `int()` / `float()`** convert between strings and numbers (may raise `ValueError`).
- **Exceptions**: `try/except`, order branches specific→general, one unnamed `except` allowed (must be last), `raise`/`raise ExceptionName`, `assert` for sanity checks.
- **Exception tree**: `BaseException → Exception → {ArithmeticError, LookupError, ...} → concrete errors`; `KeyboardInterrupt` sits outside `Exception`.
