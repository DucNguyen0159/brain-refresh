# React

Course 5 of 15. Previous: [04-typescript.md](04-typescript.md). Next: [06-nextjs.md](06-nextjs.md).

React builds a user interface from components. You keep state. React calls your functions again and updates the screen to match. This course uses function components and hooks. Class components are older and not required to read modern code.

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

A component is a function whose name starts with a capital letter. It returns elements. JSX is the syntax that looks like HTML and produces those elements.

```tsx
function Greeting({ name }: { name: string }) {
  return <p>Hello, {name}</p>;
}
```

| Rule | Why |
| --- | --- |
| One parent, or a fragment | A component returns one tree. `<>...</>` groups nodes without a DOM node |
| `className` | `class` is a JavaScript word |
| `htmlFor` | The HTML `for` attribute |
| `{value}` | A JavaScript expression inside JSX |
| Close every tag | `<img />` and `<input />` are empty |
| Components start with a capital | Lowercase names are treated as HTML tags |

`{condition && <p>Yes</p>}` renders the paragraph when `condition` is true. Do not use it with a number that might be `0`. `0 && <p />` prints `0`. Use a ternary when the alternative is "nothing": `{n > 0 ? <p>{n}</p> : null}`.

Style objects are camelCase: `style={{ marginTop: 8 }}`. Prefer a class.

## 2. Props and children

Props are the arguments. They are read-only. The parent owns the values.

```tsx
type CardProps = {
  city: string;
  price: number;
  children?: React.ReactNode;
};

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

| Idea | Meaning |
| --- | --- |
| Prop | A named input |
| `children` | Whatever was nested between the tags |
| `React.ReactNode` | Text, elements, a list of those, or null |
| Default | `function Card({ city = "Unknown" }: CardProps)` |
| Spread | `<Card {...card} />` passes each field. Fine when the object matches the props |

Lifting state: if two siblings need the same data, store it in their parent and pass it down. If a child must change it, pass a function down too.

```tsx
function Editor({ city, onCity }: { city: string; onCity: (next: string) => void }) {
  return <input value={city} onChange={(event) => onCity(event.target.value)} />;
}
```

## 3. Rendering lists and conditions

```tsx
function List({ cards }: { cards: { id: string; city: string; price: number }[] }) {
  if (cards.length === 0) return <p>No results.</p>;
  return (
    <ul>
      {cards.map((card) => (
        <li key={card.id}>
          <Card city={card.city} price={card.price} />
        </li>
      ))}
    </ul>
  );
}
```

| Topic | Rule |
| --- | --- |
| `key` | Stable and unique among siblings. A database id is good. The array index is not, if the list can be reordered or filtered |
| Key placement | On the element returned from `map`, not only on a child inside it |
| Empty | A finished empty list is different from data that has not loaded |
| Duplicate keys | React may reuse the wrong row's state |

## 4. State

`useState` keeps a value across renders.

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount((n) => n + 1)}>{count}</button>;
}
```

| Fact | Detail |
| --- | --- |
| First render | The argument to `useState` is the starting value |
| Update | `setCount` schedules a render. It does not change `count` in the current function run |
| Functional update | `setCount((n) => n + 1)` uses the latest value. Use it when the next value depends on the previous one |
| Object state | Replace the object: `setCard({ ...card, city })`. Mutating `card.city` does not rerender |
| Lazy start | `useState(() => expensive())` runs the function once |
| Same value | Setting the same state may skip a rerender |

Several `useState` calls are fine. One object is fine when the fields always change together.

## 5. Events and forms

Handlers receive a synthetic event. `event.preventDefault()` stops a form from navigating.

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

A controlled input shows state and writes state. An uncontrolled input keeps its own text and you read it with a ref when you need it. Controlled inputs are the default in this course.

| Handler | Event |
| --- | --- |
| `onClick` | Click |
| `onChange` | Value changed |
| `onSubmit` | Form submit |
| `onFocus` / `onBlur` | Focus |

Pass the function: `onClick={save}`. Do not call it during render: `onClick={save()}` runs `save` while rendering.

## 6. Effects

`useEffect` runs after paint. Use it to synchronize with something outside React: a subscription, a timer, a manual DOM call. Fetching in an effect works. A data library, covered in the TanStack course, is the usual tool once the app grows.

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

| Dependency list | Effect runs |
| --- | --- |
| omitted | After every render. Almost never what you want |
| `[]` | After the first render |
| `[city]` | After the first render, and after `city` changes |
| Cleanup function | Before the next run, and when the component unmounts |

