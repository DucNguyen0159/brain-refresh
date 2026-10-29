# Vitest

Course 14 of 15. Previous: [13-tanstack-query.md](13-tanstack-query.md). Next: [15-java.md](15-java.md).

Vitest runs tests. A test calls a function and checks the result. The file is usually `something.test.ts`. The command is `npx vitest` or the `test` script in `package.json`. `vitest run` executes once and exits. `vitest` without `run` watches.

## Contents

1. [A test file](#1-a-test-file)
2. [Matchers](#2-matchers)
3. [Async tests and time](#3-async-tests-and-time)
4. [Setup and doubles](#4-setup-and-doubles)
5. [What to test](#5-what-to-test)

## 1. A test file

```typescript
import { describe, expect, it } from "vitest";

function add(a: number, b: number): number {
  return a + b;
}

describe("add", () => {
  it("adds two numbers", () => {
    expect(add(1, 2)).toBe(3);
  });

  it("adds zero", () => {
    expect(add(1, 0)).toBe(1);
  });
});
```

| Word | Meaning |
| --- | --- |
| `describe` | A group. The string is the group name |
| `it` or `test` | One case. They are the same function |
| `expect` | The value under test |
| Matcher | The check, such as `toBe` |
| Failure | The process exits non-zero and prints the expected and actual values |

Nested `describe` blocks group a function's success and failure cases. The full name in the output is the chain of strings.

`it.skip` leaves a test in the file without running it. `it.only` runs one test and hides the rest. Remove `only` before you trust the suite. `it.todo` records a test you have not written.

## 2. Matchers

| Matcher | Passes when |
| --- | --- |
| `toBe` | Same value, `Object.is`. Use it for numbers, strings, and booleans |
| `toEqual` | Deep equality. Use it for objects and arrays |
| `toStrictEqual` | Deep equality, and no extra `undefined` keys, and matching types |
| `toBeCloseTo` | Numbers that may have float error |
| `toBeNull` / `toBeUndefined` | Those exact values |
| `toBeDefined` | Not `undefined` |
| `toBeTruthy` / `toBeFalsy` | JavaScript truthiness. Prefer a precise matcher |
| `toBeGreaterThan` / `toBeLessThan` | Order |
| `toContain` | An array contains the item, or a string contains the substring |
| `toHaveLength` | `.length` equals the number |
| `toMatch` | A string matches a regex |
| `toMatchObject` | The object contains at least these fields |
| `toThrow` | A function throws. Pass a string, regex, or error class |
| `rejects.toThrow` | A promise rejects |
| `resolves.toBe` | A promise fulfills with that value |

```typescript
expect({ city: "Irvine", price: 1 }).toEqual({ city: "Irvine", price: 1 });
expect(() => {
  throw new Error("bad price");
}).toThrow(/price/);
```

`toBe` on two objects fails when they are equal in contents and different in identity. That surprise is why objects use `toEqual`.

| You are comparing | Matcher |
| --- | --- |
| A number, string, or boolean | `toBe` |
| Two objects or arrays, same contents | `toEqual` |
| Two objects, and extra `undefined` keys must count | `toStrictEqual` |
| An object that may have more fields than you care about | `toMatchObject` |
| A float that is not exact | `toBeCloseTo` |
| A string that matches a pattern | `toMatch` |
| An array that should contain one item | `toContain` |
| A thrown error | `toThrow` |
| A rejected promise | `rejects.toThrow` |
| A fulfilled promise | `resolves.toEqual` or `resolves.toBe` |

`expect(value).not.toBe(0)` inverts a matcher.

## 3. Async tests and time

Return the promise, or use `async` and `await`. A test that forgets to await finishes before the check runs, and a later failure can be lost.

```typescript
it("loads", async () => {
  const rows = await loadCities();
  expect(rows).toEqual(["Irvine"]);
});

it("rejects on a bad id", async () => {
  await expect(loadCity("nope")).rejects.toThrow(/not found/);
});
```

Fake timers replace `setTimeout` and `setInterval`.

```typescript
import { afterEach, beforeEach, vi } from "vitest";

beforeEach(() => {
  vi.useFakeTimers();
});

afterEach(() => {
  vi.useRealTimers();
});

it("times out", async () => {
  const pending = new Promise((resolve) => {
    setTimeout(resolve, 10_000);
  });
  const result = Promise.race([
    pending.then(() => "done"),
    new Promise((resolve) => setTimeout(() => resolve("timeout"), 20)),
  ]);
  await vi.advanceTimersByTimeAsync(20);
  await expect(result).resolves.toBe("timeout");
});
```

| Timer API | Effect |
| --- | --- |
| `vi.useFakeTimers()` | Take over timers |
| `vi.advanceTimersByTimeAsync(ms)` | Move the clock and flush the resulting promises |
| `vi.runAllTimersAsync()` | Run every scheduled timer |
| `vi.useRealTimers()` | Give the clock back |

A timeout case should fail because the deadline passed, not because the suite itself hung. Fake timers make that fast.

## 4. Setup and doubles

```typescript
import { afterEach, beforeEach, vi } from "vitest";

beforeEach(() => {
  vi.restoreAllMocks();
});
```

| Hook | Runs |
| --- | --- |
| `beforeEach` | Before every test in this file or describe |
| `afterEach` | After every test |
| `beforeAll` / `afterAll` | Once per file or describe |

A double replaces a collaborator.

| Tool | Use |
| --- | --- |
| `vi.fn()` | A function that records its calls. `expect(fn).toHaveBeenCalledWith("Irvine")` |
| `vi.fn().mockReturnValue(1)` | A synchronous return |
| `vi.fn().mockResolvedValue(rows)` | A promise that fulfills |
| `vi.fn().mockRejectedValue(new Error("x"))` | A promise that rejects |
| `vi.spyOn(obj, "method")` | Wrap a real method. Restore it in `afterEach` |
| `vi.mock("./db")` | Replace a module. Hoisted. The factory returns the exports |

| | `vi.fn()` | `vi.spyOn(obj, "method")` | `vi.mock("./db")` |
| --- | --- | --- | --- |
| What it replaces | A new function you pass into the code | One method. The original is saved | A whole imported module |
| The real code still runs | Only if you set the implementation to it | Yes, until you change the implementation | No, unless the factory calls through |
| How you put it back | `mockReset`, or a new test | `mockRestore` in `afterEach` | `vi.unmock`, or reset modules |
| Use it when | The code takes a callback | You want to watch a method you own | The code imports a module you cannot pass in |

```typescript
const save = vi.fn().mockResolvedValue(undefined);
await save("Irvine");
expect(save).toHaveBeenCalledOnce();
expect(save).toHaveBeenCalledWith("Irvine");
```

Mock only the edge: the network, the clock, the database. Do not mock the function you are trying to test.

`vi.stubEnv("PORT", "4000")` sets an environment variable for the test. `vi.unstubAllEnvs()` clears stubs.

## 5. What to test

Test behavior a caller can see. A good case has an input and one expected outcome.

| Case | Assert |
| --- | --- |
| The happy path | The returned value |
| A rejected input | The error |
| A boundary | Empty list, zero, null |
| A timeout or a late result | The call ends, and it does not clobber a newer call |

Do not assert the number of times a private helper ran. Do not snapshot an entire page unless you are ready to reread the snapshot on every copy change. A snapshot of a small pure value is fine.

Coverage counts which lines ran. It does not count which behaviors you checked. A suite can cover every line and still miss the failure case. Read the uncovered lines, then add a case if that line is a behavior.

| Command | Result |
| --- | --- |
| `vitest run` | One pass |
| `vitest run path/to/file.test.ts` | One file |
| `vitest run -t "adds"` | Tests whose name matches |
| `vitest run --coverage` | A coverage report, after the coverage package is installed |

A test must be able to fail. If you delete the assertion and the test still passes, it is not testing anything.

## Drill

1. Write two cases for a function that returns `null` on an empty string and a number otherwise.
2. Write an async case that expects a rejection.
3. Say why objects use `toEqual`.
