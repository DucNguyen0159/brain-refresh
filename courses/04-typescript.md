# TypeScript

Course 4 of 15. Previous: [03-javascript.md](03-javascript.md). Next: [05-react.md](05-react.md).

TypeScript is JavaScript plus a checker. You write types. The checker reports mistakes before the program runs. The types are erased when the program runs, unless a tool is configured to keep them. At run time you still have ordinary JavaScript values.

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

```typescript
function priceLabel(cents: number): string {
  return `$${(cents / 100).toFixed(2)}`;
}
priceLabel(999000);
```

| Idea | Meaning |
| --- | --- |
| Annotation | `: number` on a parameter, `: string` on the return |
| Inference | If you write `const city = "Irvine"`, the checker already knows it is a string |
| Error | A message from the checker. The JavaScript might still run if you ignore it |
| Erasure | `number` is not present when Node or the browser runs the file |

`any` turns the checker off for that value. `unknown` is the honest version of "I do not know yet": you must narrow it before you use it.

## 2. Basic types

| Type | Allows |
| --- | --- |
| `string` | Text |
| `number` | Numbers, including `NaN` |
| `boolean` | `true` and `false` |
| `null` | Only `null` |
| `undefined` | Only `undefined` |
| `bigint` | Big integers |
| `symbol` | Symbols |
| `void` | A function that returns nothing useful |
| `never` | A function that never returns, such as one that always throws |
| `object` | A non-primitive. Prefer a more specific object type |
| `any` | Anything. Avoid |
| `unknown` | Anything, until you check |

`any`, `unknown`, and `never` all sit outside the ordinary types. They do not behave alike.

| | `any` | `unknown` | `never` |
| --- | --- | --- | --- |
| You can assign any value to it | Yes | Yes | No. Nothing is a `never` except a function that does not return |
| You can use it as a `string` with no check | Yes. The checker steps aside | No | A `never` value does not occur, so the line is unreachable |
| After a check | Still `any` | Narrows to the type you proved | Already impossible |
| A function return of this type | "I gave up" | "Caller must inspect me" | "I throw or loop forever" |
| Where it shows up | An untyped library, or a forgotten annotation | `JSON.parse`, a form value, a caught error | The `default` of a switch that handled every case |

```typescript
let city: string = "Irvine";
let beds: number | null = null;
```

`string | null` means one or the other. That is a union. Your filter objects use it so a missing filter is `null` rather than a missing key.

Optional property is different:

| Form | The key |
| --- | --- |
| `beds: number \| null` | Always present. Value may be null |
| `beds?: number` | May be absent. When present, a number |
| `beds?: number \| null` | May be absent, null, or a number |

## 3. Arrays, tuples, and objects

```typescript
const prices: number[] = [999000, 1100000];
const pair: [string, number] = ["Irvine", 999000];

type Card = {
  readonly id: string;
  city: string;
  price: number;
  beds?: number;
};

const card: Card = { id: "R1", city: "Irvine", price: 999000 };
```

| Syntax | Meaning |
| --- | --- |
| `number[]` or `Array<number>` | A list of numbers |
| `[string, number]` | A tuple. Index 0 is a string, index 1 is a number, length is fixed in the type |
| `{ city: string }` | An object type |
| `readonly id: string` | The checker rejects `card.id = "other"` |
| `?` | The property may be omitted |
| Index signature | `{ [key: string]: number }` means string keys to numbers |

A tuple is the typed version of a fixed pair, such as `[RegExp, string]`.

Excess property checks apply when you write an object literal directly into a typed slot. A variable that already exists is allowed to have extra fields when you pass it.

## 4. Unions, literals, narrowing, and enums

```typescript
type Status = "pending" | "approved" | "rejected";

function label(status: Status): string {
  switch (status) {
    case "pending":
      return "Waiting";
    case "approved":
      return "Approved";
    case "rejected":
      return "Rejected";
  }
}

function lengthOf(value: string | string[]): number {
  if (typeof value === "string") {
    return value.length;
  }
  return value.length;
}
```

| Check | Narrows to |
| --- | --- |
| `typeof value === "string"` | string |
| `typeof value === "number"` | number |
| `value === null` | removes null from a union |
| `"city" in value` | a type that has `city` |
| `value instanceof Listing` | that class |
| `Array.isArray(value)` | an array |
| Equality with a literal | that literal |
| A discriminated union | the branch whose tag matches |

A string union is the usual way to name a fixed set of strings. An `enum` is the older object form. You will read both.

```typescript
type Status = "pending" | "approved" | "rejected";

enum StatusEnum {
  Pending = "pending",
  Approved = "approved",
  Rejected = "rejected",
}
```

| | String union | `enum` |
| --- | --- | --- |
| At run time | Nothing. The type is erased. The values are ordinary strings | A real object. `StatusEnum.Approved` is `"approved"` |
| What you write in a function | `status: Status` | `status: StatusEnum` |
| Usual choice | Yes, for a set of strings | When the code already uses one, or you need the object |
| Numeric form | A union of numbers, if you want that | `enum Code { Ok = 200 }` also creates a reverse map from `200` back to `"Ok"` |
| A string that is not in the set | The checker rejects it | The checker rejects it when the parameter is the enum type |

Prefer the union. Reach for an `enum` when you are matching code that already has one.

A discriminated union is an object union with a shared literal field:

```typescript
type Result =
  | { ok: true; cards: string[] }
  | { ok: false; message: string };

function text(result: Result): string {
  if (result.ok) return result.cards.join(", ");
  return result.message;
}
```

After `result.ok` is true, `message` is not on the type. After it is false, `cards` is not.

`as` asserts a type. `value as string` tells the checker to trust you. It does not convert the value. Use it at a boundary you have already checked, not to silence an error you do not understand.

