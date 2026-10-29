# JavaScript

Course 3 of 15. Previous: [02-css.md](02-css.md). Next: [04-typescript.md](04-typescript.md).

JavaScript is the language of the browser and of Node. This course is the language and the browser APIs. TypeScript, React, and Node each have their own course. Examples here are plain JavaScript.

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

In a browser, a `script` element runs the file. In a terminal, `node file.js` runs it. `console.log` prints. A statement ends with a semicolon. Automatic semicolon insertion usually inserts one, and sometimes inserts the wrong one after `return`.

```html
<script src="app.js" defer></script>
```

```javascript
console.log("ready");
```

| Tool | Where the text goes |
| --- | --- |
| `console.log` | The console |
| `console.error` | The console, marked as an error |
| `alert` | A blocking dialog. Fine once, while you learn. Not for a real page |
| `document.body.textContent = "Hi"` | The page |

`"use strict";` at the top of a classic script rejects silent mistakes such as assigning to an undeclared name. Modules and classes are strict already.

## 2. Values and variables

| Type | Examples | `typeof` |
| --- | --- | --- |
| number | `3`, `2.5`, `NaN`, `Infinity` | `"number"` |
| string | `"Irvine"`, `'Irvine'`, `` `Irvine` `` | `"string"` |
| boolean | `true`, `false` | `"boolean"` |
| undefined | a missing value | `"undefined"` |
| object | `{ city: "Irvine" }`, `[1, 2]`, `null` | `"object"` |
| function | `function f() {}` | `"function"` |
| bigint | `10n` | `"bigint"` |
| symbol | `Symbol("id")` | `"symbol"` |

`typeof null` is `"object"`. That is a long-standing mistake in the language. Test null with `value === null`.

```javascript
let city = "Irvine";
city = "San Diego";
const pi = 3.14;
```

| Keyword | Reassign the name? | Scope |
| --- | --- | --- |
| `let` | Yes | The block `{ }` |
| `const` | No | The block |
| `var` | Yes | The function. Avoid it in new code |

`const` protects the binding, not the contents. `const card = { price: 1 }; card.price = 2` is allowed. `card = {}` is not.

A name used before its `let` or `const` line throws. That gap is the temporal dead zone. `var` is hoisted as `undefined`, which hides bugs.

Template strings interpolate and keep line breaks:

```javascript
const beds = 3;
const line = `${beds} beds`;
```

## 3. Operators

| Operator | Meaning |
| --- | --- |
| `+ - * / % **` | Arithmetic. `%` is remainder. `**` is power |
| `+` on strings | Concatenate. `1 + "2"` is `"12"` |
| `===` and `!==` | Equal, no type conversion |
| `==` and `!=` | Equal after conversion. Avoid |
| `< <= > >=` | Order |
| `&&` | First falsy value, or the last value |
| `\|\|` | First truthy value, or the last value |
| `??` | The right side only when the left is `null` or `undefined` |
| `!` | Boolean not |
| `? :` | `cond ? a : b` |
| `+= -= *=` | Update in place |
| `++` | Add one. Easy to misread. Prefer `n += 1` |
| `?.` | Stop if the base is nullish: `user?.city` |
| `??` versus `\|\|` | `0` and `""` are kept by `??` and replaced by `\|\|` |

Falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy, including `"0"`, `"false"`, and `[]`.

```javascript
const price = 0;
const label = price ?? "missing"; // 0
const other = price || "missing"; // "missing"
```

`Number("3")` is `3`. `Number("")` is `0`. `parseInt("10px", 10)` is `10`. Always pass the radix `10`.

## 4. Control flow

```javascript
if (beds >= 3) {
  console.log("large");
} else if (beds === 2) {
  console.log("mid");
} else {
  console.log("small");
}

switch (city) {
  case "Irvine":
  case "Tustin":
    console.log("orange county");
    break;
  default:
    console.log("elsewhere");
}

for (let i = 0; i < 3; i += 1) {
  console.log(i);
}

let n = 3;
while (n > 0) {
  n -= 1;
}
```

| Statement | Effect |
| --- | --- |
| `if / else` | One branch |
| `switch` | Match with `===`. `break` or execution falls into the next case |
| `for` | Count or walk an index |
| `while` | Test, then body |
| `do...while` | Body, then test. Runs at least once |
| `for...of` | Each value of an iterable |
| `for...in` | Each key of an object, including inherited enumerable keys. Prefer `Object.keys` |
| `break` | Leave the loop |
| `continue` | Skip to the next iteration |

`for (const bed of [2, 3, 4])` gives `2`, then `3`, then `4`.

