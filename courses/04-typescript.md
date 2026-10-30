# TypeScript

Course 4 of 15. Previous: [03-javascript.md](03-javascript.md). Next: [05-react.md](05-react.md).

TypeScript is JavaScript plus a checker. You write types. The checker reports mistakes before the program runs. The types are erased when the program runs, unless a tool is configured to keep them. At run time you still have ordinary JavaScript values.

Every section has the same parts: the concept, the terms, the methods, several cases with what the checker does, common mistakes, a tip, and a practice with a solution. "Does not compile" means the checker reports an error. The emitted JavaScript may still exist if you ignore that error.

## Contents

1. [What the checker does](#1-what-the-checker-does)
2. [Basic types](#2-basic-types)
3. [Arrays, tuples, and objects](#3-arrays-tuples-and-objects)
4. [Unions, literals, narrowing, and enums](#4-unions-literals-narrowing-and-enums)
5. [Functions](#5-functions)
6. [Interfaces, type aliases, and classes](#6-interfaces-type-aliases-and-classes)
7. [Generics](#7-generics)
8. [Utility types](#8-utility-types)
9. [Modules and configuration](#9-modules-and-configuration)

## 1. What the checker does

### Concept

An annotation tells the checker what a value is allowed to be. Inference fills the type in when you do not write one. An error is a message from the checker. It is not a JavaScript exception unless the same mistake also throws at run time. Erasure means `number` is not present when Node or the browser runs the file. `any` turns the checker off for that value. `unknown` means "I do not know yet", and you must narrow it before you use it.

### Terms

| Idea | Meaning |
| --- | --- |
| Annotation | `: number` on a parameter, `: string` on the return |
| Inference | `const city = "Irvine"` is already a string |
| Error | A checker message. The JavaScript might still run if you ignore it |
| Erasure | Types are removed from the output |
| `any` | The checker steps aside |
| `unknown` | A value you must inspect before using |

### Methods

| Tool | What it does |
| --- | --- |
| `: number` | The parameter must be a number |
| `: string` after `)` | The function must return a string |
| `tsc` | The checker. `noEmit` checks without writing JavaScript |
| `as string` | You tell the checker to trust you. It does not convert the value |

### Examples

**Case: a matching call**

```typescript
function priceLabel(cents: number): string {
  return `$${(cents / 100).toFixed(2)}`;
}
console.log(priceLabel(999000));
```

Result: the checker accepts the file. The program prints `$9990.00`.

Why: `999000` is a number, and the return value is a string. The `: number` and `: string` are gone in the JavaScript that actually runs.

**Case: a string passed to a number parameter**

```typescript
priceLabel("999000");
```

Result: does not compile. The argument is a `string` and the parameter is a `number`.

Why: the checker compares the argument to the annotation. At run time, if you forced the file to run, `` `$${(cents / 100).toFixed(2)}` `` would throw, because a string has no `toFixed`. The checker stops you first.

**Case: inference**

```typescript
const city = "Irvine";
const beds = 3;
```

Result: `city` has the literal type `"Irvine"`. `beds` has the literal type `3`. `city = "Tustin"` does not compile, because `const` cannot be reassigned. Change `const city` to `let city` and the type widens to `string`, so `city = "Tustin"` compiles.

Why: the checker reads the initializer. You do not have to write `: string` when the value already shows the type.

**Case: `any` hides a mistake that `unknown` catches**

```typescript
const loose: any = "Irvine";
loose.toFixed(2);

const raw: unknown = JSON.parse('{"city":"Irvine"}');
raw.city;
```

Result: `loose.toFixed(2)` compiles and throws at run time, because a string has no `toFixed`. `raw.city` does not compile.

Why: `any` disables the check. `unknown` allows the value and forbids using it as an object until you narrow it. `JSON.parse` is typed as `any` in older libraries and as `any` or a generic in newer ones. Treat parsed JSON as `unknown` when you write the annotation yourself.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Ignoring a checker error because Node still ran an old file | The bug is still in the source | Read the error and change the call |
| `any` to silence an error | Later mistakes on that value compile | `unknown`, then a check |
| Expecting `typeof cents` to be `"number"` because of the annotation | `typeof` is a runtime operator. The annotation is already erased | The runtime type is whatever value you passed |
| `as string` on a number | The checker is quiet. The value is still a number | Check the value, then narrow |

### Tip

Read the error as "this value is not that type." The fix is a different value, a narrower type, or a check. `any` and `as` are not the first fix.

### Practice

Write `priceLabel` so `priceLabel(250)` is accepted and `priceLabel("250")` is rejected.

### Solution

```typescript
function priceLabel(cents: number): string {
  return `$${(cents / 100).toFixed(2)}`;
}
```

Result: `priceLabel(250)` compiles and returns `"$2.50"`. `priceLabel("250")` does not compile.

## 2. Basic types

### Concept

The basic types name the JavaScript values. `string | null` means the value is one or the other, and the key is present. `beds?: number` means the key may be absent. Those are different. `null` is not assignable to `string` when `strictNullChecks` is on. `void` means the caller should ignore the return. `never` means the function does not return.

### Terms

| Type | Allows |
| --- | --- |
| `string` / `number` / `boolean` | Text, numbers including `NaN`, `true` and `false` |
| `null` / `undefined` | Only that value |
| `void` | A function that returns nothing useful |
| `never` | A function that always throws or loops |
| `any` | Anything. Avoid |
| `unknown` | Anything, until you check |

| | `any` | `unknown` | `never` |
| --- | --- | --- | --- |
| Assign any value to it | Yes | Yes | No |
| Use it as a `string` with no check | Yes | No | The line is unreachable |
| After a check | Still `any` | Narrows | Already impossible |

### Methods

| Form | The key |
| --- | --- |
| `beds: number \| null` | Always present. The value may be null |
| `beds?: number` | May be absent. When present, a number |
| `beds?: number \| null` | Absent, null, or a number |

### Examples

**Case: `null` is not a string**

```typescript
let city: string = "Irvine";
city = null;
```

Result: does not compile when `strictNullChecks` is on.

Why: `null` has its own type. If a city can be missing, the type is `string | null`.

**Case: a present null versus a missing key**

```typescript
type WithNull = { beds: number | null };
type WithOptional = { beds?: number };

const a: WithNull = { beds: null };
const b: WithOptional = {};
const c: WithNull = {};
```

Result: `a` and `b` compile. `c` does not compile, because `beds` is required.

Why: `| null` changes the value. `?` changes whether the key exists. Reading `b.beds` can be `number | undefined`.

**Case: `void` and `never`**

```typescript
function logCity(city: string): void {
  console.log(city);
}

function fail(message: string): never {
  throw new Error(message);
}
```

Result: `logCity("Irvine")` compiles and prints `Irvine`. Its return type is `void`, so using the return as a string does not compile. `fail("missing")` compiles as a call that never continues. A function typed as `never` that can return does not compile.

Why: `void` means "do not use this return." `never` means "there is no return."

**Case: `unknown` after a check**

```typescript
function lengthOf(value: unknown): number {
  if (typeof value === "string") {
    return value.length;
  }
  return 0;
}
console.log(lengthOf("Irvine"));
```

Result: compiles. Prints `6`.

Why: inside the `if`, `value` is a `string`. Outside that check, `value.length` does not compile.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `beds?: number` when you meant "the key exists and the value is null" | `{}` is allowed | `beds: number \| null` |
| `string` when the DOM can return null | A later `.value` is unchecked | `string \| null`, then a check |
| Treating `void` as `undefined` you can pass around on purpose | Callers get a type they should ignore | Return `undefined` only if the caller uses it |
| `any` because `unknown` was annoying | No further checks | Narrow `unknown` |

### Tip

Write `| null` when the key is there and empty. Write `?` when the key may be left out.

### Practice

Type a variable `beds` that is always present and may be `null`. Assign `null`, then assign `3`.

### Solution

```typescript
let beds: number | null = null;
beds = 3;
```

Result: both assignments compile. `beds = "3"` does not.

## 3. Arrays, tuples, and objects

### Concept

`number[]` is a list of numbers of any length. A tuple `[string, number]` fixes the type at each index. An object type lists the properties. `readonly` on a property rejects assignment. Excess property checks apply when an object literal is written directly into a typed slot. A variable that already exists may have extra fields when you pass it.

### Terms

| Term | Meaning |
| --- | --- |
| Array type | `number[]` or `Array<number>` |
| Tuple | A fixed-length, fixed-position type |
| Object type | `{ city: string }` |
| Excess property | A field the target type does not name |
| Index signature | `{ [key: string]: number }` |

### Methods

| Syntax | Meaning |
| --- | --- |
| `number[]` | A list of numbers |
| `[string, number]` | Index 0 is a string, index 1 is a number |
| `readonly id: string` | The checker rejects `card.id = "other"` |
| `beds?: number` | The property may be omitted |
| `{ [key: string]: number }` | String keys, number values |

### Examples

**Case: an array rejects a string item**

```typescript
const prices: number[] = [999000, 1100000];
prices.push("900000");
```

Result: the array compiles. `push` of a string does not compile.

Why: every element has to be a number. The length is not fixed.

**Case: a tuple fixes the positions**

```typescript
const pair: [string, number] = ["Irvine", 999000];
const swapped: [string, number] = [999000, "Irvine"];
```

Result: `pair` compiles. `swapped` does not. Index 0 is a string.

Why: a tuple is not "an array of string or number." Position matters.

**Case: `readonly` and an optional property**

```typescript
type Card = {
  readonly id: string;
  city: string;
  price: number;
  beds?: number;
};
const card: Card = { id: "R1", city: "Irvine", price: 999000 };
card.city = "Tustin";
card.id = "R2";
```

Result: the object and `card.city = "Tustin"` compile. `card.id = "R2"` does not compile.

Why: `readonly` is a checker rule. It is erased, so plain JavaScript could still assign `id`. Optional `beds` may be omitted from the literal.

**Case: excess properties**

```typescript
type Card = { city: string };
const direct: Card = { city: "Irvine", price: 1 };
const source = { city: "Irvine", price: 1 };
const aliased: Card = source;
```

Result: `direct` does not compile. `aliased` compiles.

Why: a fresh literal is checked for extra fields, which catches typos. A value that already has a wider type may be passed. The extra field is still there at run time.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `[string, number]` written as `(string \| number)[]` | Either type is allowed at either index | Use a tuple |
| Assigning a `readonly` field | Checker error | Change the variable, or build a new object |
| Adding a field "just on this literal" | Excess property error | Put the field on the type, or pass a variable |
| `readonly` expected to freeze the object at run time | JavaScript can still assign it | `Object.freeze` if you need a runtime freeze |

### Tip

Use a tuple for a pair whose positions mean different things. Use an array when every item is the same kind of value.

### Practice

Type `pair` as city then price, with the values `"Tustin"` and `400`.

### Solution

```typescript
const pair: [string, number] = ["Tustin", 400];
```

Result: compiles. `["Tustin", "400"]` does not.

## 4. Unions, literals, narrowing, and enums

### Concept

A union is one type or another. A string union is the usual way to name a fixed set of strings. An `enum` is a real object at run time. Prefer the union. Narrowing is a check that makes the type smaller inside a branch. A discriminated union is an object union that shares a literal field. `as` tells the checker to trust you and does not convert the value. `as const` keeps the literal type.

### Terms

| Term | Meaning |
| --- | --- |
| Union | `string \| string[]` |
| Literal type | `"approved"` rather than `string` |
| Narrowing | A check the checker understands |
| Discriminant | The shared field, such as `ok: true` |
| `enum` | A runtime object. A numeric enum also has a reverse map |
| `as const` | Freeze the inferred literal |

### Methods

| Check | Narrows to |
| --- | --- |
| `typeof value === "string"` | `string` |
| `value === null` | Removes `null` |
| `"city" in value` | A type that has `city` |
| `value instanceof Listing` | That class |
| `Array.isArray(value)` | An array |
| `result.ok` on a discriminant | The matching branch |

### Examples

**Case: a string union rejects an unknown status**

```typescript
type Status = "pending" | "approved" | "rejected";
function label(status: Status): string {
  return status;
}
label("approved");
label("Approved");
```

Result: `label("approved")` compiles. `label("Approved")` does not compile. Nothing is emitted for the type `Status`. The values are ordinary strings.

Why: the union is erased. The checker still compares the argument to the three literals.

**Case: an enum exists at run time**

```typescript
enum StatusEnum {
  Pending = "pending",
  Approved = "approved",
  Rejected = "rejected",
}
console.log(StatusEnum.Approved);
```

Result: prints `approved`. `StatusEnum` is an object in the JavaScript.

Why: an enum is not erased the way a string union is. A numeric enum such as `enum Code { Ok = 200 }` also maps `200` back to `"Ok"`. Prefer the union unless you are matching code that already has an enum.

**Case: narrowing a union**

```typescript
function lengthOf(value: string | string[]): number {
  if (typeof value === "string") {
    return value.length;
  }
  return value.length;
}
console.log(lengthOf("Irvine"));
console.log(lengthOf(["Irvine", "Tustin"]));
```

Result: compiles. Prints `6`, then `2`.

Why: inside the `if`, `value` is a string, so `length` is the character count. In the other branch it is an array, so `length` is the item count. Both branches type-check for different reasons.

**Case: a discriminant and `as`**

```typescript
type Result =
  | { ok: true; cards: string[] }
  | { ok: false; message: string };

function text(result: Result): string {
  if (result.ok) return result.cards.join(", ");
  return result.message;
}
console.log(text({ ok: true, cards: ["Irvine"] }));
console.log(text({ ok: false, message: "missing" }));
```

Result: prints `Irvine`, then `missing`. `result.message` inside the `ok` branch does not compile. `result.cards` in the other branch does not compile.

```typescript
const value = JSON.parse("12") as string;
```

Result: this assertion compiles. It does not make the number into a string. The value is still `12`.

Why: `as` changes the checker's belief. `as const` is different: `const status = "approved" as const` keeps the type `"approved"` instead of widening it to `string`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| An enum when you wanted a set of strings | A runtime object you did not need | A string union |
| `as string` to silence a union error | The checker stops helping | Narrow with `typeof` or a discriminant |
| Forgetting a union member in a switch | The function may not compile if the return is required in every path | Handle every literal, or return in `default` |
| `"Approved"` versus `"approved"` | The checker rejects the capital one | Match the literal exactly |

### Tip

If two object shapes share a field that is `true` in one and `false` in the other, switch on that field. The other properties appear only in the matching branch.

### Practice

Write `text` for the `Result` union above and say what `text({ ok: false, message: "missing" })` returns.

### Solution

The function in the discriminant example.

Result: `"missing"`. The `cards` field is not available in that branch.

## 5. Functions

### Concept

A function type names the parameters and the return. `void` means callers should not use the return value. An optional parameter comes after the required ones. A default parameter's type is inferred from the default. Rest collects a typed array. Overloads are several call signatures followed by one implementation. The implementation signature is hidden from callers. A `this` parameter types `this` and is not a real argument.

### Terms

| Term | Meaning |
| --- | --- |
| Call signature | `(n: number) => number` |
| Optional | `unit?: string` |
| Overload | Public signatures the implementation must satisfy |
| `this` parameter | `function f(this: Card)`. Not passed at the call |

### Methods

| Form | Meaning |
| --- | --- |
| `(n: number) => number` | A function type |
| `void` | Ignore the return |
| `...nums: number[]` | Rest |
| Several signatures, then one body | Overloads |
| `function f(this: Card)` | Types `this` only |

### Examples

**Case: the return type is checked**

```typescript
function clamp(n: number, min = 0, max = 100): number {
  return Math.min(max, Math.max(min, n));
}
console.log(clamp(150));
console.log(clamp(-4, 0, 10));
```

Result: compiles. Prints `100`, then `0`. `min` and `max` are numbers because the defaults are numbers.

Why: the checker infers the defaulted parameters. The return must be a number. Returning a string would not compile.

**Case: an optional parameter**

```typescript
function label(city: string, note?: string): string {
  return note ? `${city} (${note})` : city;
}
console.log(label("Irvine"));
console.log(label("Irvine", "condo"));
```

Result: prints `Irvine`, then `Irvine (condo)`.

Why: `note` is `string | undefined`. A required parameter cannot follow an optional one.

**Case: overloads hide the implementation**

```typescript
type Card = { id: string; city: string; price: number };
function get(id: string): Card;
function get(id: string, optional: true): Card | undefined;
function get(id: string, optional?: boolean): Card | undefined {
  return optional ? undefined : { id, city: "Irvine", price: 1 };
}
const found = get("R1");
const maybe = get("R1", true);
```

Result: `found` is `Card`. `maybe` is `Card | undefined`. A caller cannot use the implementation's optional boolean except through the two public signatures.

Why: the signatures above the implementation are what callers see. `get("R1")` is not allowed to be `undefined` in the type.

**Case: a `this` parameter is not an argument**

```typescript
type Card = { city: string };
function label(this: Card): string {
  return this.city;
}
const card = { city: "Irvine" };
console.log(label.call(card));
```

Result: prints `Irvine`. `label(card)` does not compile, because `this` is not a real parameter. `label()` does not compile, because `this` would be the wrong type.

Why: the `this` parameter types the receiver. `call` supplies it.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A required parameter after an optional one | Does not compile | Reorder them |
| Calling an overload with a mix the signatures do not list | Does not compile, even if the implementation could handle it | Add a signature or change the call |
| Using a `void` return as data | Does not compile | Change the return type if the caller needs the value |
| Passing `this` as the first argument | Does not compile | `fn.call(receiver)` |

### Tip

Write the return type on a function you export. Inference is enough for a local helper. The annotation is the contract.

### Practice

Write `clamp(150)` with defaults `0` and `100`, and state the result.

### Solution

The `clamp` function in the first example.

Result: `100`.

## 6. Interfaces, type aliases, and classes

### Concept

`type` and `interface` can both name an object. Only `type` can name a union. An interface can be reopened: two interfaces with the same name merge. A class `implements` an interface. `public` on a constructor parameter declares a field and assigns it. `private` and `protected` are checker-only. `#id` is private at run time. `readonly` blocks assignment after construction.

### Terms

| Term | Meaning |
| --- | --- |
| `type` | An alias. Unions, tuples, objects |
| `interface` | An object contract. Can be merged and extended |
| `implements` | The class must have those fields |
| Parameter property | `constructor(public id: string)` |
| `#id` | Runtime privacy |

### Methods

| | `type` | `interface` |
| --- | --- | --- |
| Object shape | Yes | Yes |
| Union | Yes | No |
| Extension | `&` | `extends` |
| Declaration merging | No | Yes |
| Prefer | Unions, tuples, mapped types | Object contracts other types extend |

### Examples

**Case: a union must be a type**

```typescript
type Money = number;
interface Identified { id: string }
interface Card extends Identified {
  city: string;
  price: number;
}
type IdOrCard = string | Card;
```

Result: these declarations compile. `interface IdOrCard = string | Card` does not compile, because an interface cannot be a union.

Why: use `type` for a union. Use `interface` when other interfaces will `extend` the object.

**Case: interfaces merge**

```typescript
interface Card { city: string }
interface Card { price: number }
const card: Card = { city: "Irvine", price: 1 };
```

Result: compiles. `Card` has both fields.

Why: two interfaces with the same name combine. Two types with the same name do not compile.

**Case: a class implements the interface**

```typescript
interface Identified { id: string }
class MemoryCard implements Identified {
  constructor(public id: string, public city: string) {}
}
const card = new MemoryCard("R1", "Irvine");
console.log(card.city);
```

Result: prints `Irvine`. Dropping `id` from the constructor does not compile, because `implements Identified` requires `id`.

Why: `public` on the parameter creates the property. The checker checks the class against the interface.

**Case: `#id` versus `private`**

```typescript
class Listing {
  #id: string;
  private city: string;
  constructor(id: string, city: string) {
    this.#id = id;
    this.city = city;
  }
}
```

Result: the class compiles. From outside the class, both `listing.city` and `listing.#id` do not compile. At run time, a JavaScript caller can still read a `private` field if the privacy was only in the type. `#id` throws if outside code reaches it.

Why: `private` is erased. `#id` is a real JavaScript private field.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| An interface for a union | Does not compile | `type` |
| `private` treated as a security boundary | The field is still on the object at run time | `#id` when runtime privacy matters |
| Forgetting `implements` fields | Does not compile | Add the field |
| Two `type` aliases with one name | Does not compile | One alias, or an interface if you meant to merge |

### Tip

Object that other types extend: `interface`. Union, tuple, or utility: `type`.

### Practice

Declare `Card` with `id` and `city` using an interface that extends `{ id: string }`, and build a value.

### Solution

```typescript
interface Identified { id: string }
interface Card extends Identified { city: string }
const card: Card = { id: "R1", city: "Tustin" };
```

Result: compiles. Omitting `city` does not.

## 7. Generics

### Concept

A generic is a type parameter. The caller or inference fills it in. `T extends string` constrains `T`. `keyof T` is the union of keys. `T[K]` is the type of that property. A default, `T = unknown`, is used when the caller does not specify a type.

### Terms

| Term | Meaning |
| --- | --- |
| Type parameter | `<T>` |
| Constraint | `T extends string` |
| `keyof T` | The keys |
| Indexed access | `T[K]` |
| Default | `T = unknown` |

### Methods

| Syntax | Meaning |
| --- | --- |
| `function first<T>(items: readonly T[]): T \| undefined` | The item type is inferred from the array |
| `T extends keyof Card` | `T` must be one of the keys |
| `type Box<T> = { value: T }` | A type with a parameter |
| `type ApiResponse<T = unknown> = { data: T }` | `unknown` when `T` is omitted |

### Examples

**Case: inference from the array**

```typescript
function first<T>(items: readonly T[]): T | undefined {
  return items[0];
}
const n = first([1, 2, 3]);
const missing = first([]);
```

Result: `n` is `number | undefined`. `missing` is `unknown[]`'s element or, for `[]` with no context, an error or `undefined` depending on inference. `first([])` with no context does not compile under `strict`, because `T` cannot be inferred. `first<number>([])` compiles and the result is `undefined`.

Why: `T` comes from the elements. An empty array has no element to read.

**Case: `keyof` and indexed access**

```typescript
type Card = { city: string; price: number };
function pluck<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
console.log(pluck({ city: "Irvine", price: 1 }, "city"));
```

Result: compiles and prints `Irvine`. The return type is `string`. `pluck(card, "beds")` does not compile.

Why: `K` has to be a key of `T`. The return type follows that key, so `"city"` returns `string` and `"price"` returns `number`.

**Case: a constraint**

```typescript
function shout<T extends string>(value: T): string {
  return value.toUpperCase();
}
console.log(shout("irvine"));
```

Result: prints `IRVINE`. `shout(3)` does not compile.

Why: `T extends string` lets the body call `toUpperCase`. A bare `<T>` would not.

**Case: a default type parameter**

```typescript
type ApiResponse<T = unknown> = { data: T };
const loose: ApiResponse = { data: "Irvine" };
const typed: ApiResponse<string> = { data: "Irvine" };
```

Result: both compile. `loose.data` is `unknown`. `typed.data` is `string`.

Why: the default applies when you write `ApiResponse` with no argument. It does not check that `data` is a particular type until you pass one.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `first([])` with no type | `T` is not inferred | `first<number>([])` |
| Returning `T` from an empty list as if it is always present | The type is `T \| undefined` | Handle `undefined` |
| `obj[key]` when `key` is `string` | The result is not a specific property | `K extends keyof T` |
| A constraint so wide it is `unknown` | The body cannot call string methods | `T extends string` |

### Tip

If a function would otherwise need `any` to work for both strings and numbers, it wants a type parameter.

### Practice

Write `first` so `first(["Irvine"])` is `string | undefined`.

### Solution

```typescript
function first<T>(items: readonly T[]): T | undefined {
  return items[0];
}
const city = first(["Irvine"]);
```

Result: `city` is `string | undefined`. The value is `"Irvine"`.

## 8. Utility types

### Concept

Utility types build a type from another type. `Partial` makes every property optional. `Pick` keeps keys. `Omit` drops keys. They do not change the types of the keys that remain. `Partial` is how an update object can carry only the fields that changed.

### Terms

| Utility | Result |
| --- | --- |
| `Partial<T>` | Every property optional |
| `Required<T>` | Every property required |
| `Readonly<T>` | Every property readonly |
| `Pick<T, K>` | Only those keys |
| `Omit<T, K>` | All keys except those |
| `Record<K, V>` | Keys `K`, values `V` |
| `Exclude` / `Extract` | Remove or keep union members |
| `NonNullable<T>` | Drops `null` and `undefined` |
| `ReturnType<F>` / `Parameters<F>` | The return and the parameter tuple |

### Methods

Start from `type Card = { id: string; city: string; price: number; beds: number }`.

| Start | Resulting shape |
| --- | --- |
| `Partial<Card>` | Every field optional |
| `Pick<Card, "id" \| "city">` | `{ id: string; city: string }` |
| `Omit<Card, "beds">` | The other three fields |
| `Record<"irvine" \| "tustin", Card>` | Those two keys, each a `Card` |

### Examples

**Case: `Partial` allows a one-field update**

```typescript
type Card = { id: string; city: string; price: number; beds: number };
type Draft = Partial<Card>;
const update: Draft = { city: "Tustin" };
const full: Card = { city: "Tustin" };
```

Result: `update` compiles. `full` does not compile, because `id`, `price`, and `beds` are missing.

Why: `Partial` adds `?` to every field. It does not make the fields nullable.

**Case: `Pick` and `Omit`**

```typescript
type Preview = Pick<Card, "id" | "city">;
type NoBeds = Omit<Card, "beds">;
const preview: Preview = { id: "R1", city: "Irvine" };
const rest: NoBeds = { id: "R1", city: "Irvine", price: 1 };
```

Result: both compile. Adding `beds` to the `Preview` literal does not compile. Omitting `price` from `NoBeds` does not compile.

Why: `Pick` is a keep-list. `Omit` is a drop-list. The remaining fields keep their original types.

**Case: `Readonly` and `Record`**

```typescript
function lock(card: Readonly<Card>) {
  card.city = "Tustin";
}
type ByCity = Record<"irvine" | "tustin", string>;
const labels: ByCity = { irvine: "Irvine", tustin: "Tustin" };
```

Result: the assignment inside `lock` does not compile. `labels` compiles. Leaving out `tustin` does not.

Why: `Readonly` rejects assignment. `Record` requires every key in the union.

**Case: `NonNullable` and `ReturnType`**

```typescript
type Maybe = string | null;
type Text = NonNullable<Maybe>;
function priceLabel(cents: number): string {
  return `$${cents}`;
}
type Label = ReturnType<typeof priceLabel>;
```

Result: `Text` is `string`. `Label` is `string`. A value of type `Text` cannot be `null`.

Why: these utilities read a type you already wrote. They do not change the runtime value.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `Partial` expected to allow `null` | `null` is still rejected | `Partial<Card>` plus `\| null` on the fields that need it |
| `Pick` and `Omit` swapped | You keep the fields you meant to drop | Keep-list versus drop-list |
| `Readonly` expected at run time | The object can still be mutated by JavaScript | A checker-only constraint |
| `Record<string, Card>` when only two keys are legal | Any string key is accepted | A union of the literal keys |

### Tip

`Partial` for a patch, `Pick` for a preview, `Omit` for "everything except this."

### Practice

Given `Card` from this section, type an update that may contain only `city`.

### Solution

```typescript
type Card = { id: string; city: string; price: number; beds: number };
const update: Partial<Card> = { city: "Tustin" };
```

Result: compiles. The same literal assigned to `Card` does not.

## 9. Modules and configuration

### Concept

Imports and exports are the JavaScript ones. `import type` marks an import the bundler can drop because it is types only. `strict` in `tsconfig.json` turns on `strictNullChecks` and `noImplicitAny` together. With `strictNullChecks`, `document.querySelector` returns `Element | null`. You handle `null` before you read a property. `noEmit` checks without writing files.

### Terms

| Term | Meaning |
| --- | --- |
| Named export | `export function emptyCard` |
| `import type` | Types only. Erased |
| `strict` | The important flags together |
| `noEmit` | Check only |
| `target` | The JavaScript you emit, such as `ES2022` |

### Methods

| Option | A strict course setting |
| --- | --- |
| `strict` | `true` |
| `strictNullChecks` | `null` is not a `string`. Included in `strict` |
| `noImplicitAny` | A missing type does not become `any`. Included in `strict` |
| `module` | How imports are emitted |
| `noEmit` | Check only |

### Examples

**Case: a type import is erased**

```typescript
export type Card = { id: string; city: string };
export function emptyCard(id: string): Card {
  return { id, city: "" };
}
```

```typescript
import { emptyCard, type Card } from "./cards";
const card: Card = emptyCard("R1");
console.log(card.city);
```

Result: prints an empty string. The JavaScript import keeps `emptyCard` and drops `Card`.

Why: `import type` tells the checker and the bundler that `Card` is not a runtime value.

**Case: a missing annotation becomes an error**

```typescript
function label(city) {
  return city;
}
```

Result: does not compile when `noImplicitAny` is on. The parameter is an implicit `any`.

Why: `strict` includes this flag. Write `city: string`.

**Case: `querySelector` includes null**

```typescript
const form = document.querySelector("form");
form.addEventListener("submit", () => {});
```

Result: does not compile. `form` is `Element | null`.

```typescript
const form = document.querySelector("form");
if (!form) throw new Error("form missing");
form.addEventListener("submit", () => {});
```

Result: compiles. After the throw, `form` is an `Element`.

Why: `strictNullChecks` puts `null` in the union. The `if` narrows it.

**Case: `noEmit` writes nothing**

A `tsconfig.json` with `"noEmit": true` and `"strict": true`.

Result: `tsc` prints errors and does not create `.js` files.

Why: a bundler or a runner such as `tsx` executes the TypeScript. The checker only reports.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Importing a type as a value | A runtime import of something that was erased | `import type` |
| Using `querySelector` without a null check | Does not compile under `strict` | Narrow, or throw |
| Turning `strict` off to finish a file | `null` and `any` slip through | Leave `strict` on and fix the lines |
| Expecting `target` to change the checker rules | It changes the emitted JavaScript syntax | `strict` is the checker. `target` is the output |

### Tip

Turn `strict` on at the start. Adding it later means every `null` in the project shows up at once.

### Practice

Import the type `Card` and the function `emptyCard` from `./cards` so the type is erased and the function remains. Print the city of `emptyCard("R1")`.

### Solution

```typescript
import { emptyCard, type Card } from "./cards";
const card: Card = emptyCard("R1");
console.log(card.city);
```

Result: prints an empty string. The emitted import does not include `Card`.

## Drill

1. Write a `Card` type with `id: string`, `price: number`, and `note: string | null`.
2. Write `first` as a generic that returns `T | undefined`.
3. Say what `Partial<Card>` allows that `Card` does not.
4. Say why `document.querySelector("form")` cannot be used immediately under `strict`.

### Solution 1

```typescript
type Card = { id: string; price: number; note: string | null };
```

`note` is always present and may be `null`.

### Solution 2

```typescript
function first<T>(items: readonly T[]): T | undefined {
  return items[0];
}
```

### Solution 3

`Partial<Card>` makes every field optional. A value can contain only `city`. `Card` requires every field.

### Solution 4

The result is `Element | null`. Reading a property before a null check does not compile.
