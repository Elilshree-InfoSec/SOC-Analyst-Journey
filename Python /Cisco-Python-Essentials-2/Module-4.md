# Python Essentials 2 
### Module 4: Miscellaneous

---

## 1. Iterators & the Iterator Protocol

| Term | Meaning |
|---|---|
| Function | returns ONE value, invoked once |
| Generator / Iterator | returns a **series** of values, invoked repeatedly |

An object is an **iterator** if it implements both:

| Method | Called | Job |
|---|---|---|
| `__iter__()` | once, at the start | returns the iterator object itself |
| `__next__()` | repeatedly | returns the next value; raises `StopIteration` when done |

```python
class Fib:
    def __init__(self, n):
        self.__n = n
        self.__i = 0
        self.__p1 = self.__p2 = 1

    def __iter__(self):
        return self              # must return itself

    def __next__(self):
        self.__i += 1
        if self.__i > self.__n:
            raise StopIteration  # signals "no more values"
        if self.__i in (1, 2):
            return 1
        self.__p1, self.__p2 = self.__p2, self.__p1 + self.__p2
        return self.__p2

for v in Fib(5):
    print(v, end=" ")   # 1 1 2 3 5
```

### 🔐 SOC relevance
```python
class LogBatchReader:
    """Iterate through a huge log file in fixed-size batches instead of
    loading it all into memory at once."""
    def __init__(self, path, batch_size=1000):
        self.__f = open(path)
        self.__batch = batch_size
    def __iter__(self):
        return self
    def __next__(self):
        lines = [self.__f.readline() for _ in range(self.__batch)]
        lines = [l for l in lines if l]
        if not lines:
            self.__f.close()
            raise StopIteration
        return lines
```
Custom iterators are the standard way to stream large log/alert files chunk-by-chunk without exhausting memory.

---

## 2. The `yield` Statement (Generators — the easy way)

`yield` = smarter `return` — **freezes** the function's state instead of discarding it.

```python
def fun(n):
    for i in range(n):
        yield i          # pauses here, resumes on next call

for v in fun(5):
    print(v, end=" ")    # 0 1 2 3 4
```

| `return` | `yield` |
|---|---|
| ends function, discards state | pauses function, **keeps** state |
| function usable normally | function becomes a generator — don't call it directly for a value |

```python
def powers_of_2(n):
    power = 1
    for i in range(n):
        yield power
        power *= 2

print(list(powers_of_2(4)))   # [1, 2, 4, 8]  <- list() drains a generator
print(3 in powers_of_2(4))    # True          <- generators work with `in` too
```

### 🔐 SOC relevance
```python
def tail_alerts(path):
    """Generator that yields new alert lines as they're appended —
    classic 'tail -f' style log monitor, without loading the whole file."""
    with open(path) as f:
        f.seek(0, 2)              # jump to end of file
        while True:
            line = f.readline()
            if line:
                yield line.strip()
```
Generators are memory-safe for continuously-growing security logs (SIEM feeds, firewall logs) — you process one event at a time instead of buffering everything.

---

## 3. List Comprehensions vs Generators

```python
the_list      = [x * 2 for x in range(5)]   # [ ] -> LIST, built fully in memory
the_generator = (x * 2 for x in range(5))   # ( ) -> GENERATOR, lazy, one at a time
```

| | List comprehension `[ ]` | Generator expression `( )` |
|---|---|---|
| Built | all at once, stored in memory | on demand, one value at a time |
| `len()` | works | ❌ `TypeError` — no length until exhausted |
| Reusable | yes, can iterate many times | ❌ single-use — exhausted after one pass |

### Conditional expression (ternary) — usable inside comprehensions
```python
value = 1 if x % 2 == 0 else 0          # "expr_if_true if condition else expr_if_false"
flags = [1 if x % 2 == 0 else 0 for x in range(10)]
```