## 5. Functions

```javascript
function toNumber(digits, unit = "") {
  let n = Number(String(digits).replaceAll(",", ""));
  if (unit === "k") n *= 1000;
  if (unit === "m") n *= 1000000;
  return n;
}

const double = (n) => n * 2;
```

| Form | Notes |
| --- | --- |
| `function f(a, b) {}` | Hoisted. Has its own `this` when called as a method |
| `const f = function () {}` | Not hoisted |
| `const f = (a) => a + 1` | No own `this`. One expression can omit `return` and braces |
| Default parameter | Used when the argument is `undefined` |
| Rest | `function sum(...nums)` collects the rest into an array |
| `return` | Hands a value back. Without it, the result is `undefined` |

Arguments are passed by value. The value of an object argument is a reference, so a function can change the object's fields and the caller sees the change. Reassigning the parameter does not reassign the caller's variable.

```javascript
function addCity(card, city) {
  card.city = city;
}
```

A callback is a function you pass so someone else can call it later:

```javascript
[1, 2, 3].map((n) => n * 10);
```

## 6. Scope and closures

A variable is visible in the block where it was declared, and in blocks inside that block. It is not visible outside.

```javascript
function makeCounter() {
  let n = 0;
  return function () {
    n += 1;
    return n;
  };
}
const next = makeCounter();
next(); // 1
next(); // 2
```

The inner function closes over `n`. `n` stays alive after `makeCounter` has returned. That is a closure. Two calls to `makeCounter` get two different `n` variables.

| Scope | Lives |
| --- | --- |
| Block | Between `{` and `}` for `let` and `const` |
| Function | Parameters and `var` |
| Module | Top-level names in a file. Not automatically global |
| Global | `window` in a classic browser script. Avoid adding to it |

## 7. Objects

An object is a set of named properties.

```javascript
const card = {
  city: "Irvine",
  price: 999000,
  label() {
    return `${this.city}: ${this.price}`;
  },
};
card.beds = 3;
const city = card.city;
const { price, beds = 0 } = card;
const copy = { ...card, price: 1000000 };
```

| Operation | Code |
| --- | --- |
| Read | `card.city` or `card["city"]` |
| Write | `card.city = "Tustin"` |
| Missing key | `undefined` |
| Delete | `delete card.beds` |
| Keys | `Object.keys(card)` |
| Values | `Object.values(card)` |
| Pairs | `Object.entries(card)` |
| Copy fields | `{ ...card }` is a shallow copy |
| Merge | `Object.assign({}, a, b)` later sources win |
| Test a key | `Object.hasOwn(card, "city")` |
| Freeze | `Object.freeze(card)` blocks changes |

`this` inside `label()` is the object before the dot when you call `card.label()`. Pull the function off and call it alone, and `this` is wrong. An arrow function does not get its own `this`. It uses the `this` of the surrounding function.

Destructuring can rename: `const { city: town } = card`.

Shorthand: `{ city }` means `{ city: city }`.

Optional chaining: `card.agent?.name` is `undefined` if `agent` is nullish, instead of throwing.

## 8. Arrays

An array is an ordered list. Index `0` is first. `length` is one past the last index.

```javascript
const prices = [999000, 1100000, 1350000];
prices.push(1400000);
const first = prices[0];
const rest = prices.slice(1);
```

Methods that return a new array do not change the original. Methods that mutate do.

| Method | Mutates? | Result |
| --- | --- | --- |
| `push` / `pop` | Yes | Add or remove at the end |
| `unshift` / `shift` | Yes | Add or remove at the start |
| `splice(i, n, ...items)` | Yes | Remove `n` at index `i` and insert items |
| `sort` | Yes | Sort in place. Default sort is as strings. Pass a compare function |
| `reverse` | Yes | Reverse in place |
| `copyWithin`, `fill` | Yes | Write over existing slots |
| `slice(start, end)` | No | A copy of that range. `end` is excluded |
| `concat` | No | A new array joined with others |
| `map` | No | A new array, one output per item |
| `filter` | No | A new array of items that pass the test |
| `find` / `findIndex` | No | The first match, or `undefined` / `-1` |
| `some` / `every` | No | True if any or all pass |
| `includes` | No | True if the value is present, using `===` |
| `indexOf` | No | Index or `-1` |
| `reduce` | No | Fold the array into one value |
| `flat` | No | One level of nesting removed |
| `join` | No | A string |
| `at(-1)` | No | The last item. Negative indexes count from the end |

