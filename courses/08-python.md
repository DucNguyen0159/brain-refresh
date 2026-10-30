# Python

Course 8 of 15. Previous: [07-node-express.md](07-node-express.md). Next: [09-flask.md](09-flask.md).

Python is a language with significant indentation. This course is the language. Flask is the next course. Examples here run with `python3 file.py`.

Every section has the same parts: the concept, the terms, the methods, several cases with what prints, common mistakes, a tip, and a practice with a solution.

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

### Concept

Indentation is the block. Use four spaces. A colon opens the block. A name such as `city = "Irvine"` binds a value. It does not declare a type. The same name can later point at a value of another type.

### Terms

| Term | Meaning |
| --- | --- |
| Block | The indented lines that belong to a `def`, `if`, `for`, `while`, `class`, or `with` |
| Colon | The `:` that opens a block |
| Comment | `#` through the end of the line. The interpreter skips it |
| `snake_case` | The usual style for functions and variables |
| `PascalCase` | The usual style for classes |
| `None` | The absence value. Its type is `NoneType` |
| Truthiness | How `if` decides. `None`, `0`, `0.0`, `""`, and empty containers are false |
| Binding | A name pointing at an object. Assignment rebinds the name |

### Methods

Syntax has no method list. These are the forms you type.

| Form | What it does |
| --- | --- |
| `def greet(name):` | Opens a function. The next lines are indented |
| `city = "Irvine"` | Binds the name. No type is written on the name |
| `# note` | A comment |
| `if value:` | Runs the block when the value is truthy |
| `f"Hello, {name}"` | An f-string. The braces are evaluated |

### Examples

**Case: four spaces and a colon**

```python
def greet(name):
    message = f"Hello, {name}"
    return message

print(greet("Irvine"))
```

Result: `Hello, Irvine`

Why: the two lines under `def` are the function body because they are indented. The colon is required.

**Case: a comment is not code**

```python
city = "Irvine"  # the city name
# print("hidden")
print(city)
```

Result: `Irvine`

Why: `#` hides the rest of that line. The second print never runs.

**Case: empty values are false**

```python
for value in (None, 0, 0.0, "", [], {}):
    print("yes" if value else "no")
```

Result:

```text
no
no
no
no
no
no
```

Why: `if` uses truthiness. Those six values are false. A non-empty string, a non-zero number, and a container with items are true.

**Case: assignment does not lock the type**

```python
city = "Irvine"
print(city)
city = 3
print(city)
```

Result:

```text
Irvine
3
```

Why: the name `city` is a binding. The second assignment points it at an int. Python does not store a declared type on the name.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Mixing tabs and spaces | `TabError`, or a block that is not the block you see | Four spaces, and an editor that inserts spaces |
| Forgetting the colon | `SyntaxError` before the program runs | Put `:` at the end of `def`, `if`, `for`, `while`, and `class` |
| Indenting a line that should be outside | The line becomes part of the block | Unindent it to the same column as the `def` or `if` |
| Treating `None` as `0` or `""` | A later method call fails, or a count is wrong | Test with `value is None` |

### Tip

If a block looks right and still fails, show whitespace. The structure of the program is the indentation, not the braces.

### Practice

Write a function `label(city)` that returns `City: Irvine` when called with `"Irvine"`. Then bind `count = 0` and print whether `count` is truthy.

### Solution

```python
def label(city):
    return f"City: {city}"

print(label("Irvine"))
count = 0
print("yes" if count else "no")
```

Result:

```text
City: Irvine
no
```

## 2. Types and operators

### Concept

Every value has a type. `True` and `False` are capitalized. `/` always makes a float. `//` is floor division. `and`, `or`, and `not` return operands, not only booleans, so `0 or "missing"` is `"missing"`. `==` compares values. `is` asks whether two names are the same object. Use `is` for `None`.

### Terms

| Type | Example | Notes |
| --- | --- | --- |
| `int` | `999000` | Underscores are allowed: `999_000` is the same number |
| `float` | `2.5` | A decimal point |
| `bool` | `True`, `False` | Capitalized. A subclass of `int` |
| `str` | `"Irvine"` | Text |
| `list` | `[1, 2]` | Ordered and mutable |
| `tuple` | `(1, 2)` | Ordered and immutable |
| `set` | `{1, 2}` | Unique items |
| `dict` | `{"city": "Irvine"}` | Key to value |
| `NoneType` | `None` | Absence |

### Methods

Operators are not methods. These are the ones used in this course.

