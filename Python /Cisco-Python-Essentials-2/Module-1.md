# Python Essentials 2 – Module 1: Modules, Packages & PIP

---

## 1. What is a Module?

A **module** = a `.py` file containing functions, variables, classes — reusable code you `import` instead of rewriting.

**Why modules exist:** large code → hard to maintain → split into smaller cooperating files (decomposition).

### Importing a module

| Syntax | Effect | Access style |
|---|---|---|
| `import math` | imports whole module | `math.pi`, `math.sin()` |
| `from math import pi` | imports only `pi` | `pi` (no prefix) |
| `from math import *` | imports everything | risky — name clashes |
| `import math as m` | aliases module | `m.pi` |
| `from math import pi as PI` | aliases entity | `PI` |

```python
import math
print(math.pi)          # 3.141592653589793

from math import sin, pi
print(sin(pi/2))         # 1.0
```

> ⚠️ `from module import *` is discouraged — you don't know every name it brings in, risking silent overwrites.

**Namespace rule:** Names inside a module don't collide with your own names unless you `from module import *` or `from module import name`. Qualified access (`math.pi`) always stays safe.

### 🔐 SOC relevance
Analysts write one-off automation scripts (parse logs, query APIs, hash files) constantly. Keeping reusable helpers (IOC lookups, alert formatting) in a module avoids copy-pasting the same function into every script.

```python
# soc_utils.py
def is_private_ip(ip):
    return ip.startswith(("10.", "192.168.", "172.16."))

# triage.py
import soc_utils
print(soc_utils.is_private_ip("10.0.0.5"))   # True
```

---

## 2. Key Standard Modules

### `dir()` — inspect a module's contents
```python
import math
print(dir(math))   # lists all names inside math
```
Requires `import module` (not `from module import x`).

### `math` — numeric operations
| Function | Purpose |
|---|---|
| `sin/cos/tan(x)` | trig (radians) |
| `radians(x)/degrees(x)` | angle conversion |
| `log(x)`, `log2`, `log10` | logarithms |
| `ceil/floor/trunc(x)` | rounding variants |
| `factorial(x)` | x! |
| `hypot(x,y)` | precise Euclidean distance |
| `pi`, `e` | constants |

```python
import math
print(math.ceil(4.1), math.floor(4.9), math.trunc(-4.9))  # 5 4 -4
```

**SOC relevance:** `math.log2()` is used to calculate **Shannon entropy** of strings — a classic technique to flag suspicious high-entropy domains (DGA malware) or base64-encoded payloads.

```python
import math
from collections import Counter

def entropy(s):
    counts = Counter(s)
    return -sum((c/len(s)) * math.log2(c/len(s)) for c in counts.values())

print(entropy("aaaaaaaa"))     # low entropy → normal
print(entropy("x9F$kQ2!zL"))   # high entropy → possible obfuscation/DGA
```

### `random` — pseudo-random values
| Function | Purpose |
|---|---|
| `random()` | float in `[0.0, 1.0)` |
| `seed(x)` | fixes sequence (reproducible) |
| `randrange(a,b)` / `randint(a,b)` | random int (randint is inclusive on both ends) |
| `choice(seq)` | one random element |
| `sample(seq, n)` | n unique random elements |

```python
import random
random.seed(42)
print(random.randint(1, 100))
print(random.sample(range(1, 50), 6))   # e.g. lottery numbers
```

**SOC relevance:** generating unique correlation IDs for alerts, or **jittering** simulated/test traffic timing so synthetic SOC test-lab traffic doesn't look perfectly periodic (which itself is an anomaly indicator).

```python
import random, time
for _ in range(3):
    time.sleep(random.uniform(0.5, 2.0))   # jittered beacon simulation
    print("test event sent")
```

### `platform` — environment/host info
| Function | Returns |
|---|---|
| `platform()` | full OS/host string |
| `system()` | OS name (`Windows`, `Linux`) |
| `machine()` | CPU architecture |
| `processor()` | CPU name |
| `python_implementation()` | e.g. `CPython` |
| `python_version_tuple()` | `(major, minor, patch)` |

```python
from platform import system, machine
print(system(), machine())   # Linux x86_64
```

