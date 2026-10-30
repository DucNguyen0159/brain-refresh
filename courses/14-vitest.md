# Vitest

Course 14 of 15. Previous: [13-tanstack-query.md](13-tanstack-query.md). Next: [15-java.md](15-java.md).

Vitest runs tests. A test calls a function and checks the result. The file is usually `something.test.ts`. The command is `npx vitest` or the `test` script in `package.json`. `vitest run` executes once and exits. `vitest` without `run` watches.

Every numbered section has the same parts: the concept, the terms, the methods, several cases with the result, common mistakes, a tip, and a practice with a solution.

## Contents

1. [A test file](#1-a-test-file)
2. [Matchers](#2-matchers)
3. [Async tests and time](#3-async-tests-and-time)
4. [Setup and doubles](#4-setup-and-doubles)
5. [What to test](#5-what-to-test)

## 1. A test file

### Concept

A test file groups cases with `describe` and writes each case with `it` or `test`. Those two are the same function. `expect` holds the value under test, and a matcher is the check. A failure makes the process exit non-zero and prints the expected and actual values. Nested `describe` blocks group a function's success and failure cases. The full name in the output is the chain of strings. `it.skip` leaves a test in the file without running it. `it.only` runs one test and hides the rest. Remove `only` before you trust the suite. `it.todo` records a test you have not written.

### Terms

| Word | Meaning |
| --- | --- |
| `describe` | A group. The string is the group name |
| `it` or `test` | One case. They are the same function |
| `expect` | The value under test |
| Matcher | The check, such as `toBe` |
| Failure | The process exits non-zero and prints the expected and actual values |
| `something.test.ts` | The usual file name. Vitest collects it |

### Methods

| Call | Effect |
| --- | --- |
| `describe("add", fn)` | Groups the cases. Nested groups join their strings in the output |
| `it("adds two numbers", fn)` | One case |
| `test("adds two numbers", fn)` | The same function as `it` |
| `expect(value)` | Starts an assertion |
| `it.skip` | Leaves the test in the file and does not run it |
| `it.only` | Runs this test and hides the rest |
| `it.todo("name")` | Records a test you have not written |
| `vitest run` | Executes once and exits |
| `vitest` | Watches |
| `npx vitest` | The same watch command, or use the `test` script in `package.json` |

### Examples

**Case: two cases in one group**

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

Result: the test runner runs both cases and marks them passed. The output names them under `add`.
Why: `describe` is the group. `it` is one case. `expect` plus `toBe` is the check.

**Case: a wrong expected value**

```typescript
it("adds two numbers", () => {
  expect(add(1, 2)).toBe(4);
});
```

Result: the test runner marks the case failed, prints the expected and actual values, and the process exits non-zero.
Why: `1 + 2` is `3`. The matcher wanted `4`. A failure is that non-zero exit plus the printed values.

**Case: nested groups**

```typescript
describe("add", () => {
  describe("success", () => {
    it("adds two numbers", () => {
      expect(add(1, 2)).toBe(3);
    });
  });
});
```

Result: the reported name is the chain `add` / `success` / `adds two numbers`.
Why: nested `describe` blocks group success and failure cases. The full name in the output is the chain of strings.

**Case: skip, only, and todo**

```typescript
it.skip("adds two numbers", () => {
  expect(add(1, 2)).toBe(3);
});

it.only("adds zero", () => {
  expect(add(1, 0)).toBe(1);
});

it.todo("adds negatives");
```

Result: the runner skips the first case, runs only "adds zero", and lists "adds negatives" as not written. The rest of the file stays hidden while `only` is present.
Why: `it.skip` leaves a test in the file without running it. `it.only` hides the rest. Remove `only` before you trust the suite. `it.todo` records a test you have not written.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Leaving `it.only` in the file | The suite looks green while other cases never ran | Remove `only` before you trust the suite |
| A file named `add.ts` with no `.test` | The runner does not collect it | Name it `add.test.ts` |
| `vitest` when you wanted one pass in a script | The process stays open and watches | `vitest run` executes once and exits |
| An empty `it` with no `expect` | The case passes and checks nothing | Add a matcher. A test must be able to fail |

### Tip

Read the failure as "expected" versus "actual" before you change the function. The printout is the check you wrote against the value you got.

### Practice

In `add.test.ts`, group two passing cases for `add` under `describe("add")`.

### Solution

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

Result: `vitest run add.test.ts` runs both cases once and exits. Both pass.

## 2. Matchers

### Concept

`toBe` uses `Object.is`. Use it for numbers, strings, and booleans. `toEqual` is deep equality. Use it for objects and arrays. `toBe` on two objects fails when they are equal in contents and different in identity. That surprise is why objects use `toEqual`. `toStrictEqual` is deep equality, and no extra `undefined` keys, and matching types. `expect(value).not.toBe(0)` inverts a matcher.

### Terms

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

### Methods

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
| `.not` | Inverts the matcher, as in `expect(value).not.toBe(0)` |

### Examples

**Case: `toBe` for a number**

```typescript
expect(add(1, 2)).toBe(3);
expect("Irvine").toBe("Irvine");
expect(true).toBe(true);
```

Result: the test runner marks each assertion passed.
Why: `toBe` is `Object.is`. Numbers, strings, and booleans with the same value pass.

**Case: `toBe` on two objects**

```typescript
expect({ city: "Irvine" }).toBe({ city: "Irvine" });
```

Result: the test runner fails the assertion. The expected and actual objects are printed as different.
Why: the contents match and the identities do not. `toBe` uses `Object.is`. Objects use `toEqual`.

**Case: `toEqual` and `toMatchObject`**

```typescript
expect({ city: "Irvine", price: 1 }).toEqual({ city: "Irvine", price: 1 });
expect({ city: "Irvine", price: 1 }).toMatchObject({ city: "Irvine" });
```

Result: both assertions pass. The second one ignores `price`.
Why: `toEqual` is deep equality of the whole value. `toMatchObject` passes when the object contains at least these fields.

**Case: a thrown error and an inverted matcher**

```typescript
expect(() => {
  throw new Error("bad price");
}).toThrow(/price/);

expect(1).not.toBe(0);
```

Result: the runner passes both checks. The function threw an error whose message matches `/price/`, and `1` is not `0`.
Why: `toThrow` accepts a string, a regex, or an error class. `.not` inverts a matcher.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `toBe` on two equal objects | The assertion fails even though the fields match | `toEqual` |
| `toEqual` when an extra `undefined` key must fail the test | Deep equality can still pass | `toStrictEqual` |
| `toBeTruthy` for a count | `1` passes, and so does any other truthy value | `toBe` or `toBeGreaterThan` |
| `toThrow` on a function you already called | The error escapes before `expect` can catch it | Pass a function: `expect(() => load()).toThrow()` |

### Tip

If the failure says the objects look the same, switch `toBe` to `toEqual`. The matcher was comparing identity.

### Practice

Assert that `{ city: "Irvine", price: 1 }` equals the same fields, and that a function throwing `bad price` matches `/price/`.

### Solution

```typescript
expect({ city: "Irvine", price: 1 }).toEqual({ city: "Irvine", price: 1 });
expect(() => {
  throw new Error("bad price");
}).toThrow(/price/);
```

Result: the runner passes the deep equality check and the thrown-message check.

## 3. Async tests and time

### Concept

Return the promise, or use `async` and `await`. A test that forgets to await finishes before the check runs, and a later failure can be lost. `rejects` and `resolves` are how you assert on a promise from inside `expect`. Fake timers replace `setTimeout` and `setInterval`. A timeout case should fail because the deadline passed, not because the suite itself hung. Fake timers make that fast.

### Terms

| Term | Meaning |
| --- | --- |
| `async` test | The function returns a promise the runner waits for |
| Lost failure | An assertion that runs after the test already finished |
| Fake timers | A clock the test controls |
| Deadline | The moment the timeout case should reject or resolve |

### Methods

| Timer API | Effect |
| --- | --- |
| `await expect(promise).resolves.toEqual(value)` | The promise fulfills with that value |
| `await expect(promise).rejects.toThrow(pattern)` | The promise rejects |
| `vi.useFakeTimers()` | Take over timers |
| `vi.advanceTimersByTimeAsync(ms)` | Move the clock and flush the resulting promises |
| `vi.runAllTimersAsync()` | Run every scheduled timer |
| `vi.useRealTimers()` | Give the clock back |

### Examples

**Case: an async success**

```typescript
it("loads", async () => {
  const rows = await loadCities();
  expect(rows).toEqual(["Irvine"]);
});
```

Result: the test runner waits for `loadCities` and passes when the array equals `["Irvine"]`.
Why: `async` and `await` return the promise to the runner. `toEqual` checks the array.

**Case: a forgotten await**

```typescript
it("loads", () => {
  const pending = loadCities();
  expect(pending).toEqual(["Irvine"]);
});
```

Result: the runner finishes the case before the request settles. The assertion compares a promise to an array and fails, or a later rejection is lost.
Why: a test that forgets to await finishes before the check runs, and a later failure can be lost. Return the promise, or use `async` and `await`.

**Case: a rejection**

```typescript
it("rejects on a bad id", async () => {
  await expect(loadCity("nope")).rejects.toThrow(/not found/);
});
```

Result: the runner waits for the promise and passes when it rejects with a message that matches `/not found/`.
Why: `rejects.toThrow` is the matcher for a rejected promise. The `await` in front of `expect` makes the runner wait.

**Case: a timeout without hanging the suite**

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

Result: the runner moves the clock 20ms, the race resolves to `"timeout"`, and the case passes without waiting 10 seconds.
Why: `vi.useFakeTimers()` takes over the clock. `advanceTimersByTimeAsync` moves it and flushes the promises. `useRealTimers` gives the clock back. The case fails because the deadline passed, not because the suite hung.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Forgetting `await` on `expect(...).rejects` | The case ends before the rejection is asserted | `await expect(promise).rejects.toThrow(...)` |
| A real `setTimeout(10_000)` in the suite | The file hangs until the deadline, or the suite times out | Fake timers, then `advanceTimersByTimeAsync` |
| `vi.useFakeTimers()` with no restore | Later tests inherit the fake clock | `vi.useRealTimers()` in `afterEach` |
| `vi.advanceTimersByTimeAsync` for less time than the deadline | The assertion runs before the timer callback | Advance at least the deadline |

### Tip

If an async test passes and a rejection shows up after the summary, the test finished early. Await the promise.

### Practice

Write an async case that expects `loadCity("nope")` to reject with `/not found/`.

### Solution

```typescript
it("rejects on a bad id", async () => {
  await expect(loadCity("nope")).rejects.toThrow(/not found/);
});
```

Result: the runner waits, and the case passes only when that promise rejects with a matching message.

## 4. Setup and doubles

### Concept

`beforeEach` runs before every test in this file or describe. `afterEach` runs after every test. `beforeAll` and `afterAll` run once. A double replaces a collaborator. `vi.fn()` is a function you pass in. `vi.spyOn` wraps one method you own. `vi.mock` replaces a whole imported module. Mock only the edge: the network, the clock, the database. Do not mock the function you are trying to test. `vi.stubEnv` sets an environment variable for the test.

### Terms

| Hook | Runs |
| --- | --- |
| `beforeEach` | Before every test in this file or describe |
| `afterEach` | After every test |
| `beforeAll` / `afterAll` | Once per file or describe |

| | `vi.fn()` | `vi.spyOn(obj, "method")` | `vi.mock("./db")` |
| --- | --- | --- | --- |
| What it replaces | A new function you pass into the code | One method. The original is saved | A whole imported module |
| The real code still runs | Only if you set the implementation to it | Yes, until you change the implementation | No, unless the factory calls through |
| How you put it back | `mockReset`, or a new test | `mockRestore` in `afterEach` | `vi.unmock`, or reset modules |
| Use it when | The code takes a callback | You want to watch a method you own | The code imports a module you cannot pass in |

### Methods

| Tool | Use |
| --- | --- |
| `vi.fn()` | A function that records its calls. `expect(fn).toHaveBeenCalledWith("Irvine")` |
| `vi.fn().mockReturnValue(1)` | A synchronous return |
| `vi.fn().mockResolvedValue(rows)` | A promise that fulfills |
| `vi.fn().mockRejectedValue(new Error("x"))` | A promise that rejects |
| `vi.spyOn(obj, "method")` | Wrap a real method. Restore it in `afterEach` |
| `vi.mock("./db")` | Replace a module. Hoisted. The factory returns the exports |
| `vi.restoreAllMocks()` | Restores spies. A common `beforeEach` line |
| `expect(fn).toHaveBeenCalledOnce()` | The double was called one time |
| `vi.stubEnv("PORT", "4000")` | Sets an environment variable for the test |
| `vi.unstubAllEnvs()` | Clears environment stubs |

### Examples

**Case: a function double**

```typescript
const save = vi.fn().mockResolvedValue(undefined);
await save("Irvine");
expect(save).toHaveBeenCalledOnce();
expect(save).toHaveBeenCalledWith("Irvine");
```

Result: the runner records one call with `"Irvine"`, the promise fulfills as `undefined`, and both matchers pass.
Why: `vi.fn()` records its calls. `mockResolvedValue` makes the promise fulfill. The matchers read the recording.

**Case: restore spies before each test**

```typescript
import { beforeEach, vi } from "vitest";

beforeEach(() => {
  vi.restoreAllMocks();
});
```

Result: each test starts with spies restored, so a `spyOn` from the previous test does not leak.
Why: `beforeEach` runs before every test in this file or describe. `mockRestore` in `afterEach` is the matching cleanup for one spy.

**Case: spy on one method**

```typescript
const store = {
  save(city: string) {
    return city.length;
  },
};

const spy = vi.spyOn(store, "save");
store.save("Irvine");
expect(spy).toHaveBeenCalledWith("Irvine");
spy.mockRestore();
```

Result: the real `save` still runs and returns `6`. The spy records the argument. `mockRestore` puts the original method back.
Why: `vi.spyOn` wraps one method and saves the original. Use it when you want to watch a method you own.

**Case: mock a module, and stub an environment variable**

```typescript
import { vi } from "vitest";

vi.mock("./db", () => ({
  loadCities: vi.fn().mockResolvedValue(["Irvine"]),
}));

vi.stubEnv("PORT", "4000");
```

Result: an import of `./db` receives the factory's exports, and `process.env.PORT` is `"4000"` for the test.
Why: `vi.mock` is hoisted, and the factory returns the exports. The real module does not run unless the factory calls through. `vi.unstubAllEnvs()` clears the stub.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Mocking the function under test | The case never runs the behavior you meant to check | Mock only the edge: the network, the clock, the database |
| `vi.mock` written as if it runs on that line | The factory is hoisted above the imports | Remember it is hoisted. The factory returns the exports |
| A spy with no `mockRestore` | The next test still sees the wrapped method | Restore it in `afterEach`, or call `vi.restoreAllMocks()` |
| `mockReturnValue` for an async edge | The caller gets a plain value where it expected a promise | `mockResolvedValue` or `mockRejectedValue` |

### Tip

Pass a `vi.fn()` into code that already takes a callback. Reach for `vi.mock` only when the code imports a module you cannot pass in.

### Practice

Create a double that fulfills with `undefined` when called with `"Irvine"`, and assert it was called once with that string.

### Solution

```typescript
const save = vi.fn().mockResolvedValue(undefined);
await save("Irvine");
expect(save).toHaveBeenCalledOnce();
expect(save).toHaveBeenCalledWith("Irvine");
```

Result: the runner passes both call matchers. The double replaced a collaborator, not the function under test.

## 5. What to test

### Concept

Test behavior a caller can see. A good case has an input and one expected outcome. Cover the happy path, a rejected input, a boundary such as an empty list, zero, or null, and a timeout or a late result that must not clobber a newer call. Do not assert the number of times a private helper ran. Do not snapshot an entire page unless you are ready to reread the snapshot on every copy change. A snapshot of a small pure value is fine. Coverage counts which lines ran. It does not count which behaviors you checked. A suite can cover every line and still miss the failure case. Read the uncovered lines, then add a case if that line is a behavior. A test must be able to fail. If you delete the assertion and the test still passes, it is not testing anything.

### Terms

| Case | Assert |
| --- | --- |
| The happy path | The returned value |
| A rejected input | The error |
| A boundary | Empty list, zero, null |
| A timeout or a late result | The call ends, and it does not clobber a newer call |

### Methods

| Command | Result |
| --- | --- |
| `vitest run` | One pass |
| `vitest run path/to/file.test.ts` | One file |
| `vitest run -t "adds"` | Tests whose name matches |
| `vitest run --coverage` | A coverage report, after the coverage package is installed |
| `expect(result).toEqual(...)` | The behavior check. Deleting it must be able to make the test fail |
| `expect(value).toMatchSnapshot()` | Fine for a small pure value. A whole page means you reread it on every copy change |

### Examples

**Case: the happy path and a boundary**

```typescript
function priceLabel(amount: number | null): string {
  if (amount === null) return "missing";
  if (amount === 0) return "free";
  return `${amount}`;
}

it("returns the amount", () => {
  expect(priceLabel(40)).toBe("40");
});

it("returns missing for null", () => {
  expect(priceLabel(null)).toBe("missing");
});
```

Result: the runner passes the returned-value case and the null boundary.
Why: each case has an input and one expected outcome. The boundary is null. Zero would be the other boundary, `"free"`.

**Case: a rejected input**

```typescript
it("rejects a bad id", async () => {
  await expect(loadCity("nope")).rejects.toThrow(/not found/);
});
```

Result: the runner passes when that input rejects, and fails when the function fulfills or throws a different message.
Why: a rejected input is asserted as the error. The caller can see that failure.

**Case: a late result must not clobber a newer call**

```typescript
it("keeps the newer city", async () => {
  const first = Promise.resolve("Irvine");
  const second = Promise.resolve("Tustin");
  let shown = "";
  const paint = (city: string) => {
    shown = city;
  };
  const newer = second.then(paint);
  await first.then((city) => {
    if (shown === "") paint(city);
  });
  await newer;
  expect(shown).toBe("Tustin");
});
```

Result: the runner passes when the value a caller can see is the newer city.
Why: a late result is a behavior. The case checks that it does not clobber a newer call. It does not count how many times a private helper ran.

**Case: a test that cannot fail**

```typescript
it("loads", async () => {
  await loadCities();
});
```

Result: the runner marks the case passed. Deleting an assertion is impossible here because there is no assertion, so the case still passes.
Why: a test must be able to fail. If you delete the assertion and the test still passes, it is not testing anything. Coverage can still count the lines inside `loadCities` and miss this gap.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Asserting how many times a private helper ran | The test breaks when you restructure the function and the caller sees the same result | Assert the returned value or the error |
| Snapshotting an entire page | Every copy change fails the suite and you stop reading the diff | Snapshot a small pure value, or assert the fields you mean |
| Trusting 100% line coverage | Every line ran, and the failure case can still be missing | Read the uncovered lines, then add a case if that line is a behavior |
| A case with no `expect` | The runner reports a pass | Add a matcher that fails when the behavior changes |

### Tip

Name the case after the outcome the caller sees: "returns null for an empty string", not "calls the helper once."

### Practice

Write two cases for a function that returns `null` on an empty string and a number otherwise. Each case needs an assertion.

### Solution

```typescript
function parseAmount(text: string): number | null {
  if (text === "") return null;
  return Number(text);
}

it("returns null for an empty string", () => {
  expect(parseAmount("")).toBe(null);
});

it("returns a number otherwise", () => {
  expect(parseAmount("40")).toBe(40);
});
```

Result: the runner passes both cases. Removing either `expect` would leave that case unable to fail.

## Drill

1. Write two cases for a function that returns `null` on an empty string and a number otherwise.
2. Write an async case that expects a rejection.
3. Say why objects use `toEqual`.

### Solution 1

```typescript
function parseAmount(text: string): number | null {
  if (text === "") return null;
  return Number(text);
}

describe("parseAmount", () => {
  it("returns null for an empty string", () => {
    expect(parseAmount("")).toBeNull();
  });

  it("returns a number otherwise", () => {
    expect(parseAmount("40")).toBe(40);
  });
});
```

Result: the runner passes the empty-string case with `toBeNull` and the other case with `toBe(40)`.

### Solution 2

```typescript
it("rejects on a bad id", async () => {
  await expect(loadCity("nope")).rejects.toThrow(/not found/);
});
```

Result: the runner waits for the promise and passes only when it rejects with a message that matches `/not found/`.

### Solution 3

`toBe` uses `Object.is`. Two objects with the same fields are different objects, so `toBe` fails when the contents match and the identity does not. `toEqual` checks deep equality, which is the comparison you want for objects and arrays.