| Operator | Result |
| --- | --- |
| `+ - *` | Arithmetic on numbers. `+` on strings concatenates |
| `/` | Division. Always a float. `7 / 2` is `3.5` |
| `//` | Floor division. `7 // 2` is `3` |
| `%` | Remainder. `7 % 2` is `1` |
| `**` | Power. `2 ** 3` is `8` |
| `==` | Value equality |
| `is` | Same object. `value is None` |
| `and` / `or` / `not` | Return operands. `0 or "missing"` is `"missing"`. `1 and "Irvine"` is `"Irvine"` |
| `in` | Membership. `"Irv" in "Irvine"` is true |
| `int("1,500".replace(",", ""))` | `1500`. A bad string raises `ValueError` |
| `float("2.5")` | `2.5`. A bad string raises `ValueError` |
| `f"{price:,}"` | Grouped digits. `999000` prints as `999,000` |

### Examples

**Case: division, floor, remainder, and power**

```python
print(7 / 2)
print(7 // 2)
print(7 % 2)
print(2 ** 3)
```

Result:

```text
3.5
3
1
8
```

Why: `/` never returns an int. `//` drops the fractional part toward negative infinity for positives here, so the result is `3`.

**Case: `or` returns the operand, and `is` checks identity**

```python
print(0 or "missing")
print(1 and "Irvine")
print(None is None)
print(999000 == 999000)
print("Irv" in "Irvine")
```

Result:

```text
missing
Irvine
True
True
True
```

Why: `or` stops at the first truthy operand and returns that operand. `is` is the test you want for `None`. `==` compares values.

**Case: underscores and a thousands separator**

```python
price = 999_000
baths = 2.5
print(price)
print(baths)
print(f"{price:,}")
print(type(True).__name__)
print(type(None).__name__)
```

Result:

```text
999000
2.5
999,000
bool
NoneType
```

Why: `999_000` is an int. The underscores are for readers. The f-string format `:,` inserts separators. It does not change `price`.

**Case: parsing text into numbers**

```python
print(int("1,500".replace(",", "")))
print(float("2.5"))
try:
    int("2.5")
except ValueError as error:
    print(type(error).__name__)
```

Result:

```text
1500
2.5
ValueError
```

Why: `int` accepts a whole-number string. `"2.5"` is a float string, so `int` raises `ValueError`. Strip the comma before you parse a grouped number. `float("2.5")` works.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Expecting `7 / 2` to be `3` | The result is `3.5` | Use `//` when you want the floor |
| `value == None` | Works for `None`, and misses the usual style | `value is None` |
| `int("1,500")` | `ValueError` | Remove the commas, then convert |
| `int("2.5")` | `ValueError` | `float("2.5")`, or `int(float("2.5"))` when you mean to drop the fraction |

### Tip

Print `type(value).__name__` when a later line fails. The failure is often a `str` that you meant to parse.

### Practice

Bind `price = 999_000`. Print `price / 1000`, `price // 1000`, and the parsed int of `"1,500"`.

### Solution

```python
price = 999_000
print(price / 1000)
print(price // 1000)
print(int("1,500".replace(",", "")))
```

Result:

```text
999.0
999
1500
```

## 3. Strings

### Concept

A string is an immutable sequence of characters. Methods return a new string. Slices exclude the end index. `"Irvine"[0:3]` is `"Irv"`. An f-string evaluates expressions: `f"{price / 1000:.0f}k"`.

A regular expression is a pattern. `re.search` finds the first match anywhere, or returns `None`.

### Terms

| Term | Meaning |
| --- | --- |
| Immutable | `city.upper()` builds a new string. It does not edit `city` |
| Index | `0` is the first character. A negative index counts from the end |
| Slice | `text[start:end]`. `end` is excluded |
| f-string | `f"{expression}"` converts the expression and inserts it |
| Raw string | `r"\d"` keeps the backslash. The pattern sees `\d` |
| Group | Parentheses in a pattern. `group(1)` is the first capture |
| `None` match | The pattern was not found. The match object has no `.group` |

### Methods

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
| `in` | `"beds" in "3 beds"` is true |

| Pattern piece | Meaning |
| --- | --- |
| `r"..."` | A raw string. `\d` stays `\d` |
| `\d+` | One or more digits |
| `(...)` | A group. `group(1)` is the text it captured |
| `re.search` | The first match anywhere in the text, or `None` |
| `re.fullmatch` | The whole string must match, or `None` |

### Examples

**Case: strip, split, join, membership, and replace**

```python
city = "  Los Angeles  "
print(city.strip().lower())
print(city.split())
print(",".join(["a", "b"]))
print("beds" in "3 beds")
print("hoa".replace("h", "H"))
```

Result:

```text
los angeles
['Los', 'Angeles']
a,b
True
Hoa
```

Why: `strip` and `lower` return new strings. `split` with no argument splits on whitespace and drops the empty pieces from the padding. `join` is called on the separator.

**Case: a slice and an f-string**