`as const` freezes the literal:

```typescript
const status = "approved" as const;
const cities = ["Irvine", "Tustin"] as const;
```

## 5. Functions

```typescript
function clamp(n: number, min = 0, max = 100): number {
  return Math.min(max, Math.max(min, n));
}

function first<T>(items: T[]): T | undefined {
  return items[0];
}

type Click = (id: string) => void;
```

| Form | Meaning |
| --- | --- |
| `(n: number) => number` | A function type |
| `void` return | Callers should not use the return value |
| Optional parameter | `unit?: string`. Must come after required ones |
| Default parameter | The type is inferred from the default |
| Rest | `...nums: number[]` |
| Overload | Several call signatures, then one implementation |
| `this` parameter | `function f(this: Card)` types `this`. It is not a real argument |

An overload:

```typescript
function get(id: string): Card;
function get(id: string, optional: true): Card | undefined;
function get(id: string, optional?: boolean): Card | undefined {
  return optional ? undefined : { id, city: "Irvine", price: 1 };
}
```

The implementation signature is hidden. Callers see the overloads above it.

## 6. Interfaces, type aliases, and classes

```typescript
interface Identified {
  id: string;
}

interface Card extends Identified {
  city: string;
  price: number;
}

type Money = number;
type IdOrCard = string | Card;
```

| | `type` | `interface` |
| --- | --- | --- |
| Object shape | Yes | Yes |
| Union | Yes | No |
| `extends` / intersection | `&` | `extends` |
| Declaration merging | No | Yes. Two interfaces with the same name combine |
| Prefer | Unions, tuples, mapped types | Object contracts that other types will extend |

A class implements an interface:

```typescript
class MemoryCard implements Identified {
  constructor(public id: string, public city: string) {}
}
```

`public` on a constructor parameter declares the field and assigns it. `private` and `protected` are checker-only names. `#id` is private at run time. `readonly` blocks assignment after construction.

Enums exist. A string union is easier to read and tree-shakes cleanly. Use a union unless you are matching an existing enum.

```typescript
type Role = "admin" | "artist" | "employee";
```

## 7. Generics

A generic is a type parameter. The caller, or inference, fills it in.

```typescript
function first<T>(items: readonly T[]): T | undefined {
  return items[0];
}

const n = first([1, 2, 3]); // number | undefined

type Box<T> = { value: T };

function pluck<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

| Syntax | Meaning |
| --- | --- |
| `<T>` | A type parameter |
| `T extends string` | A constraint. `T` must be assignable to `string` |
| `keyof T` | The union of the keys of `T` |
| `T[K]` | The type of that property |
| `readonly T[]` | The function will not mutate the array type |

Defaults: `type ApiResponse<T = unknown> = { data: T }`.

## 8. Utility types

These build a type from another type.

```typescript
type Card = { id: string; city: string; price: number; beds: number };

type Draft = Partial<Card>;
type Preview = Pick<Card, "id" | "city">;
type NoBeds = Omit<Card, "beds">;
type ById = Record<string, Card>;
```

| Utility | Result |
| --- | --- |
| `Partial<T>` | Every property optional |
| `Required<T>` | Every property required |
| `Readonly<T>` | Every property readonly |
| `Pick<T, K>` | Only those keys |
| `Omit<T, K>` | All keys except those |
| `Record<K, V>` | An object type with keys `K` and values `V` |
| `Exclude<U, E>` | Union `U` without members assignable to `E` |
| `Extract<U, E>` | Union `U` keeping members assignable to `E` |
| `NonNullable<T>` | `T` without `null` and `undefined` |
| `ReturnType<F>` | What that function type returns |
| `Parameters<F>` | The parameter tuple |

The same `Card` under each utility:

| Start from `Card` | Resulting shape |
| --- | --- |
| `Partial<Card>` | `{ id?: string; city?: string; price?: number; beds?: number }` |
| `Required<Card>` | Every field required, even if `Card` had made one optional |
| `Readonly<Card>` | The same fields, and assigning `card.city = "x"` is an error |
| `Pick<Card, "id" \| "city">` | `{ id: string; city: string }` |
| `Omit<Card, "beds">` | `{ id: string; city: string; price: number }` |
| `Record<"irvine" \| "tustin", Card>` | An object whose keys are those two names and whose values are `Card` |

`Partial` is how an update object can carry only the fields that changed. `Pick` keeps a few fields. `Omit` drops a few. They are opposites on the key list, and neither changes the types of the keys that remain.

## 9. Modules and configuration

Imports and exports are the JavaScript ones. Types can be imported too.

```typescript
export type Card = { id: string; city: string };
export function emptyCard(id: string): Card {
  return { id, city: "" };
}
```

```typescript
import { emptyCard, type Card } from "./cards";
```

`import type` tells the checker the import is types only, so a bundler can drop it.

A `tsconfig.json` turns the checker on for a folder.

| Option | A strict course setting |
| --- | --- |
| `strict` | `true`. Turns on the important flags together |
| `strictNullChecks` | `null` is not a `string`. Included in `strict` |
| `noImplicitAny` | A missing type does not become `any`. Included in `strict` |
| `target` | The JavaScript you emit, such as `ES2022` |
| `module` | How imports are emitted |
| `noEmit` | Check only. Node or a bundler runs the TypeScript itself |

With `strictNullChecks`, `document.querySelector` returns `Element | null`. You must handle `null` before you read a property.

```typescript
const form = document.querySelector("form");
if (!form) throw new Error("form missing");
form.addEventListener("submit", () => {});
```

## Drill

1. Write a `Card` type with `id: string`, `price: number`, and `note: string | null`.
2. Write `first` as a generic.
3. Say what `Partial<Card>` allows that `Card` does not.
