# Python

Course 8 of 15. Previous: [07-node-express.md](07-node-express.md). Next: [09-flask.md](09-flask.md).

Python is a language with significant indentation. This course is the language. Flask is the next course. Examples here run with `python3 file.py`.

## Contents

1. [Syntax and names](#1-syntax-and-names)
2. [Types and operators](#2-types-and-operators)
3. [Strings](#3-strings)
4. [Control flow](#4-control-flow)
5. [Functions](#5-functions)
6. [Lists, tuples, sets, and dicts](#6-lists-tuples-sets-and-dicts)
7. [Classes](#7-classes)
8. [Modules, files, and errors](#8-modules-files-and-errors)

## 1. Syntax and names

Indentation is the block. Use four spaces. A colon opens the block.

```python
def greet(name):
    message = f"Hello, {name}"
    return message
```

| Rule | Detail |
| --- | --- |
| `#` | A comment to the end of the line |
| Names | `snake_case` for functions and variables. `PascalCase` for classes |
| `None` | The absence value |
| Truthiness | `None`, `0`, `0.0`, `""`, empty containers are false |
| Assignment | `city = "Irvine"` binds a name. It does not declare a type |

## 2. Types and operators

| Type | Example |
| --- | --- |
| `int` | `999000` |
| `float` | `2.5` |
| `bool` | `True`, `False`. Capitalized |
| `str` | `"Irvine"` |
| `list` | `[1, 2]` |
| `tuple` | `(1, 2)` |
| `set` | `{1, 2}` |
| `dict` | `{"city": "Irvine"}` |
| `NoneType` | `None` |

```python
price = 999_000
baths = 2.5
label = f"{price:,}"
```

| Operator | Notes |
| --- | --- |
| `+ - * /` | `/` always makes a float |
| `//` | Floor division |
| `%` | Remainder |
| `**` | Power |
| `==` | Value equality |
| `is` | Same object. Use it for `None`: `value is None` |
| `and` / `or` / `not` | Return operands, not only booleans. `0 or "missing"` is `"missing"` |
| `in` | Membership |

`int("1,500".replace(",", ""))` is `1500`. `float("2.5")` is `2.5`. A bad string raises `ValueError`.

## 3. Strings

Strings are immutable sequences of characters.

```python
city = "  Los Angeles  "
city.strip().lower()
city.split()
",".join(["a", "b"])
"beds" in "3 beds"
"hoa".replace("h", "H")
```

| Method | Result |
| --- | --- |
| `lower` / `upper` / `title` / `capitalize` | A new string |
| `casefold` | A new string for case-insensitive comparison. Stronger than `lower` |
| `strip` / `lstrip` / `rstrip` | Ends stripped. An argument strips those characters, not a general trim of a word |
| `removeprefix` / `removesuffix` | Those exact ends removed, or the original string |
| `split` | A list. No argument splits on whitespace and drops empty pieces |
| `splitlines` | A list of lines. The line breaks are dropped |
| `join` | Called on the separator: `",".join(items)` |
| `startswith` / `endswith` | Boolean. A tuple of options is allowed |
| `find` / `rfind` | Index, or `-1`. Search from the left or the right |
| `index` / `rindex` | The same search. Raises `ValueError` if it is missing |
| `replace` | A new string. A count argument limits replacements |
| `count` | How many non-overlapping times the piece appears |
| `partition` / `rpartition` | A 3-tuple: before, the separator, after |
| `isdigit` / `isalpha` / `isalnum` / `isspace` | True if every character is in that class, and the string is not empty |
| `zfill(n)` | Zero-pad on the left until length `n` |
| `ljust` / `rjust` / `center` | Pad to a width |
| Slices | `"Irvine"[0:3]` is `"Irv"`. The end is excluded |

An f-string evaluates expressions: `f"{price / 1000:.0f}k"`.

A regular expression is a pattern. This is enough to read one.

```python
import re

text = "3 beds in Irvine"
found = re.search(r"(\d+) beds", text)
beds = int(found.group(1)) if found else None
```

| Piece | Meaning |
| --- | --- |
| `r"..."` | A raw string. `\d` stays `\d` |
| `\d+` | One or more digits |
| `(...)` | A group. `group(1)` is the text it captured |
| `re.search` | The first match anywhere in the text, or `None` |
| `re.fullmatch` | The whole string must match, or `None` |

`found` is `None` when the pattern is missing. Calling `.group` on `None` raises. Check first.

## 4. Control flow

```python
if beds >= 3:
    label = "large"
elif beds == 2:
    label = "mid"
else:
    label = "small"

for city in ["Irvine", "Tustin"]:
    print(city)

for i in range(3):
    print(i)

n = 3
while n > 0:
    n -= 1
```

| Tool | Meaning |
| --- | --- |
| `elif` | Else if |
| `range(stop)` | `0` through `stop - 1` |
| `range(start, stop, step)` | The stop is excluded |
| `break` / `continue` | Leave, or skip to the next pass |
| `for` / `else` | The `else` runs when the loop did not `break` |
| `match` | Structural match, like a switch, in modern Python |

```python
match status:
    case "approved" | "rejected":
        print("decided")
    case _:
        print("waiting")
```

## 5. Functions

```python
def to_number(digits, unit=""):
    n = float(str(digits).replace(",", ""))
    unit = unit.lower()
    if unit in ("k", "thousand"):
        n *= 1000
    if unit in ("m", "million"):
        n *= 1_000_000
    return round(n)
```

| Idea | Syntax |
| --- | --- |
| Default argument | `unit=""`. The default is evaluated once, at definition. Do not use a list or dict as a default |
| Keyword call | `to_number("1.5", unit="m")` |
| `*args` | Extra positional arguments as a tuple |
| `**kwargs` | Extra keyword arguments as a dict |
| `lambda` | `lambda n: n * 2`. One expression |
| Docstring | A string as the first statement. `help(fn)` shows it |
| Type hints | `def f(n: int) -> str`. Not enforced unless you run a checker |
| `return` | No return means the result is `None` |

A nested function can read names from the outer function. To assign an outer name, declare `nonlocal`. To assign a module global, declare `global`. Prefer returning values over `global`.

## 6. Lists, tuples, sets, and dicts

### List

A list is ordered and mutable. Index `0` is first. A negative index counts from the end.

```python
prices = [999000, 1100000]
prices.append(1350000)
prices.insert(0, 800000)
last = prices.pop()
prices.sort()
copy = prices[:]
```

| Operation | Code |
| --- | --- |
| Length | `len(prices)` |
| Slice | `prices[1:3]`. End excluded. `prices[::-1]` reverses |
| Membership | `999000 in prices` |
| Extend | `prices.extend(more)` |
| Remove by value | `prices.remove(999000)`. Raises if missing |
| Count | `prices.count(999000)` |
| Sort a copy | `sorted(prices)` |
| Comprehension | `[p for p in prices if p > 1000000]` |
| Map-like | `[p / 1000 for p in prices]` |

`sort` mutates. It accepts `key=` and `reverse=True`.

### Tuple

A tuple is ordered and immutable.

```python
pair = ("Irvine", 999000)
city, price = pair
one = (1,)
```

| Use a tuple | Use a list |
| --- | --- |
| A fixed record, such as a pair | A collection that grows |
| A dict key | Items you will sort or filter in place |

The comma builds the tuple. Parentheses only group. `(1)` is an int. `(1,)` is a tuple.

### Set

A set stores unique values. It is unordered. Items must be hashable, so a list cannot be a member. A tuple of hashable items can.

```python
cities = {"Irvine", "Tustin", "Irvine"}
cities.add("San Diego")
cities.discard("Tustin")  # no error if missing
"Irvine" in cities
```

| Operation | Result |
| --- | --- |
| `\|` or `union` | Either set |
| `&` or `intersection` | Both |
| `-` or `difference` | Left only |
| `^` | In one but not both |
| `set(items)` | Unique items from a list |
| Comprehension | `{c.lower() for c in cities}` |

`remove` raises `KeyError` if the item is missing. `discard` does not.

### Dict

A dict maps a key to a value. Keys are unique and hashable. Insertion order is preserved.

```python
card = {"city": "Irvine", "price": 999000}
card["beds"] = 3
price = card.get("price")
missing = card.get("hoa")  # None
card.get("hoa", 0)
```

| Operation | Code |
| --- | --- |
| Keys / values / pairs | `card.keys()`, `.values()`, `.items()` |
| Pop | `card.pop("beds", None)` |
| Merge | `card \| {"price": 1}` in modern Python. A new dict |
| Update | `card.update(other)` |
| Comprehension | `{k: v for k, v in card.items() if v}` |
| Delete | `del card["beds"]` |

`card["hoa"]` raises `KeyError` when the key is missing. `.get` does not.

### When to pick which

| Need | Type |
| --- | --- |
| Ordered, duplicates, index | `list` |
| Fixed record | `tuple` |
| Unique items, fast membership | `set` |
| Key to value | `dict` |
| A queue or a stack of many end-inserts | `collections.deque` |
| Count items | `collections.Counter` |
| A dict that builds a list for a missing key | `collections.defaultdict(list)` |

```python
from collections import Counter, defaultdict, deque

counts = Counter(["Irvine", "Irvine", "Tustin"])
counts["Irvine"]  # 2

line = deque()
line.append("a")
line.appendleft("b")
first = line.popleft()

groups = defaultdict(list)
groups["Irvine"].append(999000)
```

There is no `LinkedList` class in the usual course. `deque` is the double-ended queue. A dict is the hash map.

The same jobs, lined up:

| Job | `list` | `tuple` | `set` | `dict` |
| --- | --- | --- | --- | --- |
| Create | `[1, 2]` | `(1, 2)` or `1, 2` | `{1, 2}`. Empty is `set()` | `{"a": 1}` |
| Length | `len` | `len` | `len` | `len`. Counts keys |
| Read | `items[i]` | `items[i]` | No index | `d[key]` or `d.get(key, default)` |
| Membership | `x in items` | `x in items` | `x in items` | `key in d`. Tests keys, not values |
| Add | `append`, `insert`, `extend` | No | `add`, `update` | `d[key] = value`, `update`, `setdefault` |
| Remove | `pop`, `remove`, `del items[i]` | No | `remove` raises, `discard` does not, `pop` | `pop(key, default)`, `del d[key]` |
| Order | Index order | Index order | None you can rely on | Insertion order |
| Duplicates | Yes | Yes | No | One value per key |
| Mutable | Yes | No | Yes | Yes |
| Usable as a dict key | No | Yes, if every item is hashable | No | No |
| Slice | `items[1:3]` | `items[1:3]`, a new tuple | No | No |
| Sort | `sort` in place, or `sorted` | `sorted` returns a list | `sorted` returns a list | `sorted(d)` sorts the keys |

## 7. Classes

```python
class Listing:
    def __init__(self, city, price):
        self.city = city
        self.price = price

    def label(self):
        return f"{self.city}: {self.price}"

class Condo(Listing):
    def __init__(self, city, price, hoa):
        super().__init__(city, price)
        self.hoa = hoa

home = Listing("Irvine", 999000)
home.label()
```

| Piece | Meaning |
| --- | --- |
| `__init__` | The constructor. The new object arrives as `self` |
| `self` | Explicit. Always the first parameter of an instance method |
| `super()` | The parent |
| Inheritance | `class Condo(Listing)` |
| `_name` | A convention: internal. Not enforced |
| `__str__` | `str(obj)` and `print` |
| `__repr__` | The developer-facing text |
| `@dataclass` | Generates `__init__` and equality for a field list |

```python
from dataclasses import dataclass

@dataclass
class Card:
    city: str
    price: int
```

## 8. Modules, files, and errors

```python
import json
from pathlib import Path

text = Path("notes.txt").read_text()
Path("notes.txt").write_text(text + "\n")
data = json.loads(text)
```

| Tool | Role |
| --- | --- |
| `import math` | A module. Use `math.floor` |
| `from pathlib import Path` | One name |
| `pip install` | Install a third-party package into the environment |
| `venv` | A private set of packages: `python3 -m venv venv` |
| `if __name__ == "__main__"` | Code that runs only when this file is the program, not when it is imported |

```python
def parse_price(text):
    try:
        return int(text.replace(",", ""))
    except ValueError as error:
        raise ValueError("price must be a number") from error
    finally:
        pass
```

| Exception | Usual cause |
| --- | --- |
| `ValueError` | The right type, an unacceptable value |
| `TypeError` | The wrong type |
| `KeyError` | A missing dict key |
| `IndexError` | A bad list index |
| `FileNotFoundError` | The path is not there |
| `ZeroDivisionError` | Division by zero |

`raise` without a value, inside `except`, re-raises the current exception. `from error` keeps the cause.

A `with` block closes the file even if a line raises:

```python
with open("notes.txt", encoding="utf-8") as handle:
    for line in handle:
        print(line.rstrip("\n"))
```

JSON: `json.dumps(obj)` makes text. `json.loads(text)` makes objects. Dict keys become strings. Tuples become lists.

## Drill

1. Write `unique(items)` using a set, and return a list.
2. Count cities with `Counter`.
3. Read a missing dict key with `.get` and with `[]`, and say which one raises.
4. Say what `re.search` returns when the pattern is not in the text.