```python
price = 999000
print("Irvine"[0:3])
print(f"{price / 1000:.0f}k")
print("Irvine".startswith(("Ir", "Tu")))
print("7".zfill(3))
```

Result:

```text
Irv
999k
True
007
```

Why: the slice stops before index `3`. `.0f` formats the division with no decimal places. `startswith` accepts a tuple of options. `zfill` pads on the left.

**Case: `find` versus `index`, and `strip` versus a word**

```python
print("Irvine".find("x"))
try:
    "Irvine".index("x")
except ValueError:
    print("missing")
print("Irvine".casefold() == "IRVINE".casefold())
print("hoa".removeprefix("h"))
print("Irvine".strip("Irv"))
```

Result:

```text
-1
missing
True
oa
ine
```

Why: `find` returns `-1` when the text is absent. `index` raises `ValueError`. `strip("Irv")` removes any of those characters from the ends, so it eats `I`, `r`, and `v` and leaves `ine`. It does not remove the word `Irv`. `removeprefix` removes an exact prefix.

**Case: `re.search`**

```python
import re

text = "3 beds in Irvine"
found = re.search(r"(\d+) beds", text)
beds = int(found.group(1)) if found else None
print(beds)

missing = re.search(r"(\d+) beds", "no number")
print(missing)
print(int(missing.group(1)) if missing else None)
```

Result:

```text
3
None
None
```

Why: the raw pattern `\d+` captures the digits. `group(1)` is that capture. When the pattern is absent, `re.search` returns `None`. The `if found` check skips `.group`. Calling `.group` on `None` raises `AttributeError`. `re.fullmatch` would fail on this text because characters follow `beds`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Assuming `strip` edits the string | The original is unchanged | Bind the result: `city = city.strip()` |
| `strip("Los")` to remove a word | Any of those characters leave the ends | `removeprefix` / `removesuffix` for an exact end |
| `.group` when the search failed | `AttributeError` | Keep the `if found` check |
| `" ".join(items)` written as `items.join(" ")` | `AttributeError`. A list has no `join` | Call `join` on the separator string |

### Tip

`casefold` is the comparison tool. `lower` is the display tool. They differ on a few characters outside plain English.

### Practice

From `"  3 beds  "`, print the stripped string and the integer captured by `re.search(r"(\d+)", ...)`.

### Solution

```python
import re

text = "  3 beds  "
print(text.strip())
found = re.search(r"(\d+)", text)
print(int(found.group(1)) if found else None)
```

Result:

```text
3 beds
3
```

## 4. Control flow

### Concept

`if`, `elif`, and `else` pick one branch. `for` walks a collection. `while` repeats while a test is true. `range(stop)` yields `0` through `stop - 1`. `range(start, stop, step)` excludes `stop`. `break` leaves the loop. `continue` skips to the next pass. The `else` on a `for` or `while` runs when the loop did not `break`. `match` is structural matching, like a switch, in modern Python.

### Terms

| Term | Meaning |
| --- | --- |
| `elif` | Else if. Checked only when earlier tests failed |
| `else` on `if` | The branch when every test failed |
| `else` on a loop | Runs when the loop finished without `break` |
| `range` | A sequence of ints. The stop is excluded |
| `break` | Leave the innermost loop |
| `continue` | Skip the rest of this pass |
| `match` / `case` | Pick a branch by pattern. `case _` is the default |
| `\|` inside `case` | Either pattern. `"approved" \| "rejected"` |

### Methods

These are the statements, not object methods.

| Statement | Shape |
| --- | --- |
| `if` / `elif` / `else` | Tests in order. One body runs |
| `for name in items` | One pass per item |
| `for i in range(3)` | `i` is `0`, `1`, then `2` |
| `while n > 0` | Re-test before every pass |
| `break` / `continue` | Leave, or skip to the next pass |
| `for` / `else` | The `else` is about `break`, not about empty data alone |
| `match` | `case "approved" \| "rejected"` then `case _` |

### Examples

**Case: `if`, `elif`, and `else`**

```python
def size(beds):
    if beds >= 3:
        return "large"
    elif beds == 2:
        return "mid"
    else:
        return "small"

print(size(3))
print(size(2))
print(size(1))
```

Result:

```text
large
mid
small
```

Why: the first true test wins. `elif` is not checked after a true `if`.

**Case: `for` and `range`**

```python
for city in ["Irvine", "Tustin"]:
    print(city)
for i in range(3):
    print(i)
```

Result:

```text
Irvine
Tustin
0
1
2
```

Why: `range(3)` starts at `0` and stops before `3`. `range(1, 4, 2)` would be `1` then `3`.

**Case: `continue`, `break`, and loop `else`**

```python
for n in range(5):
    if n == 1:
        continue
    if n == 3:
        break
    print(n)
else:
    print("done")
print("after")
```

Result:

```text
0
2
after
```