### 🔐 SOC relevance
```python
# List: fine for a small, known alert batch
critical = [a for a in alerts if a["severity"] == "critical"]

# Generator: better for scanning a massive log stream — doesn't load it all
suspicious = (line for line in open("access.log") if "admin" in line)
```
Rule of thumb: use `[ ]` when you need the data multiple times or need `len()`; use `( )` when the source is huge or infinite (a live stream).

---

## 4. Lambda Functions

```python
lambda parameters: expression        # anonymous, one-expression function
```

```python
sqr = lambda x: x * x
pwr = lambda x, y: x ** y
print(sqr(4), pwr(2, 3))    # 16 8
```

> ⚠️ PEP 8: prefer `def f(x): return x*x` over `f = lambda x: x*x`. Lambdas are best used **inline**, unnamed, as arguments — not assigned to variables.

### `map(function, iterable)` — apply function to every element
```python
nums = [1, 2, 3]
doubled = list(map(lambda x: x * 2, nums))    # [2, 4, 6]
```

### `filter(function, iterable)` — keep elements where function returns True
```python
nums = [-3, -2, 0, 2, 5]
positives_even = list(filter(lambda x: x > 0 and x % 2 == 0, nums))  # [2]
```

### 🔐 SOC relevance
```python
ips = ["10.0.0.1", "203.0.113.5", "192.168.1.1", "8.8.8.8"]

# filter(): keep only external (non-private) IPs
external = list(filter(lambda ip: not ip.startswith(("10.","192.168.","172.")), ips))

# map(): mask the last octet of every IP before logging (privacy)
masked = list(map(lambda ip: ip.rsplit(".",1)[0] + ".xxx", ips))
print(external, masked)
```
`filter()` is exactly how you'd quickly separate internal vs external addresses in an IOC list; `map()` is handy for bulk-transforming fields (masking, normalizing case, hashing) across a whole dataset in one line.

---

## 5. Closures

A **closure** = an inner function that "remembers" variables from its enclosing function, even after that outer function has finished running.

```python
def outer(par):
    loc = par                 # local to outer()

    def inner():
        return loc             # inner "closes over" loc
    return inner                # return the function itself (not called!)

fun = outer(5)   # outer() has finished running...
print(fun())      # ...but loc is still remembered -> 5
```

```python
def make_closure(par):
    loc = par
    def power(p):
        return p ** loc
    return power

square = make_closure(2)   # remembers loc=2
cube   = make_closure(3)   # remembers loc=3
print(square(5), cube(5))  # 25 125
```

### 🔐 SOC relevance
```python
def make_threshold_checker(threshold):
    def check(value):
        return value > threshold      # "threshold" is remembered
    return check

is_brute_force = make_threshold_checker(5)   # >5 failed logins = alert
print(is_brute_force(7))     # True
print(is_brute_force(2))     # False
```
Closures let you generate many customized detection functions (different thresholds per rule/tenant) from one template, without rewriting the logic each time.

---

## 6. File Handling — Opening, Modes, Streams

```python
stream = open(file, mode='r', encoding=None)
```

| Mode | Meaning | File must exist? | Effect |
|---|---|---|---|
| `r` | read | ✅ yes | error if missing |
| `w` | write | ❌ no | creates new / **erases** existing content |
| `a` | append | ❌ no | creates new / writes to the end, keeps old content |
| `r+` | read+update | ✅ yes | both read & write allowed |
| `w+` | write+update | ❌ no | both read & write, erases first |
| `x` | exclusive create | ❌ must NOT exist | errors if file already exists |

Add to mode string:
| Suffix | Meaning |
|---|---|
| `t` (default) | text mode — handles `\n`/`\r\n` translation automatically |
| `b` | binary mode — raw bytes, no translation |

```python
try:
    stream = open("file.txt", "rt", encoding="utf-8")
    # ... process ...
    stream.close()
except IOError as exc:
    print("Cannot open file:", exc)
```

### Pre-opened streams (no need to open/close)
| Stream | Purpose |
|---|---|
| `sys.stdin` | keyboard input (used by `input()`) |
| `sys.stdout` | screen output (used by `print()`) |
| `sys.stderr` | error output — separate from normal results |