```javascript
const high = prices.filter((price) => price >= 1100000);
const labels = prices.map((price) => `$${price}`);
const total = prices.reduce((sum, price) => sum + price, 0);
prices.sort((a, b) => a - b);
```

`sort` without a compare function orders numbers wrong: `10` comes before `2` because `"10" < "2"`.

A sparse hole is not the same as `undefined` stored on purpose, but most methods visit both. Do not build arrays with empty slots.

`Array.isArray(value)` is the test. `typeof` an array is `"object"`.

## 9. Strings, numbers, and dates

```javascript
const city = "  los angeles ";
city.trim().toLowerCase();
"3 beds".includes("bed");
"a,b,c".split(",");
["a", "b"].join("-");
"HOA".slice(0, 2);
"hi".replaceAll("h", "H");
```

| String method | Result |
| --- | --- |
| `length` | Character count, UTF-16 code units |
| `at(i)` / `[i]` | One unit. `at` accepts a negative index. `[i]` does not |
| `toLowerCase` / `toUpperCase` | A new string |
| `trim` / `trimStart` / `trimEnd` | Ends stripped |
| `includes` / `startsWith` / `endsWith` | Boolean. The last two take an optional start index |
| `indexOf` / `lastIndexOf` | Index or `-1` |
| `slice(start, end)` | Substring. Negative indexes count from the end. `end` is excluded |
| `substring(start, end)` | Substring. Negative indexes become `0`. If `start > end`, they are swapped |
| `split` | An array. A second argument caps how many pieces |
| `replace` / `replaceAll` | A new string. `replace` without a global regex replaces once |
| `padStart` / `padEnd` | Pad until a length |
| `repeat(n)` | The string `n` times |
| `match` | Regex result or `null` |
| `matchAll` | Every match, from a global regex |
| `search` | Index of a regex match, or `-1` |
| `localeCompare` | Negative, zero, or positive, for sort order |
| `charCodeAt` / `codePointAt` | The numeric code at that index |

Numbers are IEEE floating point. `0.1 + 0.2` is not exactly `0.3`. Money in cents, or integers, avoids that for this course.

| Number tool | Use |
| --- | --- |
| `Number.isNaN(n)` | True only for `NaN`. The global `isNaN` coerces first |
| `Number.isFinite(n)` | False for `NaN` and infinities |
| `Math.round` / `floor` / `ceil` / `trunc` | Rounding |
| `Math.min` / `Math.max` | Spread an array: `Math.max(...prices)` |
| `Math.random()` | From 0 inclusive to 1 exclusive |
| `n.toFixed(2)` | A string, not a number |

```javascript
const today = new Date();
const stamp = Date.parse("2026-10-27T00:00:00Z");
today.getFullYear();
today.toISOString();
```

Months in `new Date(year, month, day)` are `0` through `11`. Prefer ISO strings and `toISOString` when you store a moment.

A regular expression is a pattern. `/beds?/i` matches `bed` or `beds`, case insensitive. `.test` returns a boolean. `.match` returns the captures. Build one from a string with `new RegExp(text, "i")` and escape punctuation if the text came from a user.

## 10. Set and Map

`Set` stores unique values. `Map` stores keys of any type, not only strings.

```javascript
const cities = new Set(["Irvine", "Irvine", "Tustin"]);
cities.add("San Diego");
cities.has("Irvine"); // true
cities.size; // 3
cities.delete("Tustin");

const byId = new Map();
byId.set("R100", { city: "Irvine" });
byId.get("R100");
byId.has("R100");
```

| Need | Use |
| --- | --- |
| Ordered list, duplicates allowed, numeric index | Array |
| Unique values, fast `has` | `Set` |
| String keys and JSON | Object |
| Keys that are objects, or keys you will add and delete often | `Map` |
| Keys that should not keep an object alive | `WeakMap` or `WeakSet` |

`WeakMap` keys must be objects. It is not iterable. The entry disappears if nothing else points at the key. That is for caches attached to objects, not for a list you want to print.

Walk them with `for...of`. A `Map` yields `[key, value]` pairs.

```javascript
for (const city of cities) {
  console.log(city);
}
for (const [id, card] of byId) {
  console.log(id, card.city);
}
```

| Method | `Set` | `Map` |
| --- | --- | --- |
| Add | `add(value)` | `set(key, value)` |
| Read | — | `get(key)` |
| Test | `has(value)` | `has(key)` |
| Remove | `delete(value)` | `delete(key)` |
| Count | `size` | `size` |
| Empty | `clear()` | `clear()` |

Object keys are always strings or symbols. `obj[1]` is the same slot as `obj["1"]`. Map keys are not coerced.