Why: `continue` skips the print for `1`. `break` leaves at `3`, so `4` never prints and the loop `else` does not run. `after` is outside the loop.

**Case: `match` and `while`**

```python
def label(status):
    match status:
        case "approved" | "rejected":
            return "decided"
        case _:
            return "waiting"

print(label("approved"))
print(label("open"))

n = 3
while n > 0:
    n -= 1
print(n)
```

Result:

```text
decided
waiting
0
```

Why: the `|` inside `case` means either string. `case _` is the fallback. The loop subtracts until `n` is `0`, then the test fails.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `range(3)` expected to include `3` | The values are `0`, `1`, `2` | Pass the stop you want plus one, or use an inclusive end you compute |
| Reading loop `else` as "no items" | The `else` also runs when the loop simply finishes | Use it as "did not `break`" |
| `break` intended for the outer loop | It leaves only the inner loop | Put the outer loop in a function and `return`, or use a flag |
| A `while` test that never becomes false | The program does not finish | Change a name inside the loop, the way `n -= 1` does |

### Tip

Prefer `for` when you know the items. Use `while` when the next step depends on work you have not done yet.

### Practice

Print the ints from `range(1, 4)`. Then loop `0, 1, 2` and `break` when the value is `1`, with an `else` that would print `done`.

### Solution

```python
for i in range(1, 4):
    print(i)
for n in range(3):
    if n == 1:
        break
    print("saw", n)
else:
    print("done")
```

Result:

```text
1
2
3
saw 0
```

`done` does not print, because the loop hit `break`.

## 5. Functions

### Concept

A function binds a name to a block. A default argument is evaluated once, at definition. Do not use a list or a dict as a default. `*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dict. A nested function can read names from the outer function. To assign an outer name, declare `nonlocal`. To assign a module global, declare `global`. Prefer returning values over `global`.

A function with no `return` returns `None`. A lambda is one expression. A docstring is a string as the first statement. `help(fn)` shows it. Type hints such as `def f(n: int) -> str` are not enforced unless you run a checker.

### Terms

| Term | Meaning |
| --- | --- |
| Parameter | The name in the `def` line |
| Argument | The value at the call |
| Default | `unit=""`. Used when the caller omits that argument |
| Keyword call | `to_number("1.5", unit="m")` |
| `*args` | Extra positional arguments as a tuple |
| `**kwargs` | Extra keyword arguments as a dict |
| `lambda` | `lambda n: n * 2`. One expression, no statements |
| Docstring | The first statement, a string. `help(fn)` shows it |
| Type hint | `n: int` and `-> str`. Documentation for checkers, not a runtime gate |
| `nonlocal` | Assign a name in the enclosing function |
| `global` | Assign a name at module level |

### Methods

| Form | What it does |
| --- | --- |
| `def to_number(digits, unit="")` | `unit` defaults to an empty string |
| `return round(n)` | Sends a value back. Execution of the function stops |
| `*args` / `**kwargs` | Pack extras |
| `lambda n: n * 2` | An anonymous function of one expression |
| `"""..."""` under the `def` | The docstring |
| `nonlocal count` | The assignment updates the outer function's `count` |
| `global name` | The assignment updates the module name |

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

### Examples

**Case: defaults and a keyword argument**

```python
def to_number(digits, unit=""):
    n = float(str(digits).replace(",", ""))
    unit = unit.lower()
    if unit in ("k", "thousand"):
        n *= 1000
    if unit in ("m", "million"):
        n *= 1_000_000
    return round(n)

print(to_number("1.5", unit="m"))
print(to_number("1,500"))
print(to_number(2, "k"))
```

Result:

```text
1500000
1500
2000
```

Why: `"m"` multiplies by one million. The omitted unit stays `""`, so `"1,500"` is only parsed and rounded. `"k"` multiplies by one thousand.

**Case: a list default is shared**

```python
def add(item, bucket=[]):
    bucket.append(item)
    return bucket

print(add("a"))
print(add("b"))
```

Result:

```text
['a']
['a', 'b']
```

Why: the default list is created once, when `def` runs. Both calls receive that same list. Use `bucket=None` and create a new list inside the function.

**Case: `*args`, `**kwargs`, lambda, and a missing `return`**

```python
def collect(*args, **kwargs):
    return args, kwargs

print(collect(1, 2, city="Irvine"))
print((lambda n: n * 2)(4))

def blank():
    """Nothing is returned."""
    x = 1

print(blank())
print(blank.__doc__)
```

Result:

```text
((1, 2), {'city': 'Irvine'})
8
None
Nothing is returned.
```

Why: positional extras become a tuple and keyword extras become a dict. A lambda is one expression. `blank` has no `return`, so the value is `None`. The docstring is the first statement.

**Case: `nonlocal`, and hints that do not run**

```python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1
        return count

    return inner()