The cleanup is how you ignore a stale request:

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

If `city` changes before `load` finishes, the old result is dropped.

Do not put a value into the dependency list and then set that same value unconditionally. That loops. Effects are for outside systems, not for computing a value you could compute during render.

## 7. Refs, memo, and reducer

A ref is a box that survives renders and does not, by itself, cause a render when you write `.current`.

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

| Hook | Use |
| --- | --- |
| `useRef` | A DOM node, or a mutable value that must not retrigger render |
| `useMemo(() => cost(items), [items])` | Remember a calculation. Add it when a measurement says the calculation is expensive |
| `useCallback(fn, [deps])` | Remember a function. Useful when a child is memoized and would otherwise see a new function every render |
| `useReducer` | Next state computed by a reducer, when many updates share a pattern |

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
```

`React.memo(Component)` skips a child render when its props are shallowly equal. It is an optimization, not a correctness tool.

The hooks side by side:

| Hook | When the body runs | What you get back | What causes another render |
| --- | --- | --- | --- |
| `useState` | Every render. The value is kept | `[value, setValue]` | `setValue` with a different value |
| `useReducer` | Every render | `[state, dispatch]` | `dispatch` |
| `useContext` | Every render | The nearest provider's value | The provider value changes |
| `useRef` | Every render | `{ current }` | Nothing. Writing `current` does not render |
| `useMemo` | Every render. The calculation runs again when a dependency changes | The cached value | Nothing by itself |
| `useCallback` | Every render. The function is reused when dependencies are unchanged | The cached function | Nothing by itself |
| `useEffect` | After paint, when a dependency changed | An optional cleanup | Nothing by itself. Setting state inside it does |

## 8. Context and custom hooks

Context passes a value to descendants without threading props through every layer. It is for data that many distant components need, such as the signed-in user or a theme.

```tsx
const ThemeContext = createContext("light");

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Panel />
    </ThemeContext.Provider>
  );
}

function Panel() {
  const theme = useContext(ThemeContext);
  return <p>{theme}</p>;
}
```

When the provider value changes, consumers render again. Do not put a fast-changing value in context if most of the tree does not need it.

A custom hook is a function whose name starts with `use` and that may call other hooks.

```tsx
function useToggle(start = false) {
  const [on, setOn] = useState(start);
  function toggle() {
    setOn((value) => !value);
  }
  return [on, toggle] as const;
}
```

## 9. Rules and structure

| Rule | Why |
| --- | --- |
| Call hooks at the top level | Not inside a condition or a loop. React matches hooks by call order |
| Call hooks only from components and custom hooks | A random function cannot call `useState` |
| State is a snapshot | The `count` in this render does not change later in this render |
| Keys must be stable | Index keys scramble state |
| Side effects do not belong in render | Render should be safe to run twice in development |
| One component, one job | A file can hold a small private child component |

Development mode renders twice on purpose to surface impure renders. A fetch in the component body would fire twice. A fetch in an effect is cleaned up between those runs.

## 10. Error boundaries

An error boundary catches a render error in a child and shows a fallback instead of blanking the page. The boundary itself is a class. Function components and hooks do not have this method. This is the one class worth writing in an otherwise function-component app.

```tsx
import { Component, type ReactNode } from "react";

type BoundaryState = { error: Error | null };

class ErrorBoundary extends Component<{ children: ReactNode }, BoundaryState> {
  state: BoundaryState = { error: null };

  static getDerivedStateFromError(error: Error): BoundaryState {
    return { error };
  }

  render() {
    if (this.state.error) {
      return <p>Something went wrong.</p>;
    }
    return this.props.children;
  }
}
```

```tsx
<ErrorBoundary>
  <PriceChart city={city} />
</ErrorBoundary>
```

| Error happens in | The boundary catches it |
| --- | --- |
| A child's render | Yes |
| A child's constructor | Yes |
| An event handler | No. Use `try/catch` in the handler |
| An async call or a promise | No. Catch it where you `await` |
| The boundary's own render | No. A parent boundary can catch that |

`try/catch` around JSX does not catch a child. The child renders later, on its own. `componentDidCatch` can log the error. `getDerivedStateFromError` is what switches the tree to the fallback.

## Drill

1. Write `Card` with `city` and `price` props.
2. Write a controlled city input.
3. Write an effect that starts an interval and clears it.
4. Name one error an error boundary catches, and one it does not.
4. Say why the key should be an id.