**SOC relevance:** incident-response and asset-inventory scripts fingerprint the host they're running on (or that an agent reports from) before applying OS-specific triage logic.

```python
from platform import system
if system() == "Windows":
    print("Pull Windows Event Log")
else:
    print("Pull /var/log/auth.log")
```

---

## 3. Modules → Packages

A **package** = a folder of modules, made importable by an `__init__.py` file inside it (can be empty, but must exist).

```
extra/
├── __init__.py
├── good/
│   ├── __init__.py
│   ├── alpha.py
│   └── best/
│       ├── __init__.py
│       ├── sigma.py
│       └── tau.py
└── iota.py
```

```python
import extra.iota
extra.iota.funI()

from extra.good.best import sigma
sigma.funS()

import extra.good.alpha as alp   # aliasing works the same as modules
```

### `__name__` — detect how a file was launched
```python
if __name__ == "__main__":
    print("running directly (tests go here)")
else:
    print("imported as a module")
```
Run directly → `__name__ == "__main__"`. Imported → `__name__ == "<module name>"`.

### `sys.path` — where Python looks for modules
```python
import sys
sys.path.append('../modules')   # add a custom search location
import module
```
First match wins; folder you launched from is always path[0]. Also works with `.zip` archives.

### Private-by-convention names
Prefix with `_` or `__` to signal "internal, don't touch" — Python doesn't enforce this, it's a courtesy convention.

### 🔐 SOC relevance
Real SOC tooling is organized as packages: `soc_toolkit/log_parsers/`, `soc_toolkit/enrichment/`, `soc_toolkit/alerting/` — each a sub-package, imported the same way as `extra.good.alpha` above. `sys.path.append()` is commonly used to make a shared internal toolkit importable from any analyst's script without installing it.

---

## 4. PIP — Python Package Installer

**PyPI** (pypi.org, aka "The Cheese Shop") = the public repository of packages.
**pip** = the CLI tool that downloads/installs/manages them (handles dependency resolution automatically — no manual "dependency hell").

### Core commands

| Command | Purpose |
|---|---|
| `pip --version` | check pip is installed / which Python it's tied to |
| `pip list` | show installed packages |
| `pip show package_name` | version, dependencies, "Required-by" |
| `pip install name` | install (system-wide, needs admin) |
| `pip install --user name` | install for current user only |
| `pip install -U name` | upgrade to latest |
| `pip install name==1.9.2` | install a specific version |
| `pip uninstall name` | remove a package |

```bash
pip install requests
pip show requests
pip install -U requests
pip uninstall requests
```

> On some Linux systems, use `pip3`/`python3 -m pip` if both Python 2 and 3 coexist.

### 🔐 SOC relevance
Almost every SOC automation script depends on third-party packages installed via pip:

```bash
pip install requests pandas scapy pyshark
```
```python
import requests

def check_ip_reputation(ip, api_key):
    url = f"https://api.abuseipdb.com/api/v2/check?ipAddress={ip}"
    headers = {"Key": api_key, "Accept": "application/json"}
    return requests.get(url, headers=headers).json()
```
`requests` → threat-intel API calls, `pandas` → log/CSV analysis, `scapy`/`pyshark` → packet inspection. None of these ship with core Python — pip is how they get onto the analyst's machine.

---

## Quick-Reference Summary

- **Module** = single `.py` file of reusable code → `import`, `from...import`, `as` for aliasing.
- **Namespace** = qualified names (`module.entity`) avoid collisions; unqualified imports (`from x import y`) can silently override your own names.
- **`math`** → numeric/trig/log ops (`log2` → entropy checks for DGA/obfuscation detection).
- **`random`** → pseudo-random values, seedable (`seed()` for reproducibility; jittered timing in traffic simulation).
- **`platform`** → OS/hardware fingerprinting (branch triage logic by host OS).
- **Package** = folder + `__init__.py`, imported with dot notation (`extra.good.alpha`); `sys.path` controls where Python searches.
- **`__name__ == "__main__"`** → separates "run directly" (tests) from "imported" behavior.
- **pip** → installs from PyPI, resolves dependencies; `install / show / list / uninstall / -U / --user` are the commands to know cold for an interview.