print(outer())

def f(n: int) -> str:
    return n

print(f("x"))
```

Result:

```text
1
x
```

Why: `nonlocal` lets `inner` assign the outer `count`. The hints on `f` do not stop a string argument. A checker would complain. The runtime does not.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `def add(item, bucket=[])` | Later calls see earlier items | Default `None`, then `bucket = []` inside |
| Forgetting `return` | The caller gets `None` | Return the value you computed |
| Assigning an outer name with no `nonlocal` | `UnboundLocalError`, or a new local that hides the outer name | Declare `nonlocal`, or return the new value instead |
| Trusting `n: int` to reject strings | The call succeeds | Run a type checker, and validate at the boundary if the value came from outside |

### Tip

Return a value. Reach for `global` only when a module-level setting must change, and even then prefer passing the value in.

### Practice

Write `double(n, factor=2)` and show a keyword call. Then show one shared-default bug with two calls, using a list default on purpose.

### Solution

```python
def double(n, factor=2):
    return n * factor

print(double(4, factor=3))

def add(item, bucket=[]):
    bucket.append(item)
    return bucket

print(add(1))
print(add(2))
```

Result:

```text
12
[1]
[1, 2]
```

## 6. Lists, tuples, sets, and dicts

### Concept

A list is ordered and mutable. A tuple is ordered and immutable. A set stores unique values and is unordered. A dict maps a hashable key to a value and keeps insertion order. There is no `LinkedList` class in the usual course. `collections.deque` is the double-ended queue. A dict is the hash map. `Counter` counts items. `defaultdict(list)` builds a list for a missing key.

Items in a set, and keys in a dict, must be hashable. A list cannot be a member or a key. A tuple of hashable items can.

### Terms

| Term | Meaning |
| --- | --- |
| Index | `0` is first. A negative index counts from the end |
| Slice | `items[1:3]` excludes the end. `items[::-1]` reverses |
| Shallow copy | `prices[:]` is a new list of the same items |
| Hashable | Allowed in a set and as a dict key. Numbers, strings, and tuples of hashable items |
| `KeyError` | A missing dict key, or `set.remove` of a missing item |
| Insertion order | Dicts remember the order keys were added |

| Need | Type |
| --- | --- |
| Ordered, duplicates, index | `list` |
| Fixed record | `tuple` |
| Unique items, fast membership | `set` |
| Key to value | `dict` |
| A queue or a stack of many end-inserts | `collections.deque` |
| Count items | `collections.Counter` |
| A dict that builds a list for a missing key | `collections.defaultdict(list)` |

### Methods

List:

| Operation | Code |
| --- | --- |
| Length | `len(prices)` |
| Slice | `prices[1:3]`. End excluded. `prices[::-1]` reverses |
| Membership | `999000 in prices` |
| Add | `append`, `insert`, `extend` |
| Remove | `pop`, `remove` (raises if the value is missing), `del prices[i]` |
| Count | `prices.count(999000)` |
| Sort in place | `prices.sort()`. Accepts `key=` and `reverse=True` |
| Sort a copy | `sorted(prices)` returns a new list |
| Comprehension | `[p for p in prices if p > 1000000]` |

Tuple:

| Fact | Detail |
| --- | --- |
| Create | `(1, 2)` or `1, 2`. The comma builds the tuple |
| One item | `(1,)` is a tuple. `(1)` is an int. Parentheses only group |
| Unpack | `city, price = pair` |
| Change | No item assignment. No append |

Set:

| Operation | Result |
| --- | --- |
| Create | `{1, 2}`. An empty set is `set()`, because `{}` is a dict |
| `add` | Insert one item |
| `discard` | Remove, and do nothing if it is missing |
| `remove` | Remove, or raise `KeyError` |
| `\|` or `union` | Either set |
| `&` or `intersection` | Both |
| `-` or `difference` | Left only |
| `^` | In one but not both |
| `set(items)` | Unique items from a list |
| Comprehension | `{c.lower() for c in cities}` |

Dict:

| Operation | Code |
| --- | --- |
| Read | `card["price"]` raises `KeyError` when the key is missing. `card.get("hoa")` returns `None`. `card.get("hoa", 0)` returns the default |
| Write | `card["beds"] = 3`, `update`, `setdefault` |
| Keys / values / pairs | `card.keys()`, `.values()`, `.items()` |
| Pop | `card.pop("beds", None)` |
| Merge | `card \| {"price": 1}` in modern Python. A new dict |
| Delete | `del card["beds"]` |
| Comprehension | `{k: v for k, v in card.items() if v}` |
| Membership | `key in card` tests keys, not values |

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

| Use a tuple | Use a list |
| --- | --- |
| A fixed record, such as a pair | A collection that grows |
| A dict key | Items you will sort or filter in place |

### Examples

**Case: a list mutates, and a slice is a copy**

```python
prices = [999000, 1100000]
prices.append(1350000)
prices.insert(0, 800000)
last = prices.pop()
print(last)
print(prices)
prices.sort()
print(prices)
copy = prices[:]
copy.append(1)
print(prices)
print([p for p in prices if p > 1000000])
print(prices[-1])
```

Result:

```text
1350000
[800000, 999000, 1100000]
[800000, 999000, 1100000]
[800000, 999000, 1100000]
[1100000]
1100000
```

Why: `append` and `insert` change the same list. `pop` returns the last item. `sort` mutates and accepts `key=` and `reverse=True`. `prices[:]` is a new list, so appending to `copy` leaves `prices` alone. The comprehension filters. Index `-1` is the last item.

**Case: the comma builds the tuple**

```python
pair = ("Irvine", 999000)
city, price = pair
print(city, price)
print(type((1)).__name__)
print(type((1,)).__name__)
try:
    pair[0] = "Tustin"