```python
import sys
sys.stderr.write("Something went wrong\n")   # send error separately from stdout
```

### Diagnosing failures
```python
import errno
try:
    stream = open("missing.txt", "r")
except IOError as exc:
    print(exc.errno)                     # numeric error code
    print(os.strerror(exc.errno))        # human-readable description
```

| `errno` constant | Meaning |
|---|---|
| `EACCES` | permission denied |
| `EEXIST` | file already exists |
| `EISDIR` | expected file, got a directory |
| `EMFILE` | too many open files |
| `ENOENT` | no such file/directory |
| `ENOSPC` | disk full |

### 🔐 SOC relevance
```python
import errno
try:
    stream = open("/var/log/auth.log", "r")
except IOError as exc:
    if exc.errno == errno.EACCES:
        print("Need elevated privileges to read this log")
    elif exc.errno == errno.ENOENT:
        print("Log file doesn't exist on this host")
```
Real automation scripts run across many hosts with inconsistent permissions/paths — checking `errno` lets a script give an actionable reason instead of just crashing.

---

## 7. Reading & Writing Text Files

| Method | Reads | Returns |
|---|---|---|
| `read()` | whole file | one string |
| `read(n)` | n characters | one string (empty `""` at EOF) |
| `readline()` | one line | one string (empty `""` at EOF) |
| `readlines()` | all lines | list of strings |
| `readlines(n)` | up to n bytes worth | list of strings |

```python
stream = open("text.txt")
for line in stream:            # file object is itself iterable, line by line!
    print(line, end='')
stream.close()                 # auto-closes at EOF too, but close explicitly anyway
```

```python
stream = open("newtext.txt", "w")
for i in range(1, 4):
    stream.write(f"line #{i}\n")   # write() does NOT add \n automatically
stream.close()
```

### 🔐 SOC relevance
```python
failed_logins = 0
for line in open("auth.log"):
    if "Failed password" in line:
        failed_logins += 1
print(f"{failed_logins} failed login attempts")
```
The "file object is iterable" pattern is the bread-and-butter idiom for scanning security logs line by line, memory-efficiently, in a single `for` loop.

---

## 8. Binary Files & `bytearray`

| Class | Mutable? | Use |
|---|---|---|
| `bytearray` | ✅ yes | build/modify raw byte data |
| `bytes` | ❌ no | immutable snapshot of raw byte data |

```python
data = bytearray(10)          # 10 bytes, all zero
data[0] = 255                 # must be int 0–255, else TypeError/ValueError

stream = open("out.bin", "wb")
stream.write(data)
stream.close()

buf = bytearray(10)
stream = open("out.bin", "rb")
stream.readinto(buf)          # fills an EXISTING bytearray (doesn't create new)
stream.close()
```

```python
stream = open("out.bin", "rb")
raw = stream.read()           # returns immutable `bytes`
data = bytearray(raw)         # convert to mutable if needed
stream.close()
```

### 🔐 SOC relevance
```python
# Read the first bytes of a suspicious file to check its "magic number" (file signature)
with open("suspicious.exe", "rb") as f:
    header = f.read(4)
    print(header.hex())   # e.g. '4d5a9000' -> MZ header = Windows executable
```
File-type/malware triage often starts with inspecting raw binary headers — `bytearray`/`bytes` and binary-mode reads are exactly the tool for that.

---

## 9. The `os` Module — OS Interaction

| Function | Purpose |
|---|---|
| `os.name` | `'posix'` (Unix/Mac), `'nt'` (Windows), `'java'` (Jython) |
| `os.uname()` | OS name, hostname, release, version, machine (Unix only) |
| `os.getcwd()` | current working directory |
| `os.chdir(path)` | change current working directory |
| `os.mkdir(path)` | create ONE directory (error if parent missing) |
| `os.makedirs(path)` | create directory **+ all missing parents** (recursive) |
| `os.rmdir(path)` | remove ONE empty directory |
| `os.removedirs(path)` | remove directory + empty parents (recursive) |
| `os.listdir(path='.')` | list files/dirs (excludes `.` and `..`) |
| `os.system(cmd)` | run a shell command, returns exit status |