The same jobs, lined up:

| Job | Array | `Set` | `Map` | Object |
| --- | --- | --- | --- | --- |
| Create | `[1, 2]` | `new Set([1, 2])` | `new Map([["a", 1]])` | `{ a: 1 }` |
| Add | `push`, `unshift`, `splice` | `add` | `set(key, value)` | `obj[key] = value` |
| Read | `[i]`, `at(i)` | No index | `get(key)`, or `undefined` | `obj[key]`, or `undefined` |
| Test | `includes` uses `===` | `has` | `has` | `key in obj`, or `Object.hasOwn` |
| Remove | `pop`, `shift`, `splice` | `delete` | `delete` | `delete obj[key]` |
| Count | `length` | `size` | `size` | `Object.keys(obj).length` |
| Empty | `length = 0`, or `splice` | `clear` | `clear` | Delete keys one by one |
| Duplicates | Yes | No | One value per key | One value per key |
| Order | Index order | Insertion order | Insertion order | String keys in insertion order |
| Key type | Indexes only | Any value | Any value | String or symbol |
| Walk | `for...of`, or methods | `for...of` the values | `for...of` the pairs | `for...in`, or `Object.entries` |

## 11. JSON, errors, and modules

JSON is a text format. Keys are double-quoted strings. No functions, no `undefined`, no comments.

```javascript
const text = JSON.stringify({ city: "Irvine", price: 999000 });
const data = JSON.parse(text);
```

| JavaScript | JSON |
| --- | --- |
| `undefined`, functions, symbols | Dropped from objects. `JSON.stringify` returns `undefined` for those values alone |
| `NaN` and `Infinity` | `null` |
| Date | A string if you do not convert it yourself |

```javascript
function parsePrice(text) {
  const n = Number(text);
  if (Number.isNaN(n)) {
    throw new TypeError("price must be a number");
  }
  return n;
}

try {
  parsePrice("abc");
} catch (error) {
  console.error(error.message);
} finally {
  console.log("done");
}
```

| Error | Usual cause |
| --- | --- |
| `TypeError` | Wrong kind of value, or reading a property of `null` |
| `ReferenceError` | Unknown name |
| `SyntaxError` | The file or the JSON text is not valid |
| `RangeError` | A value is outside the allowed range |
| Your `Error` | `throw new Error("not found")` |

`finally` runs whether or not the `try` threw. Use it to close a resource.

A module hides its names. It shares only what it exports. In the browser, `<script type="module" src="app.js">`. In Node, a package can set `"type": "module"`.

```javascript
export function toNumber(digits) {
  return Number(String(digits).replaceAll(",", ""));
}
export const CITIES = ["Irvine"];
```

```javascript
import { toNumber, CITIES } from "./numbers.js";
import * as numbers from "./numbers.js";
```

| Form | Meaning |
| --- | --- |
| `export function f` | A named export |
| `export default function` | One default per file |
| `import { f } from "./f.js"` | Named. The path needs the suffix in the browser |
| `import f from "./f.js"` | The default |

Modules are deferred and strict. Top-level names are not global.

## 12. Classes

A class is a function for building objects that share methods.

```javascript
class Listing {
  constructor(city, price) {
    this.city = city;
    this.price = price;
  }

  label() {
    return `${this.city}: ${this.price}`;
  }
}

class Condo extends Listing {
  constructor(city, price, hoa) {
    super(city, price);
    this.hoa = hoa;
  }
}

const home = new Listing("Irvine", 999000);
home.label();
```

| Piece | Meaning |
| --- | --- |
| `constructor` | Runs on `new`. Put starting fields here |
| `this` | The new object |
| `extends` | The child can use the parent's methods |
| `super` | Call the parent constructor or a parent method |
| `static` | A method on the class itself, not on each object |
| `#price` | A private field. Invisible outside the class |
| `instanceof` | `home instanceof Listing` is true |

Getters and setters look like properties and run code:

```javascript
class Money {
  #cents;
  constructor(cents) { this.#cents = cents; }
  get dollars() { return this.#cents / 100; }
}
```

## 13. Promises and async

A promise is a value that arrives later. It is pending, then fulfilled or rejected.

```javascript
function wait(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function load() {
  try {
    await wait(10);
    return "done";
  } catch (error) {
    return "failed";
  }
}
```

| Word | Meaning |
| --- | --- |
| `new Promise` | You call `resolve(value)` or `reject(error)` |
| `then` | Run when fulfilled |
| `catch` | Run when rejected |
| `async` | The function always returns a promise |
| `await` | Pause this function until the promise settles. Only inside `async`, or at module top level |
| `Promise.all` | Wait for every promise. Rejects if one rejects |
| `Promise.allSettled` | Wait for every promise and report each outcome |
| `Promise.race` | The first one to settle |