except TypeError as error:
    print(type(error).__name__)
```

Result:

```text
Irvine 999000
int
tuple
TypeError
```

Why: parentheses group. The comma makes the tuple, so `(1)` is an int and `(1,)` is a tuple. Item assignment fails because a tuple is immutable. A tuple of hashable items can be a dict key. A list cannot.

**Case: a set drops duplicates**

```python
cities = {"Irvine", "Tustin", "Irvine"}
cities.add("San Diego")
cities.discard("missing")
print("Irvine" in cities)
print(len(cities))
print({"Irvine", "Tustin"} & {"Tustin", "Anaheim"})
try:
    cities.remove("Nope")
except KeyError:
    print("missing")
```

Result:

```text
True
3
{'Tustin'}
missing
```

Why: the second `"Irvine"` is not stored. `discard` is silent when the item is absent. `remove` raises `KeyError`. `&` is the intersection. Do not rely on the printed order of a larger set.

**Case: dict lookup, `Counter`, `deque`, and `defaultdict`**

```python
from collections import Counter, defaultdict, deque

card = {"city": "Irvine", "price": 999000}
card["beds"] = 3
print(card.get("price"))
print(card.get("hoa"))
print(card.get("hoa", 0))
try:
    print(card["hoa"])
except KeyError:
    print("missing")
print({"price": 1} | {"price": 2})

counts = Counter(["Irvine", "Irvine", "Tustin"])
print(counts["Irvine"])

line = deque()
line.append("a")
line.appendleft("b")
print(line.popleft())

groups = defaultdict(list)
groups["Irvine"].append(999000)
print(groups["Irvine"])
```

Result:

```text
999000
None
0
missing
{'price': 2}
2
b
[999000]
```

Why: `get` returns `None` or the default you pass. `card["hoa"]` raises `KeyError`. `|` builds a new dict. `Counter` tallies items. `appendleft` and `popleft` are the deque ends. A `defaultdict(list)` creates the list on the first append for that key.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `{}` for an empty set | You built a dict | `set()` |
| `(1)` for a one-item tuple | The value is an int | `(1,)` |
| `card["hoa"]` when the key may be absent | `KeyError` | `card.get("hoa", 0)` |
| `cities.remove("Nope")` | `KeyError` | `discard` when absence is fine |
| Sorting a set in place | A set has no `sort` | `sorted(cities)` returns a list |

### Tip

Pick the type from the job in the table. If you need an index and duplicates, it is a list. If you need "have I seen this key," it is a set or a dict.

### Practice

Build `{"city": "Irvine"}`. Print `.get("beds", 0)` and show that `["beds"]` raises. Then make a set from `["Irvine", "Irvine"]` and print its length.

### Solution

```python
card = {"city": "Irvine"}
print(card.get("beds", 0))
try:
    print(card["beds"])
except KeyError:
    print("raised")
print(len({"Irvine", "Irvine"}))
```

Result:

```text
0
raised
1
```

## 7. Classes

### Concept

A class bundles data and functions. `__init__` receives the new object as `self`. `self` is explicit and is the first parameter of an instance method. `class Condo(Listing)` inherits. `super()` calls the parent. A single leading underscore, `_name`, is a convention that the name is internal. It is not enforced. `__str__` is what `str(obj)` and `print` use. `__repr__` is the developer-facing text. `@dataclass` generates `__init__` and equality for a field list.

### Terms

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
| Attribute | A name on the object, such as `home.city` |

### Methods

| Call | Role |
| --- | --- |
| `Listing("Irvine", 999000)` | Calls `__init__` and returns the object |
| `home.label()` | Passes `home` as `self` |
| `super().__init__(city, price)` | Runs the parent constructor |
| `str(obj)` / `print(obj)` | Uses `__str__` when it exists |
| `repr(obj)` | Uses `__repr__` |
| `Card("Irvine", 999000)` on a dataclass | The generated constructor |

### Examples

**Case: a class and a subclass**

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
print(home.label())
condo = Condo("Irvine", 800000, 350)
print(condo.label())
print(condo.hoa)
```