```python
import os
os.mkdir("logs")                       # error if "logs" already exists
os.makedirs("logs/2024/archive")       # creates all 3 levels at once
print(os.listdir("logs"))              # ['2024']
os.chdir("logs")
print(os.getcwd())                     # .../logs
```

### 🔐 SOC relevance
```python
import os, datetime

today = datetime.date.today().isoformat()
os.makedirs(f"case_files/{today}", exist_ok=True)   # organize incident evidence by date
os.system(f"cp raw_capture.pcap case_files/{today}/")
```
Incident-response tooling constantly needs to organize evidence/log dumps into date- or case-based folder structures — `makedirs`/`system` automate that instead of manual folder juggling.

---

## 10. The `datetime` Module

| Class | Represents |
|---|---|
| `date` | year, month, day |
| `time` | hour, minute, second, microsecond |
| `datetime` | date + time combined |
| `timedelta` | a **duration** (difference between two dates/times) |

```python
from datetime import date, time, datetime, timedelta

d = date(2024, 1, 15)                  # explicit date
today = date.today()                   # current local date
d2 = date.fromisoformat("2024-01-15")  # from "YYYY-MM-DD" string

t = time(14, 30, 0)                    # 14:30:00

dt = datetime(2024, 1, 15, 14, 30)     # date + time combined
now = datetime.now()                   # current local date+time
utc_now = datetime.utcnow()            # current UTC date+time
```

| Attribute/Method | Result |
|---|---|
| `d.year / .month / .day` | read-only components |
| `d.weekday()` | 0=Monday ... 6=Sunday |
| `d.isoweekday()` | 1=Monday ... 7=Sunday |
| `d.replace(year=...)` | returns a NEW date with one field changed |
| `dt.timestamp()` | seconds since Unix epoch (float) |

### Formatting / Parsing
```python
d.strftime("%Y/%m/%d")                                 # date -> string
datetime.strptime("2024/01/15 14:30", "%Y/%m/%d %H:%M") # string -> datetime
```

| Directive | Meaning |
|---|---|
| `%Y` | year with century (2024) |
| `%y` | year without century (24) |
| `%m` | month, zero-padded (01–12) |
| `%d` | day, zero-padded (01–31) |
| `%B` | full month name (January) |
| `%H` | hour 24h, zero-padded |
| `%M` | minute, zero-padded |
| `%S` | second, zero-padded |

### `timedelta` — date/time arithmetic
```python
delta = date(2024, 1, 15) - date(2023, 1, 15)   # timedelta: 365 days
future = date.today() + timedelta(days=30)       # add 30 days
delta2 = timedelta(weeks=1, hours=3)             # explicit duration
```

### 🔐 SOC relevance
```python
from datetime import datetime, timedelta

alert_time = datetime.strptime("2024-01-15 09:00:00", "%Y-%m-%d %H:%M:%S")
now = datetime.now()

if now - alert_time > timedelta(hours=24):
    print("SLA breached — alert older than 24h and still open")

print(alert_time.strftime("%A, %B %d %Y"))   # "Monday, January 15 2024"
```
Almost every alert/ticket has a timestamp — `datetime` parsing + `timedelta` comparisons power SLA checks, "time since last seen" logic, and log timeline correlation.

---

## 11. The `time` Module (companion to `datetime`)

| Function | Purpose |
|---|---|
| `time.time()` | current Unix timestamp (float, seconds since epoch) |
| `time.sleep(n)` | pause execution for n seconds |
| `time.ctime([secs])` | timestamp → human-readable string |
| `time.gmtime([secs])` | timestamp → `struct_time` in UTC |
| `time.localtime([secs])` | timestamp → `struct_time` in local time |
| `time.strftime(fmt, struct_time)` | format a `struct_time` as string |
| `time.strptime(str, fmt)` | parse string → `struct_time` |
| `time.mktime(struct_time)` | `struct_time` → Unix timestamp |