`await` does not freeze the page. Other events can run. A slow request can finish after a newer one. If both write the same variable, the one that finishes last wins unless you ignore stale results or run them one at a time.

```javascript
let pending = Promise.resolve();
function runAlone(task) {
  const next = pending.then(task, task);
  pending = next.then(() => {}, () => {});
  return next;
}
```

`setTimeout(fn, ms)` runs `fn` once, later. `setInterval` repeats. `clearTimeout` and `clearInterval` cancel. The delay is a minimum, not a exact appointment.

## 14. The DOM and events

The DOM is the live tree of the page. JavaScript can read it and change it.

```javascript
const form = document.querySelector("#search");
const city = document.querySelector("#city");
const list = document.querySelector("#results");

form.addEventListener("submit", (event) => {
  event.preventDefault();
  const item = document.createElement("li");
  item.textContent = city.value;
  list.append(item);
});
```

| Query | Result |
| --- | --- |
| `document.querySelector(sel)` | The first match, or `null` |
| `document.querySelectorAll(sel)` | A static NodeList of matches |
| `document.getElementById(id)` | That element, or `null` |
| `element.children` | Element children, live |
| `element.parentElement` | The parent element |
| `element.closest(sel)` | The nearest ancestor, or itself, that matches |
| `element.matches(sel)` | True if this element matches |

| Change | Code |
| --- | --- |
| Text | `el.textContent = "Hi"` |
| HTML | `el.innerHTML = "<b>Hi</b>"`. Do not put user text here. It can become markup |
| Attribute | `el.setAttribute("href", "/a")` or `el.href = "/a"` for known properties |
| Class | `el.classList.add("on")`, `.remove`, `.toggle`, `.contains` |
| Create | `document.createElement("li")` |
| Insert | `parent.append(child)`, `prepend`, `before`, `after` |
| Remove | `el.remove()` |
| Style | `el.style.color = "navy"` sets an inline style. Prefer classes |

`textContent` treats the string as text. `innerHTML` parses markup. User input goes through `textContent`.

Events bubble from the target up through ancestors. `addEventListener` listens. `event.target` is the origin. `event.currentTarget` is the element that has this listener. `event.preventDefault()` cancels the browser's default, such as submitting a form or following a link. `stopPropagation` stops the bubble. You rarely need it.

| Event | When |
| --- | --- |
| `click` | A click or activation |
| `submit` | A form submit |
| `input` | The value changed, including each keystroke |
| `change` | A committed change, such as leaving a text field or picking a select |
| `keydown` | A key went down |
| `focus` / `blur` | Entering and leaving. They do not bubble. `focusin` and `focusout` do |

One listener on a parent can handle clicks from children. Check `event.target` and use `closest` to find the row. That is event delegation, and it still works for children you add later.

## 15. Fetch and storage

`fetch` returns a promise of a response. The promise rejects on network failure. An HTTP 404 still fulfills the promise. Check `response.ok`.

```javascript
async function loadListings(url) {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error(`request failed: ${response.status}`);
  }
  return response.json();
}
```

| `fetch` option | Use |
| --- | --- |
| `method` | `"POST"`, `"PUT"`, `"PATCH"`, `"DELETE"`. Default is `GET` |
| `headers` | `{ "Content-Type": "application/json" }` |
| `body` | `JSON.stringify(data)` for JSON |

```javascript
localStorage.setItem("city", "Irvine");
const city = localStorage.getItem("city");
localStorage.removeItem("city");
```

| Store | Lifetime | Sent to the server? |
| --- | --- | --- |
| `localStorage` | Until cleared. About 5 MB. Strings only | No |
| `sessionStorage` | Until the tab closes | No |
| cookie | You set an expiry. Sent on matching requests | Yes |
| memory variable | Until reload | No |

Store JSON by stringifying it. Do not store secrets in `localStorage`. Any script on the page can read it.

The browser also gives you `location` (the URL), `history.pushState` (change the URL without a load), and `navigator`. You do not need them until a page starts acting like an app. Next.js and React cover the app case in later courses.

## Drill

Autocomplete off.

1. Write `unique(items)` that returns a `Set` turned back into an array.
2. Write `total(prices)` with `reduce`.
3. Explain why `0 || "missing"` is `"missing"` and `0 ?? "missing"` is `0`.
4. Add a submit listener that appends an `li` using `textContent`.
