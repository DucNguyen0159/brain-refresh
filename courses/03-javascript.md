# JavaScript

Course 3 of 15. Previous: [02-css.md](02-css.md). Next: [04-typescript.md](04-typescript.md).

JavaScript is the language of the browser and of Node. This course is the language and the browser APIs. TypeScript, React, and Node each have their own course. Examples here are plain JavaScript. Run them with `node file.js` or a `<script>` tag. `console.log` prints to the console.

Every section has the same parts: the concept, the terms, the methods, several cases with what prints, common mistakes, a tip, and a practice with a solution.

## Contents

1. [Running code](#1-running-code)
2. [Values and variables](#2-values-and-variables)
3. [Operators](#3-operators)
4. [Control flow](#4-control-flow)
5. [Functions](#5-functions)
6. [Scope and closures](#6-scope-and-closures)
7. [Objects](#7-objects)
8. [Arrays](#8-arrays)
9. [Strings, numbers, and dates](#9-strings-numbers-and-dates)
10. [Set and Map](#10-set-and-map)
11. [JSON, errors, and modules](#11-json-errors-and-modules)
12. [Classes](#12-classes)
13. [Promises and async](#13-promises-and-async)
14. [The DOM and events](#14-the-dom-and-events)
15. [Fetch and storage](#15-fetch-and-storage)

## 1. Running code

### Concept

In a browser, a `script` element runs the file. In a terminal, `node file.js` runs it. A statement usually ends with a semicolon. Automatic semicolon insertion usually adds one, and it can add the wrong one after `return` if the value is on the next line. `"use strict";` at the top of a classic script rejects silent mistakes such as assigning to an undeclared name. Modules and classes are already strict.

### Terms

| Term | Meaning |
| --- | --- |
| Statement | One step. `console.log("ready");` |
| Expression | A piece that produces a value. `1 + 2` |
| Script | A file of statements |
| `defer` | Run the script after the HTML is parsed, in order |
| Strict mode | Assigning to an undeclared name throws |

### Methods

| Tool | Where the text goes |
| --- | --- |
| `console.log` | The console |
| `console.error` | The console, marked as an error |
| `alert` | A blocking dialog. Fine once, while you learn. Not for a real page |
| `document.body.textContent = "Hi"` | The page |

### Examples

**Case: the console, not the page**

```html
<script src="app.js" defer></script>
```

```javascript
console.log("ready");
```

Result: the console shows `ready`. The page stays empty.

Why: `console.log` does not write into the document. `document.body.textContent = "ready"` would.

**Case: `return` and a line break**

```javascript
function city() {
  return
    "Irvine";
}
console.log(city());
```

Result: `undefined`.

Why: automatic semicolon insertion treats `return` as a finished statement. The string on the next line never gets returned. Put `"Irvine"` on the same line as `return`.

**Case: strict mode catches a typo**

```javascript
"use strict";
city = "Irvine";
```

Result: `ReferenceError`. Without strict mode, a classic script creates a global named `city`.

Why: the assignment has no `let`, `const`, or `var`. Strict mode refuses to invent the global.

**Case: `defer` sees the body**

```html
<script src="app.js" defer></script>
<p id="label">Irvine</p>
```

```javascript
console.log(document.getElementById("label").textContent);
```

Result: `Irvine`.

Why: `defer` waits until the parser has built the element. The same script in the head without `defer` can run before the paragraph exists and throw.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Expecting `console.log` to change the page | The page looks blank | Write to the document, or open the console |
| A value on the line after `return` | The function returns `undefined` | Keep the value on the `return` line |
| A script in the head with no `defer` that reads the body | `null` and then a throw | `defer` |
| Forgetting strict mode in a classic script | Typos become globals | `"use strict";` or use a module |

### Tip

If a function returns `undefined` and you can see the value in the source, look for a line break after `return`.

### Practice

Print `Irvine` from `app.js`, loaded with `defer`, and return the same word from a function without a line break after `return`.

### Solution

```html
<script src="app.js" defer></script>
```

```javascript
function city() {
  return "Irvine";
}
console.log(city());
```

Result: the console shows `Irvine`.

## 2. Values and variables

### Concept

Every value has a type. `typeof null` is `"object"`. That is a long-standing mistake in the language. Test null with `value === null`. `let` and `const` live in the block where they are declared. Using either name before its line throws. That gap is the temporal dead zone. `const` protects the binding, not the contents of an object. `var` is function-scoped and hoisted as `undefined`. Avoid it in new code.

### Terms

| Type | Examples | `typeof` |
| --- | --- | --- |
| number | `3`, `2.5`, `NaN`, `Infinity` | `"number"` |
| string | `"Irvine"`, `'Irvine'`, `` `Irvine` `` | `"string"` |
| boolean | `true`, `false` | `"boolean"` |
| undefined | A missing value | `"undefined"` |
| object | `{ city: "Irvine" }`, `[1, 2]`, `null` | `"object"` |
| function | `function f() {}` | `"function"` |
| bigint | `10n` | `"bigint"` |
| symbol | `Symbol("id")` | `"symbol"` |

| Keyword | Reassign the name? | Scope |
| --- | --- | --- |
| `let` | Yes | The block |
| `const` | No | The block |
| `var` | Yes | The function |

### Methods

Variables do not have methods. These are the checks you use on values.

| Check | Result |
| --- | --- |
| `typeof value` | One of the strings in the type table. `typeof null` is `"object"` |
| `value === null` | The null test |
| `Array.isArray(value)` | The array test. `typeof` an array is `"object"` |
| `` `${beds} beds` `` | A template string. It interpolates and keeps line breaks |

### Examples

**Case: `const` blocks a new binding and allows a new property**

```javascript
const card = { price: 1 };
card.price = 2;
console.log(card.price);
card = { price: 3 };
```

Result: the log prints `2`. The next line throws `TypeError`.

Why: `const` means the name `card` keeps pointing at the same object. The object's fields can still change.

**Case: `typeof null`**

```javascript
console.log(typeof null);
console.log(null === null);
console.log(typeof []);
console.log(Array.isArray([]));
```

Result:

```text
object
true
object
true
```

Why: `typeof` is the wrong test for null and for arrays. Use `=== null` and `Array.isArray`.

**Case: the temporal dead zone**

```javascript
console.log(city);
let city = "Irvine";
```

Result: `ReferenceError`. The name exists for the block, and it cannot be read until the `let` line runs.

Why: this is not `undefined`. `var city` would have logged `undefined` and hidden the bug.

**Case: a template keeps the number's text**

```javascript
const beds = 3;
console.log(`${beds} beds`);
console.log(beds + " beds");
```

Result: both lines print `3 beds`.

Why: a template converts the interpolated value to a string. `+` with a string does the same concatenation. `3 + 2 + " beds"` is `"5 beds"` because the numbers add first. `"beds " + 3 + 2` is `"beds 32"` because once a string appears, `+` concatenates.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `typeof value === "object"` to detect null | Null passes | `value === null` |
| `const` treated as a frozen object | Properties still change | `Object.freeze` when the contents must stay fixed |
| Reading a `let` before its line | `ReferenceError` | Declare, then use |
| `var` in new code | The name leaks through the whole function | `let` or `const` |

### Tip

`const` by default. Use `let` when you can point at the line that reassigns the name. Do not use `var`.

### Practice

Create a `const` object with `city: "Irvine"`, change the city to `Tustin`, and print it with a template.

### Solution

```javascript
const card = { city: "Irvine" };
card.city = "Tustin";
console.log(`${card.city}`);
```

Result: `Tustin`. `card = {}` would throw.

## 3. Operators

### Concept

`===` compares without converting types. `==` converts first. Avoid `==`. `&&` returns the first falsy value, or the last value. `||` returns the first truthy value, or the last value. `??` returns the right side only when the left is `null` or `undefined`. `0` and `""` are kept by `??` and replaced by `||`. Falsy values are `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`. `"0"`, `"false"`, and `[]` are truthy.

### Terms

| Term | Meaning |
| --- | --- |
| Strict equality | `===` and `!==`. No conversion |
| Loose equality | `==` and `!=`. Conversion. Avoid |
| Falsy | Acts as false in `if`, `&&`, and `\|\|` |
| Nullish | Only `null` and `undefined` |
| Optional chaining | `user?.city` stops when `user` is nullish |

### Methods

| Operator | Meaning |
| --- | --- |
| `+ - * / % **` | Arithmetic. `%` is remainder. `**` is power |
| `+` with a string | Concatenate. `1 + "2"` is `"12"` |
| `===` / `!==` | Equal, no conversion |
| `&&` | First falsy, or the last value |
| `\|\|` | First truthy, or the last value |
| `??` | Right side only for `null` or `undefined` |
| `?.` | Stop if the base is nullish |
| `? :` | `cond ? a : b` |

`Number("3")` is `3`. `Number("")` is `0`. `parseInt("10px", 10)` is `10`. Always pass the radix `10`.

### Examples

**Case: `==` converts, `===` does not**

```javascript
console.log(1 === "1");
console.log(1 == "1");
console.log(0 === false);
console.log(0 == false);
```

Result:

```text
false
true
false
true
```

Why: `==` turns the string `"1"` into a number and turns `false` into `0`. A missing price and the number 0 then look the same. `===` keeps them apart.

**Case: `||` replaces zero, `??` keeps it**

```javascript
const price = 0;
console.log(price ?? "missing");
console.log(price || "missing");
console.log("" ?? "missing");
console.log("" || "missing");
```

Result:

```text
0
missing

missing
```

Why: zero and `""` are falsy, so `||` skips them. They are not nullish, so `??` keeps them. Use `??` for a default only when the value was never provided.

**Case: optional chaining**

```javascript
const user = null;
console.log(user?.city);
console.log(user.city);
```

Result: the first log prints `undefined`. The second throws `TypeError`.

Why: `?.` checks the base before the property read. It does not check later pieces unless you write `?.` again.

**Case: string concatenation versus addition**

```javascript
console.log(1 + 2 + " beds");
console.log("beds " + 1 + 2);
```

Result:

```text
3 beds
beds 12
```

Why: `+` runs left to right. Numbers add until a string appears. After a string, `+` concatenates.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `==` for a price or an id | `0 == ""` and `0 == false` are true | `===` |
| `value \|\| "missing"` when `0` is valid | The zero is replaced | `??` |
| `user.city` when `user` may be null | `TypeError` | `user?.city` |
| `parseInt("10px")` with no radix | Usually 10, and the radix is still a foot-gun in old engines | `parseInt("10px", 10)` |

### Tip

If the legal value can be `0` or `""`, do not use `||` to fill in a default.

### Practice

Print whether `0` should be replaced by `"missing"` using `??`, and print the `||` result next to it.

### Solution

```javascript
const price = 0;
console.log(price ?? "missing");
console.log(price || "missing");
```

Result:

```text
0
missing
```

## 4. Control flow

### Concept

`if` runs one branch. `switch` compares with `===` and falls into the next case until `break`. `for` counts. `while` tests before the body. `do...while` runs the body once before the test. `for...of` walks values. `for...in` walks keys, including inherited enumerable keys. Prefer `Object.keys` or `for...of`. `break` leaves the loop. `continue` skips to the next pass.

### Terms

| Term | Meaning |
| --- | --- |
| Branch | One arm of `if` or `switch` |
| Fall-through | A `case` with no `break` keeps running |
| Iterable | Something `for...of` can walk. Arrays, strings, `Set`, `Map` |

### Methods

| Statement | Effect |
| --- | --- |
| `if / else` | One branch |
| `switch` | Match with `===`. `break`, or execution falls through |
| `for` | An index |
| `while` | Test, then body. Maybe zero times |
| `do...while` | Body, then test. At least once |
| `for...of` | Each value |
| `for...in` | Each key. Avoid for arrays |
| `break` / `continue` | Leave, or skip to the next pass |

### Examples

**Case: `else if` is the first true test**

```javascript
const beds = 2;
if (beds >= 3) {
  console.log("large");
} else if (beds === 2) {
  console.log("mid");
} else {
  console.log("small");
}
```

Result: `mid`.

Why: `beds >= 3` is false, so the next test runs. A later `else` does not run after a match.

**Case: `switch` falls through**

```javascript
const city = "Tustin";
switch (city) {
  case "Irvine":
  case "Tustin":
    console.log("orange county");
    break;
  default:
    console.log("elsewhere");
}
```

Result: `orange county`.

Why: `Tustin` has no statements of its own. Execution continues into the next case until `break`. Without that `break`, `elsewhere` would also print.

**Case: `for...of` versus `for...in` on an array**

```javascript
const beds = [2, 3];
for (const bed of beds) {
  console.log(bed);
}
for (const key in beds) {
  console.log(key);
}
```

Result: `of` prints `2` then `3`. `in` prints `"0"` then `"1"`, the indexes as strings.

Why: `for...in` is for object keys. On an array those keys are the indexes, not the values.

**Case: `do...while` runs once**

```javascript
let n = 0;
do {
  n += 1;
} while (n < 0);
console.log(n);
```

Result: `1`.

Why: the test `n < 0` is false after the first pass, and the body has already run. A `while` with the same test would have printed `0` and skipped the body.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `switch` without `break` | Later cases run too | `break`, unless you meant to share a case |
| `for...in` on an array | You get indexes, as strings | `for...of` |
| `=` inside `if (beds = 3)` | The assignment happens, and the `if` sees `3`, which is truthy | `===` |
| A `while` whose counter never changes | The page or the script never finishes | Change the counter inside the loop |

### Tip

In a `switch`, write the `break` before you write the next `case`. Shared cases are the exception, and they should be stacked with no code between them.

### Practice

Print `large` when `beds` is at least 3, `mid` when it is 2, and `small` otherwise. Try `beds = 2`.

### Solution

The `if / else if` example in this section, with `beds` set to `2`.

Result: `mid`.

## 5. Functions

### Concept

A function is a value you can call. `function f()` is hoisted: the name exists for the whole enclosing scope. `const f = function () {}` is not hoisted. An arrow function has no `this` of its own and no `arguments` object. A one-expression arrow can omit `return` and braces. A default parameter is used when the argument is `undefined`. Rest collects the leftover arguments into an array. Without `return`, the result is `undefined`.

Arguments are passed by value. The value of an object argument is a reference, so the function can change that object's fields. Reassigning the parameter does not reassign the caller's variable.

### Terms

| Term | Meaning |
| --- | --- |
| Parameter | The name inside the function |
| Argument | The value the caller passes |
| Hoisted function | A `function` declaration. Callable above its line in that scope |
| Arrow | `(n) => n * 2`. Lexical `this` |
| Callback | A function you pass so someone else can call it later |
| Rest | `...nums` gathers arguments into an array |

### Methods

| Form | Notes |
| --- | --- |
| `function f(a, b) {}` | Hoisted. Has its own `this` when called as a method |
| `const f = function () {}` | Not hoisted |
| `const f = (a) => a + 1` | No own `this`. One expression can omit `return` |
| Default | `function f(unit = "")`. Used for `undefined` |
| Rest | `function sum(...nums)` |

### Examples

**Case: no `return` means `undefined`**

```javascript
function label(city) {
  city.toUpperCase();
}
console.log(label("Irvine"));
```

Result: `undefined`.

Why: `toUpperCase` returns a new string and the function drops it. `return city.toUpperCase()` prints `IRVINE`.

**Case: changing the object versus reassigning the parameter**

```javascript
function rename(card, city) {
  card.city = city;
}
function replace(card) {
  card = { city: "Tustin" };
}
const card = { city: "Irvine" };
rename(card, "Tustin");
console.log(card.city);
replace(card);
console.log(card.city);
```

Result:

```text
Tustin
Tustin
```

Why: `rename` edits the object both names point at. `replace` points its own parameter at a new object. The caller's `card` is unchanged, so it still says `Tustin` from the first call.

**Case: a default is only for `undefined`**

```javascript
function bedsLabel(n = 1) {
  return `${n} beds`;
}
console.log(bedsLabel());
console.log(bedsLabel(0));
console.log(bedsLabel(undefined));
```

Result:

```text
1 beds
0 beds
1 beds
```

Why: a missing argument is `undefined`, so the default runs. `0` is a real argument. The default does not replace it.

**Case: an arrow and a block arrow**

```javascript
const double = (n) => n * 2;
const doubleBlock = (n) => {
  n * 2;
};
console.log(double(4));
console.log(doubleBlock(4));
```

Result:

```text
8
undefined
```

Why: without braces, the expression is the return value. With braces, you need `return`. The block version computes `8` and throws it away.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Forgetting `return` | The caller gets `undefined` | Return the value you computed |
| A block arrow with no `return` | Same | Add `return`, or drop the braces |
| Reassigning a parameter to "update" the caller | The caller keeps the old object | Mutate a field, or return the new object |
| A default expected to replace `0` or `""` | Those values stay | Test them yourself if they are not allowed |

### Tip

If the caller prints `undefined`, the function ran and did not return. The bug is inside the function, not at the call.

### Practice

Write `toNumber("1,200")` that strips commas and returns the number `1200`.

### Solution

```javascript
function toNumber(digits) {
  return Number(String(digits).replaceAll(",", ""));
}
console.log(toNumber("1,200"));
```

Result: `1200`.

## 6. Scope and closures

### Concept

A `let` or `const` name is visible from its declaration to the end of that block, and inside nested blocks. It is not visible outside. A closure is a function that uses a name from an outer function. That outer name stays alive after the outer function has returned. Two calls to the outer function get two different variables.

### Terms

| Term | Meaning |
| --- | --- |
| Block scope | Between `{` and `}` for `let` and `const` |
| Closure | An inner function plus the outer variables it uses |
| Module scope | Top-level names in a file. Not automatically global |
| Global | `window` in a classic browser script. Avoid adding to it |

### Methods

Scope is not a list of methods. These are the tools that create a scope boundary.

| Tool | Boundary |
| --- | --- |
| `{ }` | A block. `if`, `for`, and a bare block |
| `function` | Parameters and the body |
| File used as a module | Top-level names stay in the file unless exported |

### Examples

**Case: a block name does not escape**

```javascript
if (true) {
  const city = "Irvine";
  console.log(city);
}
console.log(city);
```

Result: the first log is `Irvine`. The second throws `ReferenceError`.

Why: `const` belongs to the `if` block. Outside that block the name does not exist.

**Case: a counter closes over `n`**

```javascript
function makeCounter() {
  let n = 0;
  return function () {
    n += 1;
    return n;
  };
}
const next = makeCounter();
console.log(next());
console.log(next());
```

Result:

```text
1
2
```

Why: the returned function keeps using the `n` from that call to `makeCounter`. `n` is not a global. It survives because the inner function still points at it.

**Case: two counters do not share `n`**

```javascript
const a = makeCounter();
const b = makeCounter();
console.log(a());
console.log(b());
console.log(a());
```

Result:

```text
1
1
2
```

Why: each call to `makeCounter` creates a new `n`. `a` and `b` close over different variables.

**Case: a loop closure with `let`**

```javascript
const fns = [];
for (let i = 0; i < 3; i += 1) {
  fns.push(() => i);
}
console.log(fns.map((fn) => fn()));
```

Result: `[0, 1, 2]`.

Why: each pass has its own `i` because `let` is block-scoped to the loop. `var i` would make every function print `3`, the value `i` has after the loop.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `var` in a loop of functions | Every function sees the final value | `let` |
| Expecting a closed-over counter to reset | It keeps incrementing | Call the outer function again for a new counter |
| A name inside `if` used after the block | `ReferenceError` | Declare it outside if you still need it |
| A giant global for "just this one value" | Any script can overwrite it | A `let` in the smallest block, or a module |

### Tip

If two buttons share a counter and should not, you called the factory once and reused the function. Call the factory twice.

### Practice

Using `makeCounter` from this section, print `1` then `2` from one counter.

### Solution

```javascript
function makeCounter() {
  let n = 0;
  return function () {
    n += 1;
    return n;
  };
}
const next = makeCounter();
console.log(next());
console.log(next());
```

Result:

```text
1
2
```

## 7. Objects

### Concept

An object is a set of properties. `card.city` and `card["city"]` read the same property. A method is a function stored on the object. Inside a method defined with `method()` syntax, `this` is the object before the dot at the call. An arrow does not get that `this`. Destructuring pulls properties into local names. A spread `{ ...card, price: 1 }` copies the top level and then overrides `price`. Nested objects are still shared.

### Terms

| Term | Meaning |
| --- | --- |
| Property | A name and a value |
| Method | A function that uses `this` |
| Destructuring | `const { city } = card` |
| Spread | Copy the top-level properties into a new object |
| Shallow copy | Nested objects are the same objects, not clones |

### Methods

| Operation | Code |
| --- | --- |
| Read | `card.city` or `card["city"]` |
| Write | `card.beds = 3` |
| Missing property | `undefined`. It does not throw |
| Delete | `delete card.beds` |
| Keys | `Object.keys(card)`, `Object.values`, `Object.entries` |
| Copy top level | `{ ...card }` |
| Freeze | `Object.freeze(card)` stops new, changed, and deleted properties |
| Test a key | `Object.hasOwn(card, "city")` |

### Examples

**Case: a method's `this` depends on the call**

```javascript
const card = {
  city: "Irvine",
  label() {
    return this.city;
  },
};
console.log(card.label());
const label = card.label;
console.log(label());
```

Result: the first log is `Irvine`. The second is `undefined` in strict mode, because `this` is not `card` when the function is called bare.

Why: `this` is set by the call. `card.label()` sets it to `card`. Pulling the function off the object loses that. An arrow would have closed over the outer `this` instead, which is also not `card` if you define it in a plain object literal at the top level.

**Case: a missing property is `undefined`**

```javascript
const card = { city: "Irvine" };
console.log(card.beds);
console.log(card.beds.note);
```

Result: the first log is `undefined`. The second throws `TypeError`.

Why: reading a missing property does not throw. Reading a property of that `undefined` does. `card.beds?.note` would stop and return `undefined`.

**Case: spread is shallow**

```javascript
const card = { city: "Irvine", agent: { name: "Ada" } };
const copy = { ...card, city: "Tustin" };
copy.agent.name = "Bea";
console.log(card.city);
console.log(card.agent.name);
```

Result:

```text
Irvine
Bea
```

Why: `city` was overwritten on the new object, so the original city stays `Irvine`. `agent` is a shared nested object. Changing `copy.agent.name` changes `card.agent.name`.

**Case: destructuring and a default**

```javascript
const card = { city: "Irvine" };
const { city, beds = 0 } = card;
console.log(city);
console.log(beds);
```

Result:

```text
Irvine
0
```

Why: `beds` is missing, so the default applies. If `beds` were `0` on the object, the default would not apply. Defaults here follow the same `undefined` rule as function defaults.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Passing a method as a callback | `this` is lost | Wrap it: `() => card.label()` |
| Spread treated as a deep clone | Nested objects are shared | Copy the nested object too, or structured-clone when you need a real copy |
| `card.beds.note` when `beds` is missing | `TypeError` | `card.beds?.note` |
| `Object.keys` expected to show inherited methods | It shows the object's own keys | That is usually what you want |

### Tip

After `{ ...card }`, change a nested field and print the original. If the original changed, the copy was shallow.

### Practice

Copy `card = { city: "Irvine", price: 1 }` into a new object whose price is `2`, and print both prices.

### Solution

```javascript
const card = { city: "Irvine", price: 1 };
const copy = { ...card, price: 2 };
console.log(card.price);
console.log(copy.price);
```

Result:

```text
1
2
```

## 8. Arrays

### Concept

An array is an ordered list. Index `0` is first. `length` is one past the last index. Methods that return a new array do not change the original. `push`, `pop`, `splice`, `sort`, and `reverse` do change it. `sort` with no compare function sorts as strings, so `10` comes before `2`. `map` builds a new array of the same length. `filter` may be shorter. `reduce` folds the array into one value. The second argument of `reduce` is the starting total. Without it, the first element becomes the starting total.

### Terms

| Term | Meaning |
| --- | --- |
| Index | The position. The first is `0` |
| Mutate | Change the existing array |
| Copy | A new array. The old one stays |
| Callback | The function `map` or `filter` calls for each item |

### Methods

| Method | Mutates? | Result |
| --- | --- | --- |
| `push` / `pop` | Yes | Add or remove at the end. `push` returns the new length. `pop` returns the item |
| `unshift` / `shift` | Yes | Add or remove at the start |
| `splice(i, n, ...items)` | Yes | Remove `n` at index `i` and insert items |
| `sort` | Yes | Sort in place. Pass `(a, b) => a - b` for numbers |
| `reverse` | Yes | Reverse in place |
| `slice(start, end)` | No | A copy. `end` is excluded |
| `concat` | No | A new array |
| `map` | No | One output per item |
| `filter` | No | Items that pass the test |
| `find` / `findIndex` | No | The first match, or `undefined` / `-1` |
| `some` / `every` | No | True if any, or all, pass |
| `includes` | No | `===` membership |
| `reduce` | No | One accumulated value |
| `at(-1)` | No | The last item |

`Array.isArray(value)` is the array test.

### Examples

**Case: `slice` copies, `splice` edits**

```javascript
const prices = [10, 20, 30, 40];
console.log(prices.slice(1, 3));
console.log(prices);
prices.splice(1, 2);
console.log(prices);
```

Result:

```text
[20, 30]
[10, 20, 30, 40]
[10, 40]
```

Why: `slice(1, 3)` copies indexes 1 and 2 and stops before 3. The original stays. `splice(1, 2)` removes two items starting at index 1.

**Case: `map` and `filter` do not edit the source**

```javascript
const prices = [999000, 1100000];
const labels = prices.map((price) => `$${price}`);
const high = prices.filter((price) => price >= 1100000);
console.log(labels);
console.log(high);
console.log(prices);
```

Result:

```text
["$999000", "$1100000"]
[1100000]
[999000, 1100000]
```

Why: both methods return a new array. Forgetting to use the return value leaves you holding the original.

**Case: `reduce` needs a starting value**

```javascript
const prices = [10, 20, 30];
const total = prices.reduce((sum, price) => sum + price, 0);
console.log(total);
```

Result: `60`.

Why: `0` is the starting `sum`. The function returns the next sum. Without the `0`, the first number is used as the start, which works for a sum of numbers and fails for an empty array.

**Case: numeric `sort`**

```javascript
const prices = [10, 2, 30];
console.log(prices.sort());
console.log(prices.sort((a, b) => a - b));
```

Result: the first sort is `[10, 2, 30]` because `"10" < "2"` is false and the default compare is string order. After the numeric compare the same array is `[2, 10, 30]`.

Why: `sort` mutates. The string order is not numeric order. `a - b` is negative when `a` should come first.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `sort()` on numbers | `10` can land before `2` | `(a, b) => a - b` |
| Using `map` for a side effect and ignoring the array | You built an array of `undefined` and did not use it | `for...of`, or use the returned array |
| `slice` expected to delete | The original is unchanged | `splice` |
| `reduce` with no start on an empty array | `TypeError` | Pass the starting value |

### Tip

If the method's name is `slice`, `map`, `filter`, `concat`, or `reduce`, look at what it returns. The array you called it on is the same array.

### Practice

From `[10, 20, 30]`, build the labels `$10`, `$20`, `$30` and the total `60` without changing the original array.

### Solution

```javascript
const prices = [10, 20, 30];
const labels = prices.map((price) => `$${price}`);
const total = prices.reduce((sum, price) => sum + price, 0);
console.log(labels);
console.log(total);
console.log(prices);
```

Result:

```text
["$10", "$20", "$30"]
60
[10, 20, 30]
```

## 9. Strings, numbers, and dates

### Concept

String methods return a new string. `slice` keeps negative indexes. `substring` turns negatives into `0` and swaps the ends if they are backwards. `replace` replaces the first match unless the pattern is a global regex. `replaceAll` replaces every literal match. Numbers are floating point. `0.1 + 0.2 === 0.3` is false. Store money as integer cents if you need exactness. Months in `new Date(year, month, day)` are `0` through `11`.

### Terms

| Term | Meaning |
| --- | --- |
| UTF-16 unit | What `length` and `[i]` count. Some characters are two units |
| `NaN` | Not a number. `NaN === NaN` is false |
| Radix | The base for `parseInt`. Pass `10` |
| Month index | January is `0` |

### Methods

| String method | Result |
| --- | --- |
| `length` | UTF-16 units |
| `at(i)` / `[i]` | One unit. `at` accepts a negative index |
| `toLowerCase` / `toUpperCase` | A new string |
| `trim` | Strip both ends |
| `includes` / `startsWith` / `endsWith` | Boolean |
| `indexOf` | Index or `-1` |
| `slice(start, end)` | `end` excluded. Negatives count from the end |
| `substring(start, end)` | Negatives become `0`. Ends are swapped if `start > end` |
| `split` | An array. A second argument caps the pieces |
| `replace` / `replaceAll` | A new string |
| `padStart(n, ch)` | Pad on the left until length `n` |
| `repeat(n)` | The string `n` times |

| Number tool | Use |
| --- | --- |
| `Number.isNaN(n)` | True only for `NaN`. Global `isNaN` converts first |
| `Number.isFinite(n)` | False for `NaN` and infinities |
| `Math.round` / `floor` / `ceil` / `trunc` | Rounding |
| `n.toFixed(2)` | A string, not a number |

### Examples

**Case: the original string does not change**

```javascript
const city = "  Irvine  ";
console.log(city.trim());
console.log(city);
```

Result:

```text
Irvine
  Irvine
```

Why: `trim` returns a new string. `city` still has the spaces. Use the return value.

**Case: `slice` versus `substring`**

```javascript
console.log("HOA".slice(0, 2));
console.log("HOA".slice(-2));
console.log("HOA".substring(2, 0));
```

Result:

```text
HO
OA
HO
```

Why: `slice(0, 2)` stops before index 2. `slice(-2)` takes the last two units. `substring` swaps `2` and `0`, so it becomes `substring(0, 2)`.

**Case: `replace` once, `replaceAll` every time**

```javascript
console.log("ha".replace("h", "H"));
console.log("hah".replaceAll("h", "H"));
```

Result:

```text
Ha
HaH
```

Why: `replace` with a string changes the first match. `replaceAll` changes each one. A regex needs the `g` flag to replace more than once: `"hah".replace(/h/g, "H")`.

**Case: floating point and months**

```javascript
console.log(0.1 + 0.2 === 0.3);
console.log(new Date(2026, 0, 27).getMonth());
```

Result:

```text
false
0
```

Why: the sum is a tiny bit off `0.3`. `getMonth()` returns the index. January was passed as `0`, so the result is `0`, not `1`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Ignoring the return value of `trim` | The old string is still spaced | Assign the result |
| `substring` with a negative index | It becomes `0` | `slice` |
| `=== 0.3` after adding tenths | False | Compare cents, or accept a tolerance |
| `new Date(2026, 1, 1)` for January | That is February | Month `0` |

### Tip

Print `city` and `city.trim()` on two lines the first time. The difference is the whole lesson.

### Practice

Turn `"  irvine  "` into `"Irvine"` by trimming, lower-casing, and then upper-casing the first character.

### Solution

```javascript
const raw = "  irvine  ";
const lower = raw.trim().toLowerCase();
const name = lower[0].toUpperCase() + lower.slice(1);
console.log(name);
```

Result: `Irvine`.

## 10. Set and Map

### Concept

A `Set` stores unique values compared with SameValueZero, which is `===` except that `NaN` matches `NaN`. A `Map` stores pairs. Its keys can be objects, not only strings. An object used as a map coerces keys to strings, so `obj[1]` and `obj["1"]` are the same property. `Set` and `Map` remember insertion order. `WeakMap` keys must be objects. It is not iterable. The entry can disappear when nothing else points at the key.

### Terms

| Term | Meaning |
| --- | --- |
| `Set` | Unique values |
| `Map` | Keys of any type, paired with values |
| Object as a map | String keys only, plus inheritance surprises |
| `WeakMap` | Object keys, not iterable, for a cache attached to an object |

### Methods

| Need | Use |
| --- | --- |
| Ordered list, duplicates, numeric index | Array |
| Unique values and `has` | `Set` |
| String keys and JSON | Object |
| Object keys, or frequent add and delete | `Map` |
| A cache that must not keep an object alive | `WeakMap` or `WeakSet` |

| `Set` | `Map` |
| --- | --- |
| `add(value)` | `set(key, value)` |
| `has(value)` | `has(key)` |
| `delete(value)` | `delete(key)` |
| `size` | `size` |
| no `get` | `get(key)`, or `undefined` |

### Examples

**Case: a set drops the duplicate**

```javascript
const cities = new Set(["Irvine", "Irvine", "Tustin"]);
console.log(cities.size);
cities.add("San Diego");
console.log(cities.size);
console.log(cities.has("Irvine"));
cities.delete("Tustin");
console.log([...cities]);
```

Result:

```text
2
3
true
["Irvine", "San Diego"]
```

Why: the second `"Irvine"` was not stored. `size` starts at 2. Spread walks insertion order, which is not alphabetical.

**Case: object keys collapse**

```javascript
const bag = {};
bag[1] = "number";
bag["1"] = "string";
console.log(bag[1]);
const map = new Map();
map.set(1, "number");
map.set("1", "string");
console.log(map.get(1));
console.log(map.size);
```

Result:

```text
string
number
2
```

Why: an object's keys are strings (or symbols). `1` became `"1"` and the second write replaced the first. A `Map` keeps a number key and a string key as two entries.

**Case: `get` on a missing key**

```javascript
const byId = new Map();
byId.set("R100", { city: "Irvine" });
console.log(byId.get("R100").city);
console.log(byId.get("R200"));
```

Result:

```text
Irvine
undefined
```

Why: `get` returns `undefined` when the key is absent. It does not throw. `has` is how you tell "stored `undefined`" from "missing", if you ever store `undefined`.

**Case: a `WeakMap` is not a list**

```javascript
const cache = new WeakMap();
const card = { city: "Irvine" };
cache.set(card, "loaded");
console.log(cache.get(card));
```

Result: `loaded`.

Why: the key is the object, not the string `"Irvine"`. You cannot loop the `WeakMap` to print every card. When `card` is no longer used anywhere else, the cache entry can be collected.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Using an object when the key is an id that might be a number | `"1"` and `1` collide | `Map`, or stringify on purpose |
| `set.size()` | Throws. `size` is a property | `set.size` |
| Expecting `Set` to sort | It keeps insertion order | Sort a copy if you need alphabetical order |
| Looping a `WeakMap` | It is not iterable | Use a `Map` when you need to list the keys |

### Tip

If you would write `if (obj[id])`, and `id` might be a number, use a `Map` before you have spent an hour on a overwritten key.

### Practice

Build a `Set` from `["Irvine", "Irvine", "Tustin"]` and print its size and whether it has `"Irvine"`.

### Solution

```javascript
const cities = new Set(["Irvine", "Irvine", "Tustin"]);
console.log(cities.size);
console.log(cities.has("Irvine"));
```

Result:

```text
2
true
```

## 11. JSON, errors, and modules

### Concept

`JSON.stringify` turns a value into text. `JSON.parse` turns that text back. Functions, `undefined` properties, and symbols are dropped. A date becomes a string. `try` / `catch` handles a throw. `catch (error)` receives the thrown value. `finally` runs either way. A module file uses `export` and `import`. Top-level names stay private to the file unless exported. Modules are strict and deferred.

### Terms

| Term | Meaning |
| --- | --- |
| JSON | Text. Not a JavaScript object until you parse it |
| Throw | Leave the function until a `catch`, or crash |
| Module | A file with its own scope and `export` |
| Default export | `export default`. One main value |
| Named export | `export function load` . Import it by name |

### Methods

| Call | Result |
| --- | --- |
| `JSON.stringify({ city: "Irvine" })` | `'{"city":"Irvine"}'` |
| `JSON.parse(text)` | The value. Throws on broken text |
| `throw new Error("missing")` | An error with a message and a stack |
| `error.message` | The string you passed |
| `export function f() {}` | A named export |
| `import { f } from "./file.js"` | The binding. The path needs the extension in the browser |

### Examples

**Case: stringify drops a function**

```javascript
const card = {
  city: "Irvine",
  label() {
    return this.city;
  },
};
console.log(JSON.stringify(card));
```

Result: `{"city":"Irvine"}`.

Why: JSON has data, not functions. `label` is gone. Parsing the text does not bring it back.

**Case: parse throws on a trailing comma**

```javascript
try {
  console.log(JSON.parse('{ "city": "Irvine", }'));
} catch (error) {
  console.log(error.name);
}
```

Result: `SyntaxError`.

Why: that text is legal JavaScript and illegal JSON. `catch` runs. Without `try`, the script stops.

**Case: `finally` runs after a return**

```javascript
function load() {
  try {
    return "Irvine";
  } finally {
    console.log("done");
  }
}
console.log(load());
```

Result:

```text
done
Irvine
```

Why: `finally` runs before the caller receives the return value. It also runs when the `try` throws.

**Case: a named export**

```javascript
// city.js
export function label(city) {
  return city.toUpperCase();
}
```

```javascript
// main.js
import { label } from "./city.js";
console.log(label("Irvine"));
```

Result: `IRVINE`, when `main.js` is loaded as a module (`<script type="module" src="main.js">` or `node main.js` with the right type).

Why: `label` is not global. Another file cannot see it without importing. The import name has to match the export name.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `JSON.parse` on an object | Throws, or stringifies oddly if you passed the wrong type | Parse text. Do not parse a value you already have |
| A trailing comma in JSON | `SyntaxError` | Remove it. JSON is stricter than JavaScript |
| Empty `catch` | The error disappears | Log `error.message`, or rethrow |
| Importing a classic script as if it exported | No exports | `type="module"` and `export` |

### Tip

If `JSON.parse` throws, the text is not JSON. Log the first 80 characters before you look at your object code.

### Practice

Stringify `{ city: "Irvine" }`, parse that text, and print the city.

### Solution

```javascript
const text = JSON.stringify({ city: "Irvine" });
const card = JSON.parse(text);
console.log(card.city);
```

Result: `Irvine`.

## 12. Classes

### Concept

A class is a way to build objects that share methods. `new Listing("Irvine")` runs `constructor` and returns the object. Methods live on the prototype, not as a fresh copy on every object. `extends` sets the prototype chain. `super` in the constructor must run before you use `this`. A child method overrides a parent method. The variable's type is not checked, because JavaScript has no variable types. The object you constructed decides which method runs.

### Terms

| Term | Meaning |
| --- | --- |
| `constructor` | The function `new` calls |
| Prototype | The shared place methods live |
| `extends` | The parent class |
| `super(...)` | The parent constructor |
| Override | A child method with the same name |

### Methods

| Form | Meaning |
| --- | --- |
| `class Listing { constructor(city) { this.city = city; } }` | Instances have a `city` field |
| `label() { return this.city; }` | A method. `this` is the object before the dot |
| `class Condo extends Listing` | `Condo` instances can use `Listing` methods |
| `static count` | A value on the class, not on each object |

### Examples

**Case: `new` sets the fields**

```javascript
class Listing {
  constructor(city) {
    this.city = city;
  }
  label() {
    return this.city;
  }
}
const home = new Listing("Irvine");
console.log(home.label());
```

Result: `Irvine`.

Why: `new` creates the object, runs the constructor, and returns it. `home.label()` sets `this` to `home`.

**Case: a child overrides, `super` still reaches the parent**

```javascript
class Condo extends Listing {
  constructor(city, hoa) {
    super(city);
    this.hoa = hoa;
  }
  label() {
    return `${super.label()} (condo)`;
  }
}
const home = new Condo("Irvine", 350);
console.log(home.label());
console.log(home.hoa);
```

Result:

```text
Irvine (condo)
350
```

Why: `home` is a `Condo`, so `label` is the child's method. `super.label()` runs `Listing`'s method on this same object. Forgetting `super(city)` before `this.hoa = hoa` throws `ReferenceError`.

**Case: the variable does not hide the child method**

```javascript
const home = new Condo("Irvine", 350);
console.log(home.label());
```

Result: `Irvine (condo)`, whether you think of `home` as a listing or a condo.

Why: there is no compile-time variable type. The object is a `Condo`, so the override runs. A parent-only method that the child does not have is still callable if it lives on `Listing`.

**Case: a static value is shared**

```javascript
class Listing {
  static count = 0;
  constructor() {
    Listing.count += 1;
  }
}
new Listing();
new Listing();
console.log(Listing.count);
```

Result: `2`.

Why: `count` sits on the class. It is not a field of either object. `home.count` would be `undefined` unless you also set an instance field.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Calling `Listing("Irvine")` without `new` | `this` is wrong and strict mode throws | `new Listing` |
| Using `this` before `super` in a child | `ReferenceError` | Call `super(...)` first |
| An arrow as a method, expecting `this` to be the object | The arrow uses the outer `this` | Use `label() {}` |
| Copying methods into the constructor | Every instance gets its own function | Put methods in the class body |

### Tip

`new` and `super` are the two calls that build the object. If construction throws, one of them is missing or out of order.

### Practice

Create a `Condo` with city `Tustin` and hoa `400`. Print the label from the override in this section.

### Solution

```javascript
const home = new Condo("Tustin", 400);
console.log(home.label());
```

Result: `Tustin (condo)`, using the classes from the examples above.

## 13. Promises and async

### Concept

A promise is a value that is not ready yet. It starts pending. It settles once, as fulfilled with a value or rejected with a reason. `then` registers work for fulfillment. `catch` registers work for rejection. Both return a new promise. `async` makes a function return a promise. `await` pauses that function until the promise settles. It does not freeze the page or the rest of the program. A rejection at `await` becomes a throw, so `try` / `catch` handles it.

`fetch` returns a promise. The promise rejects when the network fails. An HTTP 404 still fulfills the promise. You have to check `response.ok`. That check is in section 15. This section is the promise itself.

### Terms

| Term | Meaning |
| --- | --- |
| Pending | No value yet |
| Fulfilled | Settled with a value |
| Rejected | Settled with a reason |
| Settle | Fulfill or reject. It happens once |
| `async` | This function's return value is wrapped in a promise |
| `await` | Pause this function, not the thread |

### Methods

| Call | What it does |
| --- | --- |
| `Promise.resolve(value)` | An already fulfilled promise |
| `Promise.reject(reason)` | An already rejected promise |
| `promise.then(fn)` | Run `fn` with the value. Returns a new promise of `fn`'s return value |
| `promise.catch(fn)` | Run `fn` with the reason |
| `promise.finally(fn)` | Run either way. The value passes through |
| `Promise.all(list)` | Wait for every promise. Rejects if one rejects |
| `Promise.allSettled(list)` | Wait for every promise and report each outcome |
| `Promise.race(list)` | The first one to settle |

### Examples

**Case: `await` unwraps the value**

```javascript
function wait(ms) {
  return new Promise((resolve) => {
    setTimeout(() => resolve("done"), ms);
  });
}

async function load() {
  const value = await wait(10);
  console.log(value);
}
load();
console.log("after call");
```

Result: the lines print in this order:

```text
after call
done
```

Why: `load()` returns a promise immediately. The log after the call runs before the timer. Ten milliseconds later `await` receives `"done"` and the function continues. `await` did not freeze the script.

**Case: forgetting `await`**

```javascript
async function load() {
  const value = wait(10);
  console.log(value);
}
load();
```

Result: the log prints a `Promise`, not `"done"`.

Why: `wait(10)` is the promise. Without `await`, `value` is that object. You cannot read the string off it. `await` is what turns the promise into the string `"done"`.

**Case: a rejection becomes a throw**

```javascript
function fail() {
  return Promise.reject(new Error("nope"));
}

async function load() {
  try {
    await fail();
    console.log("continued");
  } catch (error) {
    console.log(error.message);
  }
}
load();
```

Result: `nope`. `continued` does not print.

Why: `await` on a rejected promise throws. `catch` receives the `Error`. The lines after `await` in the `try` do not run. Without `try` / `catch`, the promise returned by `load` rejects, and nothing in this file is listening.

**Case: `then` and `await` are the same settlement**

```javascript
Promise.resolve(1)
  .then((n) => n + 1)
  .then((n) => {
    console.log(n);
  });
```

Result: `2`.

Why: `then` passes the return value to the next `then`. Forgetting to `return` inside `then` makes the next one receive `undefined`. `await` is the same chain written top to bottom: `const n = await Promise.resolve(1); console.log(n + 1)`.

**Case: `Promise.all` fails fast, `allSettled` does not**

```javascript
const ok = Promise.resolve("Irvine");
const bad = Promise.reject(new Error("nope"));

Promise.all([ok, bad]).catch((error) => {
  console.log(error.message);
});

Promise.allSettled([ok, bad]).then((results) => {
  console.log(results.map((result) => result.status));
});
```

Result:

```text
nope
["fulfilled", "rejected"]
```

Why: `all` rejects when one promise rejects, so you do not get the fulfilled values in the success path. `allSettled` waits for both and tells you the status of each. Use `all` when one failure should cancel the group. Use `allSettled` when you want every outcome.

**Case: a slow response must not overwrite a newer one**

```javascript
let latest = 0;
async function search(id, delay, city) {
  const request = ++latest;
  await wait(delay);
  if (request !== latest) {
    return;
  }
  console.log(city);
}
search(1, 30, "Irvine");
search(2, 10, "Tustin");
```

Result: `Tustin` only.

Why: Irvine's timer is slower. It finishes last, but `latest` is already `2`, so the stale call returns without printing. If you delete the `if`, the log is `Tustin` and then `Irvine`, and the older search wins the screen.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Using the promise as if it were the value | You print `Promise { ... }` or read `undefined` fields | `await` it |
| `try` / `catch` around a call with no `await` | The rejection is not caught | `await` inside the `try` |
| No `return` inside `.then` | The next `.then` gets `undefined` | Return the value |
| `Promise.all` when you needed every result | One rejection hides the others | `allSettled` |
| A late response writes over a newer one | The user sees the old search | Ignore the result when a newer request has started |

### Tip

Say what the variable holds before you use it. "This is a promise" means you still need `await`. "This is the city string" means `await` already happened.

### Practice

Write `async function load()` that awaits `Promise.resolve("Irvine")` and returns that string. Call it and print the string from a `.then`.

### Solution

```javascript
async function load() {
  return await Promise.resolve("Irvine");
}
load().then((city) => {
  console.log(city);
});
```

Result: `Irvine`.

`load()` itself is a promise. The `.then` is what receives the string in the caller. `return await` is optional here. `return "Irvine"` from an `async` function is already wrapped in a fulfilled promise.

## 14. The DOM and events

### Concept

The DOM is the live tree of the page. `document.getElementById` and `querySelector` find elements. `textContent` is text. `innerHTML` parses HTML, so user text passed to it can become markup. Events bubble from the target up through the parents. A listener on a parent sees the child's click. `event.target` is the element that was clicked. `event.currentTarget` is the element the listener is attached to. `preventDefault` stops the browser's default, such as a form submit.

### Terms

| Term | Meaning |
| --- | --- |
| Node | An element, a text node, or the document |
| `textContent` | The text, with tags shown as text |
| `innerHTML` | Parsed HTML. Dangerous with untrusted strings |
| Bubble | The event travels from the target toward the document |
| `submit` | A form event. Prevent it to stay on the page |

### Methods

| Call | Result |
| --- | --- |
| `document.getElementById("city")` | The element, or `null` |
| `document.querySelector(".card")` | The first match, or `null` |
| `element.textContent = "Irvine"` | Replaces the text |
| `element.addEventListener("click", fn)` | Runs `fn` on that event |
| `event.preventDefault()` | Cancels the default action |
| `element.dataset.city` | The `data-city` attribute |

### Examples

**Case: `textContent` does not parse tags**

```html
<p id="label"></p>
```

```javascript
const label = document.getElementById("label");
label.textContent = "<strong>Irvine</strong>";
```

Result: the paragraph shows the characters `<strong>Irvine</strong>`. It is not bold.

Why: `textContent` is plain text. `innerHTML` would create a `strong` element. Use `textContent` for a city a user typed.

**Case: a missing element**

```javascript
const label = document.getElementById("missing");
console.log(label);
label.textContent = "Irvine";
```

Result: the log is `null`. The next line throws `TypeError`.

Why: the id is wrong, or the script ran before the element existed. Check for `null` before you use the element, and load the script with `defer`.

**Case: bubbling**

```html
<div id="card"><button id="save">Save</button></div>
```

```javascript
document.getElementById("card").addEventListener("click", () => {
  console.log("card");
});
document.getElementById("save").addEventListener("click", () => {
  console.log("save");
});
```

Result: clicking the button logs `save`, then `card`.

Why: the target is the button. The event then bubbles to the div. `stopPropagation` on the button would stop the card from seeing it. You rarely need that.

**Case: prevent a submit**

```html
<form id="search">
  <input name="city">
  <button type="submit">Search</button>
</form>
```

```javascript
document.getElementById("search").addEventListener("submit", (event) => {
  event.preventDefault();
  console.log("staying");
});
```

Result: the console shows `staying`. The browser does not navigate.

Why: the default of submit is a navigation. `preventDefault` cancels that. The listener still has to read the fields and do the search.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `innerHTML` with a typed city | A `<` in the city becomes markup | `textContent` |
| Calling a method on a `null` query | `TypeError` | Test for `null`. Check the id and `defer` |
| Listening for `click` on a submit button and also letting the form submit | The page navigates anyway | Listen for `submit` and `preventDefault` |
| `getElementById` with the `#` | Always `null` | Pass `"city"`, not `"#city"` |

### Tip

`querySelector` uses a CSS selector, so it wants `#city`. `getElementById` wants `city` with no hash.

### Practice

Find `#label` and set its text to `Irvine` without parsing HTML.

### Solution

```javascript
document.querySelector("#label").textContent = "Irvine";
```

Result: the element shows `Irvine`. If the query returns `null`, this throws. The element has to exist.

## 15. Fetch and storage

### Concept

`fetch(url)` returns a promise of a `Response`. The promise rejects when the network fails. A 404 or a 500 still fulfills the promise. `response.ok` is true for status 200 through 299. `response.json()` is another promise. It parses the body. `localStorage` stores strings in the browser for this origin. It survives reloads. It is not a database and not private from scripts on the same origin. `sessionStorage` lasts until the tab closes.

### Terms

| Term | Meaning |
| --- | --- |
| `response.ok` | Status 200–299 |
| Network rejection | DNS, offline, or CORS blocked the request |
| HTTP error | The server answered. `fetch` fulfilled |
| `localStorage` | String pairs kept for the origin |
| JSON body | Text you still have to parse |

### Methods

| Call | Result |
| --- | --- |
| `fetch(url)` | A promise of a `Response` |
| `response.ok` | Boolean |
| `response.status` | The number, such as `404` |
| `response.json()` | A promise of the parsed body |
| `response.text()` | A promise of the raw text |
| `localStorage.setItem(key, value)` | Stores a string |
| `localStorage.getItem(key)` | The string, or `null` |
| `JSON.stringify` / `JSON.parse` | How you store an object in `localStorage` |

### Examples

**Case: a 404 does not throw**

```javascript
async function load(url) {
  const response = await fetch(url);
  console.log(response.ok);
  console.log(response.status);
}
```

Result: for a missing page, `ok` is `false` and `status` is `404`. The `await` did not throw.

Why: the server answered. That is fulfillment. Check `ok` before you parse the body. Throw your own error if you want `catch` to handle HTTP failures:

```javascript
if (!response.ok) {
  throw new Error(`request failed: ${response.status}`);
}
```

**Case: `json` is a second await**

```javascript
async function loadCity(url) {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error(String(response.status));
  }
  const card = await response.json();
  console.log(card.city);
}
```

Result: when the body is `{"city":"Irvine"}` and the status is 200, the log is `Irvine`.

Why: `response` is not the JSON. `response.json()` reads the body and returns a promise. Forgetting the second `await` leaves `card` as a promise, and `card.city` is `undefined`.

**Case: storage keeps strings**

```javascript
localStorage.setItem("city", "Irvine");
console.log(localStorage.getItem("city"));
localStorage.setItem("card", JSON.stringify({ city: "Irvine" }));
console.log(JSON.parse(localStorage.getItem("card")).city);
```

Result:

```text
Irvine
Irvine
```

Why: `setItem` stores text. An object becomes `"[object Object]"` if you pass it without `JSON.stringify`. `getItem` of a missing key is `null`, and `JSON.parse(null)` throws. Check for `null` first.

**Case: a stale fetch must not paint**

```javascript
let latest = 0;
async function show(id, url) {
  const request = ++latest;
  const response = await fetch(url);
  if (request !== latest) {
    return;
  }
  const card = await response.json();
  document.querySelector("#label").textContent = card.city;
}
```

Result: if the user starts a second search before the first response arrives, only the second response changes the label.

Why: `await` gives other code time to run. `latest` records which call is current. The slower call sees a newer id and returns. This is the same idea as the timer in section 13, applied to `fetch`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Treating a 404 as a thrown error | `catch` does not run | Check `response.ok` |
| One `await` and then `response.city` | `undefined`. The city is in the body | `await response.json()` |
| `localStorage.setItem("card", card)` | The string `"[object Object]"` | `JSON.stringify` |
| Two searches, and the slower one paints last | The label shows the old city | Ignore responses that are no longer `latest` |

### Tip

Count the awaits in a fetch. One for the response. One for the body. Then ask whether this response is still the latest request.

### Practice

Write `load(url)` that throws when `response.ok` is false and otherwise returns the parsed JSON.

### Solution

```javascript
async function load(url) {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error(`request failed: ${response.status}`);
  }
  return await response.json();
}
```

Result: a 200 with `{"city":"Irvine"}` resolves to that object. A 404 rejects with `request failed: 404`. A network failure rejects before `response.ok` runs, because `fetch` itself rejected.

## Drill

1. Explain why `0 || "missing"` is `"missing"` and `0 ?? "missing"` is `0`.
2. From `[10, 20, 30]`, make labels and a total without mutating the array.
3. Write `makeCounter` and show that two counters do not share state.
4. `await` a resolved promise of `"Irvine"` and print the string, not the promise.
5. Say what `fetch` does on a 404, and where `response.ok` fits.

### Solution 1

`||` treats `0` as falsy and returns the right side. `??` replaces only `null` and `undefined`, so it returns `0`.

### Solution 2

The practice solution in section 8.

### Solution 3

The practice solution in section 6. A second `makeCounter()` call starts again at `1`.

### Solution 4

The practice solution in section 13.

### Solution 5

`fetch` fulfills on a 404. `response.ok` is false and `response.status` is `404`. Throw if you want that status in `catch`. The promise rejects only when the network fails.
