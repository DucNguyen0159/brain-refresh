# React

Course 5 of 15. Previous: [04-typescript.md](04-typescript.md). Next: [06-nextjs.md](06-nextjs.md).

React builds a user interface from components. You keep state. React calls your functions again and updates the screen to match. This course uses function components and hooks. The error-boundary section is the one class. The rest are functions.

Every section has the same parts: the concept, the terms, the methods, several cases with what the screen does, common mistakes, a tip, and a practice with a solution.

## Contents

1. [Components and JSX](#1-components-and-jsx)
2. [Props and children](#2-props-and-children)
3. [Rendering lists and conditions](#3-rendering-lists-and-conditions)
4. [State](#4-state)
5. [Events and forms](#5-events-and-forms)
6. [Effects](#6-effects)
7. [Refs, memo, and reducer](#7-refs-memo-and-reducer)
8. [Context and custom hooks](#8-context-and-custom-hooks)
9. [Rules and structure](#9-rules-and-structure)
10. [Error boundaries](#10-error-boundaries)

## 1. Components and JSX

### Concept

A component is a function whose name starts with a capital letter. It returns elements. JSX looks like HTML and produces those elements. A component returns one tree. A fragment `<>...</>` groups nodes without adding a DOM node. `class` is a JavaScript word, so the attribute is `className`. `htmlFor` is the HTML `for` attribute. `{value}` is a JavaScript expression. Empty tags close themselves: `<img />`.

### Terms

| Term | Meaning |
| --- | --- |
| Component | A function that returns elements |
| JSX | The HTML-like syntax |
| Fragment | `<>...</>` with no DOM node |
| Expression | `{name}` inside JSX |
| Element | The object JSX produces. Not a DOM node yet |

### Methods

| Rule | Why |
| --- | --- |
| One parent, or a fragment | A return has one tree |
| `className` | `class` is reserved |
| `htmlFor` | The label's `for` |
| Close every tag | `<img />`, `<input />` |
| Capital component name | A lowercase name is an HTML tag |

### Examples

**Case: a component renders text**

```tsx
function Greeting({ name }: { name: string }) {
  return <p>Hello, {name}</p>;
}
```

Result: `<Greeting name="Irvine" />` shows `Hello, Irvine`.

Why: `{name}` inserts the string. The function runs again when the parent renders it again.

**Case: two siblings need a fragment**

```tsx
function Pair() {
  return (
    <>
      <h1>Irvine</h1>
      <p>3 beds</p>
    </>
  );
}
```

Result: the heading and the paragraph appear. No extra wrapper element is in the DOM.

Why: `return <h1>...</h1><p>...</p>` does not compile. JSX wants one root.

**Case: a lowercase component is a DOM tag**

```tsx
function greeting() {
  return <p>Hi</p>;
}
```

```tsx
<greeting />
```

Result: React looks for an HTML tag named `greeting`, not your function. The function is ignored.

Why: names that start with a lowercase letter are host components. Rename the function `Greeting`.

**Case: zero and `&&`**

```tsx
function Count({ n }: { n: number }) {
  return <p>{n && <span>{n} beds</span>}</p>;
}
```

Result: `<Count n={3} />` shows `3 beds`. `<Count n={0} />` shows `0`, because `0` is rendered. The span is not shown.

Why: `&&` returns the left value when it is falsy. `0` is falsy and it is still a renderable number. Use `{n > 0 ? <span>{n} beds</span> : null}`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Two elements returned with no fragment | Does not compile | Wrap them in `<>...</>` |
| `class=` in JSX | Does not compile | `className` |
| A component named `card` | React treats it as a DOM tag | `Card` |
| `{count && <span />}` when count can be 0 | The page shows `0` | A ternary that returns `null` |

### Tip

If the alternative is "show nothing", write `condition ? <p>Yes</p> : null`. Use `&&` only when the left side is a boolean.

### Practice

Write `Greeting` so the name `Ada` renders `Hello, Ada`.

### Solution

```tsx
function Greeting({ name }: { name: string }) {
  return <p>Hello, {name}</p>;
}
```

Result: `<Greeting name="Ada" />` shows `Hello, Ada`.

## 2. Props and children

### Concept

Props are the arguments. They are read-only. The parent owns the values. `children` is whatever was nested between the tags. If two siblings need the same data, store it in the parent and pass it down. If a child must change it, pass a function down too.

### Terms

| Term | Meaning |
| --- | --- |
| Prop | A named input |
| `children` | The nested content |
| `React.ReactNode` | Text, elements, a list, or null |
| Lifting state | The parent holds the value both children need |
| Controlled child | The child receives the value and a function that changes it |

### Methods

| Form | Meaning |
| --- | --- |
| `function Card({ city }: CardProps)` | Reads props. Does not assign to them |
| `children?: React.ReactNode` | The nested nodes are optional |
| `city = "Unknown"` | A default when the prop is `undefined` |
| `<Card {...card} />` | Passes each field. Fine when the object matches the props |
| `onCity: (next: string) => void` | The parent hands down the updater |

### Examples

**Case: props render, the child does not own them**

```tsx
type CardProps = { city: string; price: number; children?: React.ReactNode };
function Card({ city, price, children }: CardProps) {
  return (
    <article>
      <h2>{city}</h2>
      <p>${price.toLocaleString()}</p>
      {children}
    </article>
  );
}
```

Result: `<Card city="Irvine" price={999000}><p>3 beds</p></Card>` shows the heading `Irvine`, the price `999,000` in a US locale, and the paragraph `3 beds`.

Why: `children` is the paragraph between the tags. `price` is a number. The child does not store a second copy.

**Case: a default**

```tsx
function Card({ city = "Unknown" }: { city?: string }) {
  return <h2>{city}</h2>;
}
```

Result: `<Card />` shows `Unknown`. `<Card city="Tustin" />` shows `Tustin`.

Why: the default runs when the prop is `undefined`. It does not run when the parent passes `""`.

**Case: lifting the city**

```tsx
function Editor({ city, onCity }: { city: string; onCity: (next: string) => void }) {
  return <input value={city} onChange={(event) => onCity(event.target.value)} />;
}
```

Result: the input shows the parent's `city`. Typing calls `onCity` with the new string. The input does not keep its own city.

Why: the parent holds the state so a sibling can read the same string. The child is a view.

**Case: assigning to a prop**

```tsx
function Card({ city }: { city: string }) {
  city = "Tustin";
  return <h2>{city}</h2>;
}
```

Result: the local parameter changes for this render only. The parent does not update. The next render from the parent puts the old city back. A `const` destructure would not compile if you assign to `city`.

Why: props are inputs. Change them by calling the function the parent passed.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Copying a prop into state "so the child can edit it" | The child goes stale when the parent changes | Pass the value and an updater |
| Forgetting `children` in the props type | The nested content is dropped or does not type-check | `children?: React.ReactNode` |
| `<Card {...card} />` when `card` has extra fields | Those fields are passed too | Pass the fields you mean |
| Mutating `props.city` | The parent does not rerender | Call `onCity` |

### Tip

If two components must agree, the state lives in the parent they share. Pass the value down and the updater down.

### Practice

Render a `Card` for `Tustin` at price `400000` with a child paragraph `condo`.

### Solution

```tsx
<Card city="Tustin" price={400000}>
  <p>condo</p>
</Card>
```

Result: heading `Tustin`, a formatted price, and the word `condo`.

## 3. Rendering lists and conditions

### Concept

`map` turns an array into elements. The `key` goes on the element `map` returns, and it must be stable and unique among siblings. A database id is a good key. The array index is a bad key when the list can be reordered or filtered. An empty list that finished loading is different from data that has not arrived.

### Terms

| Term | Meaning |
| --- | --- |
| `key` | How React matches an old row to a new row |
| Stable key | The same row keeps the same key across renders |
| Index key | The position. It changes when the list is sorted |
| Empty state | A finished request with no rows |

### Methods

| Rule | Why |
| --- | --- |
| `key={card.id}` | Survives a sort |
| Key on the `map` result | React reads it there, not on a grandchild |
| `cards.length === 0` | Show "No results" only after the load finished |
| Duplicate keys | React may reuse the wrong row's state |

### Examples

**Case: a list with ids**

```tsx
function List({ cards }: { cards: { id: string; city: string; price: number }[] }) {
  if (cards.length === 0) return <p>No results.</p>;
  return (
    <ul>
      {cards.map((card) => (
        <li key={card.id}>{card.city}</li>
      ))}
    </ul>
  );
}
```

Result: one item per card. An empty array shows `No results.`

Why: the early return is the empty state. Each `li` has a key React can match next time.

**Case: the key on the wrong element**

```tsx
cards.map((card) => (
  <li>
    <Card key={card.id} city={card.city} price={card.price} />
  </li>
));
```

Result: React warns that the `li` has no key. The key on `Card` does not identify the `li`.

Why: the key belongs on the element returned from `map`.

**Case: index keys scramble an input**

```tsx
{cities.map((city, index) => (
  <input key={index} defaultValue={city} />
))}
```

Result: the first paint looks right. After you delete the first city, the remaining input can keep the deleted city's typed text, because React reused the component at index 0.

Why: the key is the position, and the position now holds a different city. `key={city}` stays with the city.

**Case: loading is not empty**

```tsx
function List({ status, cards }: { status: "loading" | "ready"; cards: { id: string; city: string }[] }) {
  if (status === "loading") return <p>Loading</p>;
  if (cards.length === 0) return <p>No results.</p>;
  return <ul>{cards.map((card) => <li key={card.id}>{card.city}</li>)}</ul>;
}
```

Result: while `status` is `"loading"`, the page says `Loading` even if `cards` is `[]`. When the status is `"ready"` and the array is empty, it says `No results.`

Why: an empty array means two different things until you record whether the request finished.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| No key | A warning, and state can attach to the wrong row | `key={card.id}` |
| `key={index}` on a sortable list | Typed state sticks to the position | A stable id |
| Key only on the inner component | The warning remains on the outer element | Put the key on the `map` result |
| Treating `[]` as "still loading" | The empty message flashes before the data arrives | A status flag |

### Tip

The key answers "which row is this?" The index answers "where is it right now?" Use the answer that stays true after a sort.

### Practice

Map `{ id: "R1", city: "Irvine" }` into one `li` with a stable key.

### Solution

```tsx
<li key="R1">Irvine</li>
```

Inside `map`, that is `key={card.id}`. Result: one list item, and the key stays `R1` if the row moves.

## 4. State

### Concept

`useState` keeps a value across renders. The argument is the starting value on the first render. `setCount` schedules a new render. It does not change `count` in the current function run. Use the functional updater when the next value depends on the previous one. Replacing an object is `setCard({ ...card, city })`. Mutating `card.city` does not rerender.

### Terms

| Term | Meaning |
| --- | --- |
| State | A value React keeps between renders |
| Snapshot | `count` in this render stays the same until the next render |
| Updater | `setCount` |
| Functional update | `setCount((n) => n + 1)` |
| Lazy start | `useState(() => expensive())` runs once |

### Methods

| Call | Result |
| --- | --- |
| `useState(0)` | `[count, setCount]`. First render uses `0` |
| `setCount(count + 1)` | Next render uses this render's count plus one |
| `setCount((n) => n + 1)` | Next render uses the latest count plus one |
| `setCard({ ...card, city })` | A new object. React sees a change |
| `useState(() => expensive())` | `expensive` runs on the first render only |

### Examples

**Case: a click schedules the next count**

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount((n) => n + 1)}>{count}</button>;
}
```

Result: the button shows `0`. Each click shows the next integer.

Why: the click calls the updater. React renders again and `count` is the new value. `console.log(count)` immediately after `setCount` still prints the old value.

**Case: two updates from the same snapshot**

```tsx
function onClick() {
  setCount(count + 1);
  setCount(count + 1);
}
```

Result: one click from `0` leaves the button at `1`, not `2`.

Why: both calls read `count` from this render, which is `0`. The functional form stacks: `setCount((n) => n + 1)` twice ends at `2`.

**Case: mutating an object does not render**

```tsx
const [card, setCard] = useState({ city: "Irvine" });
function onClick() {
  card.city = "Tustin";
}
```

Result: the heading stays `Irvine`. The object changed and React was not told.

Why: the state is still the same object reference. `setCard({ ...card, city: "Tustin" })` passes a new object and the heading updates.

**Case: a lazy initial value**

```tsx
const [now, setNow] = useState(() => new Date());
```

Result: `new Date()` runs on the first render. Later renders reuse that date until `setNow` is called.

Why: `useState(new Date())` also creates a date on later renders and then throws it away. The function form skips that work.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Reading state right after `setState` | You see the old value | Use the value in the next render, or the updater |
| `setCount(count + 1)` twice | One increment | `setCount((n) => n + 1)` |
| `card.city = "Tustin"` | No render | Replace the object |
| `useState(expensive())` | `expensive` runs every render | `useState(() => expensive())` |

### Tip

State is a snapshot. The `count` in your hand does not change. `setCount` asks for a new snapshot.

### Practice

Write a button that shows a count starting at `0` and adds `1` on each click, correctly if the handler updates twice.

### Solution

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button
      onClick={() => {
        setCount((n) => n + 1);
        setCount((n) => n + 1);
      }}
    >
      {count}
    </button>
  );
}
```

Result: each click adds `2`.

## 5. Events and forms

### Concept

Handlers receive an event. `preventDefault` stops a form from navigating. A controlled input shows state and writes state on every change. An uncontrolled input keeps its own text and you read a ref when you need it. Controlled inputs are the default in this course. Pass a function to `onClick`. `onClick={save()}` calls `save` while rendering.

### Terms

| Term | Meaning |
| --- | --- |
| Controlled | `value` and `onChange` are tied to state |
| Uncontrolled | The DOM holds the text. A ref reads it |
| Synthetic event | React's event object. `preventDefault` still works |
| Submit | The form event, including the Enter key |

### Methods

| Handler | Event |
| --- | --- |
| `onClick` | Click |
| `onChange` | The value changed |
| `onSubmit` | Form submit |
| `onFocus` / `onBlur` | Focus entered or left |
| `event.preventDefault()` | Cancels the browser's default |

### Examples

**Case: a controlled search**

```tsx
function Search() {
  const [city, setCity] = useState("");
  const [message, setMessage] = useState("");
  function onSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();
    setMessage(city.trim() ? `Searching ${city}` : "Enter a city");
  }
  return (
    <form onSubmit={onSubmit}>
      <input value={city} onChange={(event) => setCity(event.target.value)} />
      <button type="submit">Search</button>
      <p>{message}</p>
    </form>
  );
}
```

Result: typing `Irvine` shows `Irvine` in the box. Submit shows `Searching Irvine` and the page does not navigate. An empty submit shows `Enter a city`.

Why: `value={city}` makes the input controlled. `preventDefault` cancels the navigation. The message is state, so it renders.

**Case: calling the handler during render**

```tsx
<button onClick={save()}>Save</button>
```

Result: `save` runs while rendering, not on click. If `save` sets state, you can get an infinite loop.

Why: `save()` is a call. `onClick={save}` is the function to call later. Arguments need a wrapper: `onClick={() => save(id)}`.

**Case: a missing `preventDefault`**

```tsx
<form onSubmit={() => setMessage("Searching")}>
  <button type="submit">Search</button>
</form>
```

Result: the message may flash, and the browser then navigates to the current URL. The state is gone.

Why: submit's default is navigation. `preventDefault` has to run first.

**Case: uncontrolled read**

```tsx
function Field() {
  const ref = useRef<HTMLInputElement>(null);
  return (
    <form
      onSubmit={(event) => {
        event.preventDefault();
        console.log(ref.current?.value);
      }}
    >
      <input ref={ref} defaultValue="Irvine" />
      <button type="submit">Search</button>
    </form>
  );
}
```

Result: the console prints the input's current text on submit. Typing does not rerender the component.

Why: there is no state. `defaultValue` sets the initial DOM value only. This course uses controlled inputs unless you have a reason to skip rendering on each keystroke.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `onClick={save()}` | Runs during render | `onClick={save}` |
| No `preventDefault` on submit | The page reloads | Call it on the form event |
| `value` without `onChange` | The input is frozen | Update state in `onChange` |
| `defaultValue` and `value` together | `value` wins and `defaultValue` is ignored | One mode |

### Tip

`onClick={save}` waits. `onClick={save()}` runs now. The parentheses are the whole difference.

### Practice

Write a controlled input whose state starts as `""` and whose submit logs the trimmed city without navigating.

### Solution

The `Search` component in the first example, with `console.log(city.trim())` in place of `setMessage`.

Result: submit prints the trimmed city and stays on the page.

## 6. Effects

### Concept

`useEffect` runs after paint. Use it to synchronize with something outside React: a subscription, a timer, a manual DOM call. Fetching in an effect works. A data library is the usual tool once the app grows. The dependency list decides when the effect runs again. The cleanup function runs before the next run and when the component unmounts. Do not set a state value from an effect that depends on that same value with no condition. That loops.

### Terms

| Term | Meaning |
| --- | --- |
| Effect | A function after paint |
| Dependencies | The values the effect reads |
| Cleanup | The function the effect returns |
| Stale response | A slow request that finishes after a newer one |

### Methods

| Dependency list | Effect runs |
| --- | --- |
| Omitted | After every render |
| `[]` | After the first render, and the cleanup on unmount |
| `[city]` | After the first render, and after `city` changes |
| Returned function | Before the next run, and on unmount |

### Examples

**Case: an interval and its cleanup**

```tsx
function Clock() {
  const [now, setNow] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => setNow(new Date()), 1000);
    return () => clearInterval(id);
  }, []);
  return <p>{now.toLocaleTimeString()}</p>;
}
```

Result: the clock updates every second. When the component unmounts, the interval stops.

Why: `[]` means "start once." The returned function is the cleanup. Without it, every mount leaves another interval running.

**Case: a stale request**

```tsx
useEffect(() => {
  let cancelled = false;
  load(city).then((rows) => {
    if (!cancelled) setRows(rows);
  });
  return () => {
    cancelled = true;
  };
}, [city]);
```

Result: changing `city` from `Irvine` to `Tustin` drops Irvine's rows if they arrive late.

Why: the cleanup sets `cancelled` on the old effect. The old callback still runs, and it does not call `setRows`.

**Case: a missing dependency**

```tsx
useEffect(() => {
  setRows(filter(all, city));
}, []);
```

Result: the rows match the first `city` only. Later city changes do not update them.

Why: `[]` tells React the effect does not use `city`. It does. Put `city` in the list, or compute `filter(all, city)` during render and skip the effect.

**Case: an effect that sets its own dependency**

```tsx
useEffect(() => {
  setCity(city.toUpperCase());
}, [city]);
```

Result: if `toUpperCase` returns a different string, React runs the effect again, and again. If the city is already upper case, the same string can stop the loop. Depending on that is fragile.

Why: the effect writes the value it reads. Compute the label during render: `const label = city.toUpperCase()`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A timer with no cleanup | Intervals pile up | Return `clearInterval` |
| Fetch with no cancel flag | An old city overwrites a new one | Ignore the result in cleanup |
| An omitted dependency array | The effect runs after every render | Name the values, or `[]` if there are none |
| Derived state stored in an effect | Extra renders, sometimes a loop | Compute it while rendering |

### Tip

An effect talks to the outside world. A value you can compute from props and state is not an effect.

### Practice

Start an interval that increments a count every second, and clear it when the component unmounts.

### Solution

```tsx
function Ticker() {
  const [n, setN] = useState(0);
  useEffect(() => {
    const id = setInterval(() => setN((value) => value + 1), 1000);
    return () => clearInterval(id);
  }, []);
  return <p>{n}</p>;
}
```

Result: the number increases once a second and stops when the component is removed.

## 7. Refs, memo, and reducer

### Concept

A ref is a box that survives renders. Writing `.current` does not by itself cause a render. `useMemo` remembers a calculation. `useCallback` remembers a function. Add them when a measurement says the work matters, or when a memoized child would otherwise see a new function every render. `useReducer` is for updates that share a pattern. `React.memo` skips a child render when its props are shallowly equal. It is an optimization, not a correctness tool.

### Terms

| Term | Meaning |
| --- | --- |
| Ref | `{ current }` that persists |
| Memo | A cached value or a cached component render |
| Reducer | `(state, action) => nextState` |
| Dispatch | Sends an action to the reducer |

### Methods

| Hook | What you get back | What causes another render |
| --- | --- | --- |
| `useState` | `[value, setValue]` | `setValue` with a different value |
| `useReducer` | `[state, dispatch]` | `dispatch` |
| `useRef` | `{ current }` | Nothing |
| `useMemo` | The cached value | Nothing by itself |
| `useCallback` | The cached function | Nothing by itself |
| `React.memo` | A component that can skip | A changed prop |

### Examples

**Case: a ref focuses an input**

```tsx
function Field() {
  const inputRef = useRef<HTMLInputElement>(null);
  return (
    <>
      <input ref={inputRef} />
      <button type="button" onClick={() => inputRef.current?.focus()}>Focus</button>
    </>
  );
}
```

Result: the button moves focus to the input. The click does not store the focus in state, and it does not rerender because of the ref.

Why: `inputRef.current` is the DOM node after React commits. It is `null` on the first render function, before the commit.

**Case: a reducer**

```tsx
type Action = { type: "inc" } | { type: "set"; value: number };
function reducer(state: number, action: Action): number {
  switch (action.type) {
    case "inc":
      return state + 1;
    case "set":
      return action.value;
  }
}
function Counter() {
  const [count, dispatch] = useReducer(reducer, 0);
  return <button onClick={() => dispatch({ type: "inc" })}>{count}</button>;
}
```

Result: the button starts at `0` and shows `1` after a click. `dispatch({ type: "set", value: 4 })` shows `4`.

Why: the reducer is a pure function of state and action. The component does not compute the next count itself.

**Case: memo skips equal props**

```tsx
const Price = React.memo(function Price({ price }: { price: number }) {
  return <p>{price}</p>;
});
```

Result: if the parent renders again and `price` is still `999000`, `Price` does not run. If `price` changes, it runs.

Why: `memo` compares props with `Object.is`. A new object prop `{{ city: "Irvine" }}` is a new reference every time, so the child still runs. That is why `memo` without stable props does nothing useful.

**Case: `useMemo` does not change the result**

```tsx
const total = useMemo(() => prices.reduce((sum, price) => sum + price, 0), [prices]);
```

Result: `total` is the sum. It is recomputed when `prices` changes. It is not a state update.

Why: `useMemo` is a cache. The same line without `useMemo` returns the same sum. Use the cache when the calculation is expensive.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Storing input text in a ref and expecting the screen to update | The screen stays old | `useState` |
| Reading `ref.current` during the first render | It is still `null` | Read it in an effect or a handler |
| `memo` on a child that receives a fresh object | The child still renders | Pass primitives, or memoize the object |
| A reducer that mutates state | Later updates see a corrupted snapshot | Return a new value |

### Tip

If changing it should change the screen, it is state. If changing it should focus a box or remember a timer id, it is a ref.

### Practice

Focus an input from a button using a ref. Do not store the focus flag in state.

### Solution

The `Field` component in the first example.

Result: the click focuses the input and does not rerender because of the ref write.

## 8. Context and custom hooks

### Concept

Context passes a value to descendants without threading props through every layer. It is for data many distant components need, such as the signed-in user or a theme. When the provider value changes, the consumers render again. A custom hook is a function whose name starts with `use` and that may call other hooks.

### Terms

| Term | Meaning |
| --- | --- |
| Provider | The component that sets the value |
| Consumer | A descendant that calls `useContext` |
| Default | The value used when no provider is above the consumer |
| Custom hook | A `use*` function that can call hooks |

### Methods

| Call | Result |
| --- | --- |
| `createContext("light")` | A context whose default is `"light"` |
| `<ThemeContext.Provider value="dark">` | Descendants see `"dark"` |
| `useContext(ThemeContext)` | The nearest provider's value |
| `function useToggle()` | A hook. It must follow the rules of hooks |

### Examples

**Case: a provider overrides the default**

```tsx
const ThemeContext = createContext("light");
function Panel() {
  const theme = useContext(ThemeContext);
  return <p>{theme}</p>;
}
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Panel />
    </ThemeContext.Provider>
  );
}
```

Result: `Panel` shows `dark`. The same `Panel` with no provider shows `light`.

Why: `useContext` walks up to the nearest provider. The default is only for the missing-provider case.

**Case: a new object value rerenders consumers**

```tsx
function App() {
  const [city, setCity] = useState("Irvine");
  return (
    <ThemeContext.Provider value={{ city, setCity }}>
      <Panel />
    </ThemeContext.Provider>
  );
}
```

Result: every time `App` renders, the provider value is a new object. Every consumer renders, even if `city` did not change.

Why: context uses `Object.is` on `value`. Build the object with `useMemo`, or split a fast-changing value out of a theme that rarely changes.

**Case: a custom hook**

```tsx
function useToggle(start = false) {
  const [on, setOn] = useState(start);
  function toggle() {
    setOn((value) => !value);
  }
  return [on, toggle] as const;
}
```

Result: `const [on, toggle] = useToggle()` starts `false`. Calling `toggle()` flips it. `on` is a boolean and `toggle` is a function because of `as const`.

Why: the hook is a function that calls `useState`. Without `as const`, the array is inferred as a mix and the caller has a worse type.

**Case: a hook called from a plain function**

```tsx
function notAHook() {
  useState(0);
}
```

Result: if a component calls `notAHook`, it works only by accident of call order. If you call `notAHook` from an event or from outside a component, React throws.

Why: hooks are allowed in components and in custom hooks. The name should start with `use` so that rule is visible.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Context for one parent and one child | Extra machinery | A prop |
| A fresh object as `value` | Consumers render every time | Memoize the value, or pass a primitive |
| A custom hook that sometimes calls `useState` | The hook order changes and React throws | Call the hook unconditionally |
| Forgetting the provider | Consumers see the default and look "empty" | Wrap the tree |

### Tip

Context replaces a prop that would otherwise be forwarded through components that do not use it. It does not replace a local `useState`.

### Practice

Provide `"dark"` and read it in a child with `useContext`.

### Solution

The `App` and `Panel` in the first example.

Result: the child shows `dark`.

## 9. Rules and structure

### Concept

React matches hooks by call order. A hook inside a condition changes that order between renders, and React throws. Render should be safe to run twice in development. A fetch in the component body fires twice. A fetch in an effect is cleaned up between those runs. State in this render does not change later in this render.

### Terms

| Term | Meaning |
| --- | --- |
| Hook order | The sequence of hook calls on one render |
| Pure render | Same inputs, same output, no side effects |
| Strict mode double render | A development check. Production renders once |
| Snapshot | The state and props of this render |

### Methods

| Rule | Why |
| --- | --- |
| Hooks at the top level | Not inside `if` or a loop |
| Hooks only in components and custom hooks | A random function is not tracked |
| No side effects in render | Render may run twice in development |
| Stable keys | Index keys scramble state |
| One job per component | A file can hold a small private child |

### Examples

**Case: a conditional hook**

```tsx
function Panel({ on }: { on: boolean }) {
  if (on) {
    const [n, setN] = useState(0);
    return <p>{n}</p>;
  }
  return <p>off</p>;
}
```

Result: the first render may work. Toggling `on` changes how many hooks run, and React throws.

Why: hooks are stored by position. Call `useState` before the `if`, and use `on` only to choose the JSX.

**Case: a fetch in the body**

```tsx
function Panel() {
  fetch("/cards").then((response) => response.json());
  return <p>Irvine</p>;
}
```

Result: in development the request can fire twice. There is no cleanup. The component also cannot use the response unless it sets state, which would render again and fetch again.

Why: the body is render. Put the fetch in an effect with a cleanup, or in a data library.

**Case: the snapshot**

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  function onClick() {
    setCount(count + 1);
    console.log(count);
  }
  return <button onClick={onClick}>{count}</button>;
}
```

Result: the first click logs `0` and the button then shows `1`.

Why: `count` is the snapshot for this render. The log does not see the next snapshot.

**Case: a pure label**

```tsx
function Beds({ n }: { n: number }) {
  return <p>{n === 1 ? "1 bed" : `${n} beds`}</p>;
}
```

Result: `n` of `1` shows `1 bed`. `n` of `3` shows `3 beds`. Rendering twice shows the same text and starts nothing.

Why: the text is computed from props. No effect is required.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `useState` inside `if` | React throws after a toggle | Move the hook up |
| Fetching in the function body | Duplicate requests and no place to store the result | An effect or a query library |
| Assuming `setState` changes the local variable | The log shows the old value | Read state on the next render |
| One component that fetches, formats, and lays out the page | Hard to test | Split the pieces |

### Tip

Count the hook calls. If a branch can skip one, the hooks are in the wrong place.

### Practice

Rewrite a component that called `useState` only when `on` is true so the hook always runs.

### Solution

```tsx
function Panel({ on }: { on: boolean }) {
  const [n] = useState(0);
  if (!on) return <p>off</p>;
  return <p>{n}</p>;
}
```

Result: toggling `on` shows `off` or `0`. The hook count stays at one.

## 10. Error boundaries

### Concept

An error boundary catches a render error in a child and shows a fallback. The boundary itself is a class. Function components do not have this method. `try/catch` around JSX does not catch a child, because the child renders later. Event handlers and rejected promises are not caught by the boundary.

### Terms

| Term | Meaning |
| --- | --- |
| Render error | A throw while React is calling a component |
| Fallback | The UI the boundary shows instead of the child |
| `getDerivedStateFromError` | Switches state during render |
| `componentDidCatch` | Logs the error. It does not replace the tree by itself |

### Methods

| Error happens in | The boundary catches it |
| --- | --- |
| A child's render | Yes |
| A child's constructor | Yes |
| An event handler | No. Use `try/catch` there |
| `async` or a rejected promise | No. Catch it at the `await` |
| The boundary's own render | No. A parent boundary can |

### Examples

**Case: a child throw shows the fallback**

```tsx
import { Component, type ReactNode } from "react";

type BoundaryState = { error: Error | null };

class ErrorBoundary extends Component<{ children: ReactNode }, BoundaryState> {
  state: BoundaryState = { error: null };
  static getDerivedStateFromError(error: Error): BoundaryState {
    return { error };
  }
  render() {
    if (this.state.error) return <p>Something went wrong.</p>;
    return this.props.children;
  }
}

function PriceChart(): React.ReactElement {
  throw new Error("bad data");
}
```

```tsx
<ErrorBoundary>
  <PriceChart />
</ErrorBoundary>
```

Result: the page shows `Something went wrong.` The rest of the app outside the boundary stays up.

Why: `PriceChart` throws during render. `getDerivedStateFromError` stores the error. The next render of the boundary returns the fallback.

**Case: `try/catch` around the child does not catch**

```tsx
function Parent() {
  try {
    return <PriceChart />;
  } catch {
    return <p>Caught</p>;
  }
}
```

Result: you still get the React error screen, or the nearest boundary. You do not see `Caught`.

Why: `<PriceChart />` creates an element. It does not call the function. React calls it later.

**Case: a click is not a render error**

```tsx
function Save() {
  return <button onClick={() => { throw new Error("nope"); }}>Save</button>;
}
```

Result: the click throws. An error boundary around `Save` does not show the fallback.

Why: the throw is in the handler, after render. Wrap the handler body in `try/catch`.

**Case: a rejected fetch**

```tsx
useEffect(() => {
  fetch("/missing").then((response) => {
    if (!response.ok) throw new Error(String(response.status));
  });
}, []);
```

Result: an unhandled rejection. The boundary does not render the fallback.

Why: the throw is inside a promise, outside render. `catch` on the promise, or `try/catch` around `await`, is what handles it. Set an error state if the screen should change.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A function component named as a boundary with no class methods | Render errors still blank the tree | The class with `getDerivedStateFromError` |
| `try/catch` around JSX | The child throw escapes | A boundary |
| Expecting the boundary to catch `onClick` | It does not | `try/catch` in the handler |
| The boundary and the throwing component are the same class | A boundary does not catch itself | A parent boundary |

### Tip

A boundary is for broken render. A handler and a promise need their own `catch`.

### Practice

Name one error the boundary catches and one it does not, using `PriceChart` and a button handler.

### Solution

`PriceChart` throwing during render is caught and shows `Something went wrong.` A button `onClick` that throws is not caught.

## Drill

1. Write `Card` with `city` and `price` props.
2. Write a controlled city input.
3. Write an effect that starts an interval and clears it.
4. Name one error an error boundary catches, and one it does not.
5. Say why the key should be an id.

### Solution 1

```tsx
function Card({ city, price }: { city: string; price: number }) {
  return <article><h2>{city}</h2><p>{price}</p></article>;
}
```

`<Card city="Irvine" price={999000} />` shows that city and price.

### Solution 2

```tsx
const [city, setCity] = useState("");
<input value={city} onChange={(event) => setCity(event.target.value)} />
```

The box shows `city`. Typing replaces that state.

### Solution 3

The `Ticker` practice in section 6. The cleanup calls `clearInterval`.

### Solution 4

A child's render throw is caught. An event-handler throw is not.

### Solution 5

The id stays with the row when the list is sorted or filtered. The index moves, so React can attach the old row's state to a different row.