```python
import time
ts = time.time()
print(time.ctime(ts))        # 'Mon Nov  4 14:53:00 2024'
time.sleep(2)                 # pause 2 seconds
```

### 🔐 SOC relevance
```python
import time
for attempt in range(3):
    # retry a flaky threat-intel API call with backoff
    try:
        response = query_api()
        break
    except ConnectionError:
        time.sleep(2 ** attempt)   # 1s, 2s, 4s backoff
```
`time.sleep()` with exponential backoff is standard practice when polling rate-limited security APIs (VirusTotal, AbuseIPDB, etc.).

---

## 12. The `calendar` Module

| Function | Purpose |
|---|---|
| `calendar.calendar(year)` | full-year text calendar |
| `calendar.month(year, month)` | single-month text calendar |
| `calendar.prcal(year)` / `prmonth(y,m)` | same, but auto-prints (no `print()` needed) |
| `calendar.weekday(y, m, d)` | day-of-week as int (0=Monday) for a specific date |
| `calendar.weekheader(width)` | short weekday name header, e.g. `"Mo Tu We..."` |
| `calendar.isleap(year)` | True/False — leap year check |
| `calendar.leapdays(y1, y2)` | count of leap years in `[y1, y2)` |
| `calendar.setfirstweekday(day)` | change week start (default Monday=0) |

```python
import calendar
print(calendar.isleap(2024))          # True
print(calendar.weekday(2024, 1, 15))  # 0 -> Monday
calendar.setfirstweekday(calendar.SUNDAY)
```

### `Calendar` class (object-based, more flexible)
```python
c = calendar.Calendar(firstweekday=0)
for wd in c.iterweekdays():
    print(wd, end=" ")                 # 0 1 2 3 4 5 6 (Mon..Sun)

for d in c.itermonthdates(2024, 1):    # datetime.date objects, padded to full weeks
    print(d)
```

### 🔐 SOC relevance
```python
import calendar
# Was a suspicious login on a weekend? (higher-risk pattern)
login_date = (2024, 1, 13)
if calendar.weekday(*login_date) >= 5:   # 5=Saturday, 6=Sunday
    print("⚠️ Off-hours weekend login — flag for review")
```
Weekday/weekend detection is a simple but real heuristic used in behavioral anomaly detection (logins/activity outside normal business patterns).

---

## Quick-Reference Summary

| Concept | Key Syntax |
|---|---|
| Iterator protocol | `__iter__(self): return self` / `__next__(self): ... raise StopIteration` |
| Generator function | use `yield` instead of `return` — pauses & resumes |
| List comp vs generator | `[ ]` = eager list, `( )` = lazy, one-shot, no `len()` |
| Lambda | `lambda x: expr` — anonymous, single-expression |
| `map(f, iterable)` | apply `f` to every element |
| `filter(f, iterable)` | keep elements where `f` returns True |
| Closure | inner function "remembers" outer function's variables after outer returns |
| Open file | `open(path, mode, encoding=...)` — modes `r/w/a/r+/w+/x` + `t`/`b` |
| Read | `read()`, `read(n)`, `readline()`, `readlines()`, or just `for line in stream:` |
| Write | `stream.write(text)` — no auto `\n` |
| Binary data | `bytearray` (mutable) vs `bytes` (immutable); `readinto()` for binary read |
| Diagnose I/O errors | `except IOError as e: e.errno`, compare to `errno.*` constants |
| `os` module | `getcwd/chdir/mkdir/makedirs/rmdir/removedirs/listdir/system/name/uname` |
| `datetime` module | `date`, `time`, `datetime`, `timedelta`; `.strftime()`/`.strptime()` for format ↔ string |
| `time` module | `time()`, `sleep()`, `ctime()`, `gmtime()/localtime()`, `struct_time` |
| `calendar` module | `calendar()`, `month()`, `weekday()`, `isleap()`, `Calendar` class + `iterweekdays()`/`itermonthdates()` |