Result:

```text
Irvine: 999000
Irvine: 800000
350
```

Why: `Listing(...)` calls `__init__`. `label` receives the instance as `self`. `Condo` calls the parent constructor, then stores `hoa`.

**Case: `__str__` and `__repr__`**

```python
class Card:
    def __init__(self, city):
        self.city = city

    def __str__(self):
        return self.city

    def __repr__(self):
        return f"Card({self.city!r})"

card = Card("Irvine")
print(card)
print(repr(card))
```

Result:

```text
Irvine
Card('Irvine')
```

Why: `print` uses `__str__`. `repr` uses `__repr__`. The `!r` in the f-string asks for the representation of the city, including quotes.

**Case: a dataclass compares fields**

```python
from dataclasses import dataclass

@dataclass
class Card:
    city: str
    price: int

a = Card("Irvine", 999000)
b = Card("Irvine", 999000)
print(a == b)
print(a.city)
```

Result:

```text
True
Irvine
```

Why: the decorator writes `__init__` and `__eq__`. Two cards with the same fields compare equal. You did not type either method.

**Case: a leading underscore is a convention**

```python
class Box:
    def __init__(self):
        self._code = "a"

box = Box()
print(box._code)
```

Result: `a`

Why: Python still allows the read. The underscore tells other programmers that the name is internal to the class.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Forgetting `self` on a method | `TypeError` when you call it, or the attributes are missing | Put `self` first, and use `self.city` for instance data |
| Skipping `super().__init__` | The parent attributes were never set | Call `super().__init__(...)` before using parent fields |
| Treating `_code` as private | Any caller can read it | Privacy is a convention. Do not rely on the underscore for security |
| Expecting two objects with the same fields to be equal | A plain class compares identity | Use a dataclass, or write `__eq__` |

### Tip

A dataclass is enough for a record of fields. Write a class with methods when the object has behavior, the way `label` does.

### Practice

Define `Listing` with `city` and `price`, and `Condo` that adds `hoa` through `super()`. Print the condo's city and hoa for `("Irvine", 800000, 350)`.

### Solution

```python
class Listing:
    def __init__(self, city, price):
        self.city = city
        self.price = price

class Condo(Listing):
    def __init__(self, city, price, hoa):
        super().__init__(city, price)
        self.hoa = hoa

condo = Condo("Irvine", 800000, 350)
print(condo.city)
print(condo.hoa)
```

Result:

```text
Irvine
350
```

## 8. Modules, files, and errors

### Concept

`import math` loads a module. You then use `math.floor`. `from pathlib import Path` brings in one name. `pip install` puts a third-party package into the environment. `python3 -m venv venv` makes a private set of packages. Code under `if __name__ == "__main__":` runs only when this file is the program, not when it is imported.

`try` watches a block. `except ValueError` handles that type. `finally` runs whether the block succeeded or raised. `raise` without a value, inside `except`, re-raises the current exception. `raise ... from error` keeps the cause. A `with` block closes the file even if a line raises.

`json.dumps(obj)` makes text. `json.loads(text)` makes objects. Dict keys become strings. Tuples become lists.

### Terms

| Term | Meaning |
| --- | --- |
| Module | A file of Python you can import |
| Package | A third-party library installed with `pip` |
| `venv` | A private set of packages for one project |
| `__name__` | `"__main__"` when this file is the program. Otherwise the module name |
| Exception | An object raised to report a failure |
| `from error` | Links the new exception to the one that caused it |
| `with` | Calls the cleanup method even when a line raises |
| JSON | Text that `dumps` and `loads` convert to and from Python objects |

| Exception | Usual cause |
| --- | --- |
| `ValueError` | The right type, an unacceptable value |
| `TypeError` | The wrong type |
| `KeyError` | A missing dict key |
| `IndexError` | A bad list index |
| `FileNotFoundError` | The path is not there |
| `ZeroDivisionError` | Division by zero |

### Methods

| Tool | Role |
| --- | --- |
| `import math` | A module. Use `math.floor` |
| `from pathlib import Path` | One name |
| `Path("notes.txt").read_text()` | The whole file as a string |
| `Path("notes.txt").write_text(text)` | Replace the file |
| `open(path, encoding="utf-8")` | A file handle. Prefer it inside `with` |
| `json.dumps` / `json.loads` | Text to objects and back. Keys become strings. Tuples become lists |
| `pip install` | Install a third-party package into the environment |
| `python3 -m venv venv` | Create a virtual environment |
| `raise ValueError("...") from error` | A new message that keeps the cause |
| `raise` inside `except` | Re-raise the current exception |

### Examples

**Case: import, floor, and the main guard**

```python
import math

print(math.floor(2.9))
if __name__ == "__main__":
    print("program")
```

Result: when you run the file with `python3 file.py`,

```text
2
program
```

Why: `import math` loads the standard library module. `floor` rounds toward negative infinity, so `2.9` becomes `2`. The guard's body runs because `__name__` is `"__main__"`. Importing this file from another module still runs the `floor` line and skips the print, because `__name__` is then the module name.

**Case: write a file, then read it with `with`**

```python
from pathlib import Path

Path("notes.txt").write_text("Irvine\nTustin\n")
with open("notes.txt", encoding="utf-8") as handle:
    for line in handle:
        print(line.rstrip("\n"))
```

Result:

```text
Irvine
Tustin
```

Why: `write_text` replaces the file. `with` closes the handle after the loop, including when a line inside the block raises. `rstrip("\n")` drops the line break that the file stored. `encoding="utf-8"` tells `open` how to decode the bytes.

**Case: JSON turns tuples into lists and keys into strings**

```python
import json

print(json.dumps({"city": "Irvine", "pair": (1, 2)}))
print(json.loads(json.dumps({1: "a"})))
```

Result:

```text
{"city": "Irvine", "pair": [1, 2]}
{'1': 'a'}
```

Why: `dumps` makes text. A tuple becomes a JSON array, so loading it back yields a list. An int key becomes the string `"1"`.

**Case: `raise ... from`, and three other exceptions**

```python
def parse_price(text):
    try:
        return int(text.replace(",", ""))
    except ValueError as error:
        raise ValueError("price must be a number") from error

try:
    parse_price("nope")
except ValueError as error:
    print(error)
    print(type(error.__cause__).__name__)

try:
    print(1 / 0)
except ZeroDivisionError:
    print("zero")

try:
    print([1][3])
except IndexError:
    print("index")

try:
    print({}["city"])
except KeyError:
    print("key")
```

Result:

```text
price must be a number
ValueError
zero
index
key
```

Why: `int("nope")` raises `ValueError`. The new error keeps that as `__cause__` because of `from error`. A bare `raise` inside `except` would re-raise the current exception instead of wrapping it. The other three lines show `ZeroDivisionError`, `IndexError`, and `KeyError`. `FileNotFoundError` is the same pattern when `Path("missing-notes.txt").read_text()` cannot find the path. `finally` still runs if you add it, including when the `except` raises.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `open` without `with` or `close` | The handle stays open if a later line raises | Use `with` |
| Catching `Exception` and hiding the cause | The real bug disappears | Catch the type you expect, or use `raise ... from error` |
| `json.loads` of a file path | You parsed the path string, not the file | `read_text()`, then `json.loads` |
| Assuming a tuple survives JSON | It comes back as a list | Convert it again if you need a tuple |

### Tip

Let a bug raise while you are writing the program. Catch an exception at the boundary where you can say something useful, such as "price must be a number."

### Practice

Parse `"999,000"` with the `parse_price` pattern and print the int. Then dump `("Irvine", 1)` with `json.dumps` and show the text.

### Solution

```python
import json

def parse_price(text):
    try:
        return int(text.replace(",", ""))
    except ValueError as error:
        raise ValueError("price must be a number") from error

print(parse_price("999,000"))
print(json.dumps(("Irvine", 1)))
```

Result:

```text
999000
["Irvine", 1]
```

## Drill

1. Write `unique(items)` using a set, and return a list.
2. Count cities with `Counter`.
3. Read a missing dict key with `.get` and with `[]`, and say which one raises.
4. Say what `re.search` returns when the pattern is not in the text.

### Solution 1

```python
def unique(items):
    return list(set(items))

print(sorted(unique(["Irvine", "Tustin", "Irvine"])))
```

Result: `['Irvine', 'Tustin']`

`sorted` is only for a stable print. `unique` itself returns a list of the set's items, and a set does not promise order.

### Solution 2

```python
from collections import Counter

counts = Counter(["Irvine", "Irvine", "Tustin"])
print(counts["Irvine"])
```

Result: `2`

### Solution 3

```python
card = {"city": "Irvine"}
print(card.get("hoa"))
try:
    print(card["hoa"])
except KeyError:
    print("raised")
```

Result:

```text
None
raised
```

`.get` returns `None`. `[]` raises `KeyError`.

### Solution 4

```python
import re

print(re.search(r"(\d+) beds", "no number"))
```

Result: `None`

Calling `.group` on that `None` raises `AttributeError`. Check the match first.
