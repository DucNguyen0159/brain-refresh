# Next.js

Course 6 of 15. Previous: [05-react.md](05-react.md). Next: [07-node-express.md](07-node-express.md).

Next.js is a React framework. It turns files into routes, can render HTML on the server, and can host the result on a platform such as Vercel. This course is the App Router, which uses the `app` directory. Every section has the same parts: the concept, the terms, the methods, several cases with what the compiler or the browser does, common mistakes, a tip, and a practice with a solution.

## Contents

1. [What a route is](#1-what-a-route-is)
2. [Layouts, loading, and errors](#2-layouts-loading-and-errors)
3. [Server and client components](#3-server-and-client-components)
4. [Navigation and dynamic routes](#4-navigation-and-dynamic-routes)
5. [Data, mutations, and metadata](#5-data-mutations-and-metadata)
6. [Images, fonts, and environment](#6-images-fonts-and-environment)
7. [Cookies, headers, and middleware](#7-cookies-headers-and-middleware)

## 1. What a route is

### Concept

A folder under `app` is a URL segment. `page.tsx` is the UI for that URL. The default export of that file is the page. Named exports from that file are not the page, even when they are special functions such as `generateMetadata`. A route group folder whose name is wrapped in parentheses, such as `(marketing)`, organizes files and does not appear in the URL. `route.ts` is an HTTP handler instead of a page. Do not put `page.tsx` beside it in the same folder.

### Terms

| Term | Meaning |
| --- | --- |
| Segment | One folder under `app`. It becomes one piece of the URL, unless it is a route group |
| Page | The `page.tsx` file. Its default export is what the URL renders |
| Route group | A folder named `(marketing)`. It does not add a URL segment |
| Dynamic segment | A folder named `[city]` or `[id]`. The text in the URL becomes a parameter |
| Special file | `layout.tsx`, `template.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, or `route.ts` |

### Methods

| File | URL or role |
| --- | --- |
| `app/page.tsx` | `/` |
| `app/search/page.tsx` | `/search` |
| `app/reports/[city]/page.tsx` | `/reports/irvine` and any other city |
| `app/homes/[id]/page.tsx` | `/homes/42` and any other id |
| `app/(marketing)/about/page.tsx` | `/about`. The `(marketing)` group does not appear in the URL |
| `layout.tsx` | Wraps child pages. Stays mounted across navigations inside it |
| `template.tsx` | Like a layout, but remounts on navigation |
| `loading.tsx` | Fallback while the page's async work is pending |
| `error.tsx` | Error boundary for this segment. Must be a client component |
| `not-found.tsx` | Rendered when `notFound()` is called |
| `route.ts` | An HTTP handler instead of a page. No `page.tsx` beside it in the same folder |

### Examples

**Case: the home page is the default export**

```tsx
export default function HomePage() {
  return <h1>Homes</h1>;
}
```

Result: a request for `/` renders `Homes`. The file is `app/page.tsx`.

Why: the folder `app` is the root segment, and `page.tsx` is the UI for that URL. A named export in the same file is not a second page.

**Case: a folder becomes a segment**

```text
app/search/page.tsx
```

Result: the browser URL `/search` renders that file. `/search/extra` does not, unless another folder exists.

Why: each folder under `app` is one URL segment. The file name `page.tsx` marks the segment that has a UI. `loading.tsx` in that folder is not a URL.

**Case: a route group is not a segment**

```text
app/(marketing)/about/page.tsx
```

Result: the URL is `/about`. The word `marketing` is not in the path.

Why: parentheses mark a group for organization, such as a shared layout that only those pages use. The group name is skipped when Next builds the URL.

**Case: brackets are a parameter, and `route.ts` replaces the page**

```text
app/reports/[city]/page.tsx
app/homes/[id]/page.tsx
app/health/route.ts
```

Result: `/reports/irvine` and `/homes/42` render the matching `page.tsx`. `/health` runs `route.ts` and does not look for a page component. Putting `page.tsx` in `app/health` beside `route.ts` is an invalid segment.

Why: `[city]` and `[id]` match one path piece and pass it to the page as `params`. `route.ts` exports HTTP functions such as `GET` instead of a React page. One folder has one of those roles, not both.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Naming the page `Search.tsx` instead of `page.tsx` | The folder is not a route | The UI file must be named `page.tsx` |
| Expecting `(marketing)` to show up in the URL | The link `/marketing/about` 404s | Link to `/about` |
| Adding `page.tsx` next to `route.ts` | The segment is invalid | Keep one of the two files |
| Treating a named export as the page | The URL renders the default export only | `export default function Page` |

### Tip

If the URL is wrong, read the folders from `app` downward and skip any name in parentheses.

### Practice

Which file is the URL `/search`, and which URL does `app/(marketing)/about/page.tsx` serve?

### Solution

`/search` is `app/search/page.tsx`. `app/(marketing)/about/page.tsx` is `/about`.

```text
app/search/page.tsx          -> /search
app/(marketing)/about/page.tsx -> /about
```

Result: `/search` renders the default export of `app/search/page.tsx`. The group name is not part of `/about`.

## 2. Layouts, loading, and errors

### Concept

The root layout wraps every page and is the only layout that holds `<html>` and `<body>`. `children` is the matched page, or a nested layout. A layout stays mounted when you move between pages inside it, so its state survives. A template looks like a layout and remounts on each navigation. `loading.tsx` is shown immediately and replaced when the page's async work resolves. `error.tsx` receives `error` and `reset`. It must be a client component. `reset` tries to render the segment again.

### Terms

| Term | Meaning |
| --- | --- |
| Root layout | `app/layout.tsx`. Required. Owns `<html>` and `<body>` |
| Nested layout | `app/search/layout.tsx` wraps only `/search` and below |
| `children` | The matched page, or the next layout down |
| `loading.tsx` | Instant fallback. Next wraps the segment in a suspense boundary |
| `error.tsx` | Error boundary for this segment. `"use client"` is required |
| `reset` | A function that tries to render the segment again |

### Methods

| Piece | Behavior |
| --- | --- |
| `children` | The matched page, or a nested layout |
| `loading.tsx` | Shown immediately, replaced when the page resolves |
| `error.tsx` | Receives `error` and `reset`. `reset` tries to render the segment again |
| Nested layouts | A layout in `app/search/layout.tsx` wraps only `/search` and below |
| `template.tsx` | Same props as a layout, and a new instance on each navigation |
| `global-error.tsx` | Replaces the root layout when the root itself throws. It must include `<html>` and `<body>` |

### Examples

**Case: the root layout owns the document**

```tsx
import type { ReactNode } from "react";

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

Result: every page is rendered inside this `<body>`. The browser receives one `<html>` document.

Why: the root layout wraps every route. A nested layout must not render `<html>` or `<body>` again. `children` is the matched page or the next layout.

**Case: a nested layout does not wrap the rest of the app**

```tsx
import type { ReactNode } from "react";

export default function SearchLayout({ children }: { children: ReactNode }) {
  return (
    <section>
      <h1>Search</h1>
      {children}
    </section>
  );
}
```

Result: `/search` and `/search/maps` show the `Search` heading. `/` does not. Navigating between those search URLs keeps this layout mounted.

Why: the file is `app/search/layout.tsx`, so it wraps only that segment and below. Because it does not remount, state in the layout survives those navigations. Fetch data that changes on every navigation in the page, not in the layout.

**Case: `loading.tsx` is the immediate fallback**

```tsx
export default function Loading() {
  return <p>Loading report</p>;
}
```

Result: the browser shows `Loading report` at once, then replaces it when the page's async work finishes.

Why: `loading.tsx` is a suspense fallback for the segment. It does not wait for the page's `await` to finish before the first UI. The page replaces it. It is not an error screen.

**Case: `error.tsx` must be a client component**

```tsx
"use client";

export default function SegmentError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <p>{error.message}</p>
      <button type="button" onClick={() => reset()}>
        Try again
      </button>
    </div>
  );
}
```

Result: if the page in this segment throws, Next renders this UI instead of the page. The button calls `reset`, which tries to render the segment again.

Why: an error boundary uses client rendering, so the file starts with `"use client"`. This file catches the page and components under it. It does not catch the layout in the same folder. A throw in that layout goes to the parent error boundary. A throw in the root layout needs `global-error.tsx`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A second `<html>` in a nested layout | Invalid document structure | Only the root layout renders `<html>` and `<body>` |
| Fetching navigation-specific data in the layout | The layout does not remount, so that work is easy to get wrong and its state survives | Fetch that data in the page |
| `error.tsx` without `"use client"` | The build fails. An error boundary is a client component | Put `"use client"` at the top of the file |
| Expecting `error.tsx` to catch the layout beside it | A throw in that layout skips this file | Put the boundary in the parent segment, or use `global-error.tsx` for the root |

### Tip

Use a layout for UI that should stay put, and a template when the wrapper must remount and reset its state on every navigation.

### Practice

Write a root layout that renders `{children}` inside `<body>`, and name the two props `error.tsx` receives.

### Solution

```tsx
import type { ReactNode } from "react";

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

Result: every route renders inside that body. `error.tsx` receives `error` and `reset`. `reset` tries to render the segment again, and the file must start with `"use client"`.

## 3. Server and client components

### Concept

By default, a component in the App Router is a server component. It runs on the server, can `await` data, and does not ship its own code as a client bundle. It cannot use `useState`, `useEffect`, or browser events. A file that starts with `"use client"` is a client component. It can use state and events. It is still pre-rendered to HTML, then hydrated in the browser. `"use client"` is a boundary, not a label on one function. Every module that file imports is client code too.

### Terms

| Term | Meaning |
| --- | --- |
| Server component | The default. Runs on the server. Can `await`. No `useState` or click handlers |
| Client component | A file whose first statement is `"use client"` |
| Boundary | The `"use client"` file and every module it imports ship to the browser |
| Hydration | The browser attaches event handlers to the HTML the server already sent |
| `children` slot | A way to pass a server-rendered node into a client component without importing it there |

### Methods

| Need | Where |
| --- | --- |
| Read a database or a server-only value | Server component or a route handler |
| `useState`, clicks, effects | Client component |
| A client child inside a server page | Allowed. Import the client component into the server page |
| A server component imported into a client file | It becomes part of the client graph. Pass server-rendered nodes as `children` instead |
| Browser-only API such as `window` | Client component, and only after the component has mounted if the server render must not touch it |

### Examples

**Case: a server page awaits, and it has no event handler**

```tsx
export default async function ReportPage() {
  const city = "Irvine";
  return <h1>{city}</h1>;
}
```

Result: the server sends HTML with `Irvine`. This file's function is not bundled as a client component.

Why: there is no `"use client"`, so the page is a server component. It may be `async` and `await` a database call. Adding `useState` or `onClick` here fails, because those need the browser.

**Case: a click handler needs `"use client"`**

```tsx
"use client";

import { useState } from "react";

export function CityField() {
  const [city, setCity] = useState("");
  return <input value={city} onChange={(event) => setCity(event.target.value)} />;
}
```

Result: the input renders on the server as HTML, then hydrates. Typing updates `city` in the browser.

Why: `useState` and the change event run on the client. `"use client"` marks the whole module. It is still pre-rendered, so the first HTML is not empty. The directive is not a label you can put on one function inside an otherwise server file.

**Case: a server page may render a client child**

```tsx
import { CityField } from "./city-field";

export default function Page() {
  return (
    <main>
      <h1>Search</h1>
      <CityField />
    </main>
  );
}
```

Result: the heading is server HTML. `CityField` hydrates, and the input accepts typing. The page file itself stays a server component.

Why: importing a client component into a server page is allowed. The boundary is the client file, not the parent. The server page can still `await` data and pass plain props, such as a string, into `CityField`.

**Case: importing a server component into a client file pulls it across**

```tsx
"use client";

import type { ReactNode } from "react";

export function Frame({ children }: { children: ReactNode }) {
  return <section>{children}</section>;
}
```

```tsx
import { Frame } from "./frame";
import { Report } from "./report";

export default function Page() {
  return (
    <Frame>
      <Report />
    </Frame>
  );
}
```

Result: `Report` renders on the server and is passed into `Frame` as `children`. It is not added to the client bundle by this page.

Why: `children` is already a rendered node. If `frame.tsx` imported `Report` itself, `Report` would become part of the client graph and could no longer stay a server component. Pass server-rendered nodes as `children` instead of importing them inside the client file.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `useState` or `onClick` in a file with no `"use client"` | The compiler rejects the hook or the event in a server component | Start that file with `"use client"` |
| Putting `"use client"` on one function in a server file | The directive applies to the module, and a misplaced directive does not make one function client-only | Move the interactive component into its own file |
| Importing a server component from a client file | That component joins the client bundle and loses server-only privileges | Pass it as `children` from a server parent |
| Passing a server-only string into a client component as a prop | That string is sent to the browser in the payload | Read it on the server and pass only values the browser should see |
| Passing a plain function that is not a Server Action | The compiler rejects the prop | Pass a Server Action, or keep the handler in the client file |

### Tip

Start on the server. Add `"use client"` only on the leaf that needs state or an event.

### Practice

Say why a file with a click handler needs `"use client"`, and what the browser still receives before hydration.

### Solution

The handler uses a browser event, and it usually needs `useState`. A server component cannot use either. The file starts with `"use client"`, so the component and its imports are in the client graph.

```tsx
"use client";

import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

Result: the button is in the server HTML, then hydration makes the click work. Without the directive, the file is a server component and the event is rejected.

## 4. Navigation and dynamic routes

### Concept

`Link` from `next/link` navigates inside the app without a full browser reload. In production it prefetches the route when the link is in view. An ordinary `<a>` does a full load and discards JavaScript state. Use `Link` for routes in your app. A dynamic segment such as `[id]` or `[city]` arrives on the page as `params`. In current Next, the page receives `params` and `searchParams` as promises, and you `await` them. `notFound()` renders the closest `not-found.tsx`. `redirect("/login")` sends the browser to another URL. `generateStaticParams` lists which dynamic values to prebuild. Omit it and the route can still render on demand.

### Terms

| Term | Meaning |
| --- | --- |
| `Link` | Client navigation to a route in this app. Prefetches in production |
| `<a>` | A full document load |
| `params` | Dynamic segments. A promise of an object such as `{ id: string }` |
| `searchParams` | The query string. Also a promise in current page props |
| `generateStaticParams` | The list of dynamic values to prebuild at build time |

### Methods

| API | Use |
| --- | --- |
| `params` | Dynamic segments. In current Next, the page receives it as a promise and you `await` it |
| `searchParams` | The query string. Also a promise in current page props |
| `notFound()` | Render the closest `not-found.tsx` |
| `redirect("/login")` | Send the browser to another URL. Import it from `next/navigation` |
| `useRouter` | Client navigation: `push`, `replace`, `back`. Requires a client component. Import it from `next/navigation` |
| `usePathname` | The current path, in a client component |
| `generateStaticParams` | Returns `{ id: string }[]` (or whichever segment you named) to prebuild |

### Examples

**Case: `Link` stays in the app**

```tsx
import Link from "next/link";

export function Nav() {
  return <Link href="/search">Search</Link>;
}
```

Result: clicking Search goes to `/search` without a full browser reload. In production, Next prefetches that route when the link is in view.

Why: `Link` is the App Router navigation component. An ordinary `<a href="/search">` loads the document from scratch and drops client state. Use `Link` for routes in your app. Use `<a>` for a site on another origin.

**Case: `[id]` is a promise**

```tsx
export default async function HomePage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  return <h1>{id}</h1>;
}
```

Result: `/homes/42`, from `app/homes/[id]/page.tsx`, renders `42`.

Why: in current Next the page receives `params` as a promise. `await` it before you read `id`. The value is a string, even when it looks like a number. The same pattern is how `app/reports/[city]/page.tsx` reads `city`.

**Case: the query string is `searchParams`**

```tsx
export default async function SearchPage({
  searchParams,
}: {
  searchParams: Promise<{ city?: string }>;
}) {
  const { city } = await searchParams;
  return <p>{city ?? "all"}</p>;
}
```

Result: `/search?city=Irvine` renders `Irvine`. `/search` renders `all`.

Why: `searchParams` is the query string, and in current page props it is a promise too. Reading it opts the page into rendering with the request, because the query is not known at build time.

**Case: missing data, a redirect, and a client router**

```tsx
import { notFound, redirect } from "next/navigation";

export default async function ReportPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  if (id === "0") notFound();
  if (id === "login") redirect("/login");
  return <h1>{id}</h1>;
}
```

```tsx
"use client";

import { usePathname, useRouter } from "next/navigation";

export function Back() {
  const router = useRouter();
  const path = usePathname();
  return (
    <button type="button" onClick={() => router.back()}>
      Back from {path}
    </button>
  );
}
```

Result: `id` of `0` renders the closest `not-found.tsx`. `id` of `login` sends the browser to `/login`. The button runs in the browser: `usePathname` shows the current path, and `router.back()` goes back. `push` and `replace` are the other `useRouter` navigations.

Why: `notFound()` and `redirect()` are server control flow. They throw a signal Next catches, so do not swallow them in an empty `catch`. `useRouter` and `usePathname` need a client component. Import them from `next/navigation`, not from the older pages-router `next/router`. `generateStaticParams` lists the dynamic values to prebuild, such as `{ id: "42" }`. Omit that export and the `[id]` route can still render on demand when a request arrives.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Reading `params.id` without `await` | The value is a promise, so the UI prints nothing useful or the types fail | `const { id } = await params` |
| Using `<a>` for an in-app route | Full page load. Client state is discarded | `Link` from `next/link` |
| Calling `useRouter` in a server component | The render fails. The hook is client-only | A `"use client"` file, imported from `next/navigation` |
| Wrapping `redirect()` in a `catch` that ignores the error | The redirect never happens | Let `redirect()` and `notFound()` leave the function |

### Tip

`generateStaticParams` prebuilds the values you list. A `[id]` route without that export can still render on demand when a request arrives.

### Practice

Render `Link` to `/search`, and read `id` from `app/homes/[id]/page.tsx`.

### Solution

```tsx
import Link from "next/link";

export function Nav() {
  return <Link href="/search">Search</Link>;
}

export default async function HomePage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  return <h1>{id}</h1>;
}
```

Result: the link navigates to `/search` without a full reload. `/homes/42` renders `42` after `await params`.

## 5. Data, mutations, and metadata

### Concept

A server component may `await` a function that reads your database. That function runs on the server while the page renders. Do not import a server-only value into a client component. A variable is public in the browser when its name starts with `NEXT_PUBLIC_`. Current Next does not cache `fetch` by default. A static result can be reused across requests. A dynamic result depends on the request, such as cookies or a search param. `revalidate` is the number of seconds before a cached result may be refreshed. `noStore`, or an uncached fetch, reads fresh data. Server Actions are async functions that run on the server when a form submits or when you call them. Mark the file or the function with `"use server"`. Validate the input on the server anyway.

### Terms

| Caching idea | Meaning |
| --- | --- |
| Static | The result can be reused across requests. `fetch(url, { cache: "force-cache" })` asks for that |
| Dynamic | The result depends on the request, such as cookies or a search param |
| `revalidate` | Seconds before the cached result may be refreshed. `next: { revalidate: 60 }` on `fetch`, or `export const revalidate = 60` |
| `noStore` / uncached fetch | Read fresh data. Current Next leaves `fetch` uncached unless you opt in. `cache: "no-store"` says so on one call |

| | Server component | Client `fetch` | Server Action |
| --- | --- | --- | --- |
| Runs | On the server, while the page renders | In the browser, usually inside an effect | On the server, when the client calls it |
| Starts | As the page is built | After the page is on screen | On submit, or when you call the function |
| Can read a server-only value | Yes | No | Yes |
| Can use `useState` | No | Yes | The function itself cannot. A client component may call it |
| Shape | `const rows = await getSales(city)` | `fetch("/api/sales")` then `setRows` | `"use server"` and a form `action` or a direct call |
| When it is the wrong tool | The UI needs a click handler | The data is server-only, or the page should arrive with the data | You only needed to read, and a server component can `await` it |

### Methods

| Tool | Role |
| --- | --- |
| `await getSales(city)` | Read on the server during render |
| `fetch(url, { cache: "force-cache" })` | Reuse the response across requests |
| `fetch(url, { next: { revalidate: 60 } })` | Reuse, then allow a refresh after 60 seconds |
| `fetch(url, { cache: "no-store" })` | Read fresh data for this request |
| `export async function GET()` in `route.ts` | An HTTP handler. `return Response.json({ status: "ok" })` |
| `"use server"` | Marks a file, or one function, as a Server Action. The docs also call these Server Functions |
| `export const metadata` | Title and description known at build time |
| `generateMetadata` | The async version when the title depends on `params` |
| Title template | In the root layout: `{ default: "Homes", template: "%s · Homes" }` |

### Examples

**Case: the page awaits the read**

```tsx
export default async function ReportPage() {
  const rows = await getSales("Irvine");
  return (
    <ul>
      {rows.map((row) => (
        <li key={row.id}>{row.total}</li>
      ))}
    </ul>
  );
}
```

Result: the HTML arrives with one list item per row. The browser does not run `getSales`.

Why: the server component awaits before it returns elements. That is the right tool when the page should arrive with the data. It is the wrong tool when the UI needs a click handler.

**Case: a client fetch starts after paint**

```tsx
"use client";

import { useEffect, useState } from "react";

export function Sales() {
  const [total, setTotal] = useState<string>("Loading");

  useEffect(() => {
    fetch("/api/sales")
      .then((response) => response.json())
      .then((body: { total: string }) => setTotal(body.total));
  }, []);

  return <p>{total}</p>;
}
```

Result: the first paint says `Loading`. After the browser finishes `fetch("/api/sales")`, the paragraph shows the total.

Why: a client fetch starts after the page is on screen, usually inside an effect. It can then `setRows` or `setTotal`. It cannot read a server-only value. It only sees what the route returns. If the page should arrive with the data, await it in a server component instead.

**Case: a route handler and a Server Action**

```tsx
export async function GET() {
  return Response.json({ status: "ok" });
}
```

```tsx
"use server";

export async function saveCity(formData: FormData) {
  const city = String(formData.get("city") ?? "").trim();
  if (city === "") return { error: "city is required" };
  await saveCityRow(city);
  return { ok: true };
}
```

```tsx
import { saveCity } from "./actions";

export function CityForm() {
  return (
    <form action={saveCity}>
      <input name="city" />
      <button type="submit">Save</button>
    </form>
  );
}
```

Result: `GET` on that `route.ts` segment returns JSON `{ "status": "ok" }`. Submitting the form runs `saveCity` on the server. An empty city returns `{ error: "city is required" }` and does not write a row.

Why: route handlers live in `route.ts` and export functions named for HTTP methods. A Server Action is a way to change data without writing a separate API route. `"use server"` marks the file or the function. You can also call the function directly from a client component. Validate on the server, because the client can send any field. A file that starts with `"use server"` should export async functions only.

**Case: metadata and a title that depends on `params`**

```tsx
export const metadata = {
  title: "Irvine report",
  description: "Twelve-month sales summary.",
};
```

```tsx
export async function generateMetadata({
  params,
}: {
  params: Promise<{ city: string }>;
}) {
  const { city } = await params;
  return { title: `${city} report` };
}
```

Result: the static page's document title is `Irvine report`. A dynamic city page sets its own title from `params`. With a root template of `{ default: "Homes", template: "%s · Homes" }`, a child title `Irvine report` is shown as `Irvine report · Homes`, and a page with no title uses `Homes`.

Why: `metadata` is for values you know without the request. `generateMetadata` is the async version when the title depends on `params`. `await` that promise the same way the page does.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Importing a server-only module into a client component | That module joins the client graph. A `NEXT_PUBLIC_` value is inlined. A server-only `process.env` name is `undefined` there, and a string you pass as a prop is sent | Keep the read in a server component, a Server Action, or `route.ts` |
| Using a client `fetch` for data the first paint needs | The page arrives empty and then pops in | `await` the read in the server component |
| A Server Action that trusts the form | The client can post any string | Validate on the server, then write |
| `export const metadata` when the title depends on `params` | The title cannot see the dynamic segment | `generateMetadata` and `await params` |

### Tip

Cache on purpose. `force-cache` or `revalidate` reuses a result. Cookies, search params, `noStore`, and `cache: "no-store"` are how a result stays tied to one request.

### Practice

Show a `GET` route handler that returns `{ status: "ok" }`, and a metadata object with a title and a description.

### Solution

```tsx
export async function GET() {
  return Response.json({ status: "ok" });
}

export const metadata = {
  title: "Irvine report",
  description: "Twelve-month sales summary.",
};
```

Result: the route responds with JSON `{ "status": "ok" }`. The document title is `Irvine report`. A title that depends on `params` belongs in `generateMetadata` instead.

## 6. Images, fonts, and environment

### Concept

`next/image` requires `width` and `height`, or `fill` inside a positioned parent. It reserves space and can serve a resized file. Remote image hosts must be listed in `next.config`. Local files in `public/` are served from `/`. `next/font` loads a font without a layout jump. You apply the class it returns on `body`. `DATABASE_URL` is not visible in the browser if you only read it on the server. `NEXT_PUBLIC_MAP_KEY` is visible. Anything that may be public uses the `NEXT_PUBLIC_` prefix. A push to a host such as Vercel can build and publish the app. That publish step is not a test suite. Tests run where you configured them.

### Terms

| Variable | Visible in the browser? |
| --- | --- |
| `DATABASE_URL` | No, if you only read it on the server |
| `NEXT_PUBLIC_MAP_KEY` | Yes. The value is inlined into the client bundle |
| Any `NEXT_PUBLIC_` name | Yes. Anything that may be public |
| A name without that prefix | Server-only, as long as a client module does not import it |

### Methods

| Tool | Use |
| --- | --- |
| `<Image width={800} height={600} />` | Reserves a box and can serve a resized file |
| `<Image fill />` | Fills a parent that is `relative`, `absolute`, or `fixed` |
| `images.remotePatterns` in `next.config` | Allows a remote host |
| `public/house.jpg` | Served as `/house.jpg` |
| `next/font/google` | A font with a class that avoids a layout jump |
| `body` `className` | Where the font class belongs, usually in the root layout |

### Examples

**Case: width and height reserve the box**

```tsx
import Image from "next/image";

export function Photo() {
  return <Image src="/house.jpg" alt="Front door" width={800} height={600} />;
}
```

Result: the browser lays out an 800 by 600 box and shows `public/house.jpg` at the URL `/house.jpg`. The file can be served resized.

Why: `next/image` needs the size so the layout does not jump when the file arrives. `alt` is the accessible name. The `public` folder name is not part of the URL.

**Case: `fill` needs a positioned parent**

```tsx
import Image from "next/image";

export function Cover() {
  return (
    <div style={{ position: "relative", width: "100%", height: 320 }}>
      <Image src="/house.jpg" alt="Front door" fill />
    </div>
  );
}
```

Result: the image fills the 320-pixel-tall box. The box keeps its height while the file loads.

Why: `fill` omits `width` and `height` on the image. The parent must be positioned (`relative`, `absolute`, or `fixed`) so the image is anchored to that box.

**Case: a remote host is listed, or the image is refused**

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "images.example.com" },
    ],
  },
};

export default nextConfig;
```

Result: `src="https://images.example.com/house.jpg"` is allowed. A hostname that is not in `images.remotePatterns` makes the server throw when that image renders.

Why: remote hosts must be listed in `next.config`. Local files in `public/` do not need an entry. The config is the allow-list, not a rewrite of the URL.

**Case: the font class goes on `body`, and only `NEXT_PUBLIC_` is inlined**

```tsx
import type { ReactNode } from "react";
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"] });

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

```tsx
const mapKey = process.env.NEXT_PUBLIC_MAP_KEY;
const databaseUrl = process.env.DATABASE_URL;
```

Result: the font loads without a layout jump, because the class from `next/font` is on `body`. `NEXT_PUBLIC_MAP_KEY` is visible in the browser bundle. `DATABASE_URL` is not, if this read stays in a server module.

Why: `next/font` self-hosts the file and reserves the metrics so text does not jump. A name that starts with `NEXT_PUBLIC_` is inlined into the browser bundle on purpose. Referencing `DATABASE_URL` from a client module does not inline the value; Next leaves it `undefined` there. Passing that value as a prop from a server component does send it to the browser, so read it only on the server. Publishing on a host such as Vercel builds and ships the app. That step is not a test suite. Tests run where you configured them.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `Image` with neither size nor `fill` | The compiler requires `width` and `height`, or `fill` | Pass both dimensions, or `fill` in a positioned parent |
| A remote `src` whose host is not configured | The server throws while rendering the image | Add the host to `images.remotePatterns` |
| Putting a database URL in `NEXT_PUBLIC_DATABASE_URL` | The value is inlined into the browser bundle | Keep the server-only name, with no `NEXT_PUBLIC_` prefix |
| Treating a production deploy as the test run | The app publishes even when your tests never ran | Run tests in the step you configured for them |

### Tip

If the browser can see a value, the name was prefixed with `NEXT_PUBLIC_` or a client module read it. Keep `DATABASE_URL` on the server.

### Practice

Which environment name would leak into the browser, and which `Image` props reserve an 800 by 600 box for `/house.jpg`?

### Solution

`NEXT_PUBLIC_MAP_KEY` is visible in the browser. `DATABASE_URL` is not, if you only read it on the server.

```tsx
import Image from "next/image";

export function Photo() {
  return <Image src="/house.jpg" alt="Front door" width={800} height={600} />;
}
```

Result: the client bundle can contain `NEXT_PUBLIC_MAP_KEY`. The image reserves an 800 by 600 box for `/house.jpg`.

## 7. Cookies, headers, and middleware

### Concept

A server component can read the incoming request. In current Next, `cookies()` and `headers()` are async. Reading is allowed while the page renders. Setting a cookie is not. `cookieStore.set` belongs in a Server Action or a route handler. Calling it during render throws. `middleware.ts` sits next to the `app` directory, or inside `src` when the project uses `src`. It runs before the route renders. Current Next.js docs rename this file to `proxy.ts` and rename the exported function to `proxy`. The `matcher` config is the same idea. This section keeps `middleware.ts`, which is the name many projects still use. `matcher` limits which paths run the function. A middleware that matches every page runs on static files too unless you exclude them.

### Terms

| Term | Meaning |
| --- | --- |
| `cookies()` | Async. The cookie jar for this request, from `next/headers` |
| `headers()` | Async. The incoming headers, from `next/headers` |
| Middleware | `middleware.ts` beside `app` or inside `src`. Runs before the matched route |
| `matcher` | `export const config = { matcher: [...] }`. Which paths invoke the function |
| `NextResponse.next()` | Continue to the route |
| `NextResponse.redirect` | Send the browser elsewhere before the page renders |

### Methods

| | Server component | Middleware |
| --- | --- | --- |
| Cookies | `await cookies()` from `next/headers` | `request.cookies` |
| Headers | `await headers()` from `next/headers` | `request.headers` |
| When | While that page renders | Before the matched route renders |
| Redirect | `redirect("/login")` from `next/navigation` | `NextResponse.redirect` |
| Set a cookie | In a Server Action or a route handler | On the `NextResponse` you return |

### Examples

**Case: the page awaits cookies and headers**

```tsx
import { cookies, headers } from "next/headers";

export default async function Page() {
  const cookieStore = await cookies();
  const headerStore = await headers();
  const theme = cookieStore.get("theme")?.value ?? "light";
  const host = headerStore.get("host");
  return (
    <p>
      {theme} on {host}
    </p>
  );
}
```

Result: a request with cookie `theme=dark` and host `localhost:3000` renders `dark on localhost:3000`. With no theme cookie, the text starts with `light`.

Why: in current Next both functions are async, so you `await` them. `get` returns the cookie object, and `value` is the string. Reading during render is allowed. Using either function ties the route to the request.

**Case: setting a cookie during render throws**

```tsx
import { cookies } from "next/headers";

export default async function Page() {
  const cookieStore = await cookies();
  cookieStore.set("theme", "dark");
  return <p>Theme saved</p>;
}
```

Result: rendering this page throws. The cookie is not set.

Why: the server component is building a response body, and the headers may already be streaming. `cookieStore.set` belongs in a Server Action or a route handler, where the response headers can still be written. Middleware can set a cookie on the `NextResponse` it returns.

**Case: middleware reads the cookie on the request**

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.cookies.get("session")?.value;
  if (!token && request.nextUrl.pathname.startsWith("/reports")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ["/reports/:path*"],
};
```

Result: a request to `/reports` or `/reports/irvine` with no `session` cookie is redirected to `/login` before the page renders. A request with that cookie continues. `/search` does not run this function.

Why: the file is `middleware.ts` next to `app`, or inside `src`. It sees `request.cookies`, not `await cookies()`. `matcher` limits the paths. `NextResponse.redirect` is the middleware redirect. A page uses `redirect()` from `next/navigation` instead.

**Case: no matcher means static files too**

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(_request: NextRequest) {
  return NextResponse.next();
}
```

Result: this function runs for pages and, because nothing excludes them, for static files as well.

Why: `matcher` is the opt-in list. A middleware that matches every page runs on static files too unless you exclude them, for example `_next/static` and files with an extension. Keep the matcher as narrow as the gate you mean, such as `["/reports/:path*"]`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Calling `cookies()` or `headers()` without `await` | You read a promise, not the jar or the headers | `const cookieStore = await cookies()` |
| `cookieStore.set` in a server component render | The render throws | Set the cookie in a Server Action or a route handler |
| Reading `await cookies()` inside `middleware.ts` | Middleware uses the request object, and `next/headers` is for the rendering functions | `request.cookies` and `request.headers` |
| A middleware file with no `matcher` | It also runs for static assets | `export const config = { matcher: [...] }` and exclude static paths |

### Tip

The page asks `next/headers` while it renders. Middleware asks the `NextRequest` before that render starts.

### Practice

Say where a page reads a cookie, and where middleware reads that same cookie.

### Solution

The page reads during render:

```tsx
import { cookies } from "next/headers";

export default async function Page() {
  const cookieStore = await cookies();
  const theme = cookieStore.get("theme")?.value ?? "light";
  return <p>{theme}</p>;
}
```

Middleware reads the request, before the route renders:

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const theme = request.cookies.get("theme")?.value ?? "light";
  const response = NextResponse.next();
  response.headers.set("x-theme", theme);
  return response;
}
```

Result: the page uses `await cookies()` from `next/headers`. Middleware uses `request.cookies` and returns `NextResponse.next()`. Setting the cookie during page render throws. Set it in a Server Action, a route handler, or on the `NextResponse` from middleware.

## Drill

1. Say which file is the URL `/search`.
2. Say why a click handler file needs `"use client"`.
3. Say which env name would leak into the browser.
4. Say where a page reads a cookie, and where middleware reads that same cookie.

### Solution 1

The practice solution in section 1. The file is `app/search/page.tsx`.

Result: `/search` renders the default export of `app/search/page.tsx`. `app/(marketing)/about/page.tsx` is `/about`, because the group is not a segment.

### Solution 2

The practice solution in section 3. A click handler uses a browser event and usually `useState`. A file without `"use client"` is a server component, and those APIs are rejected there.

Result: the client file is still pre-rendered to HTML, then hydrated so the click runs in the browser.

### Solution 3

The practice solution in section 6. A name that starts with `NEXT_PUBLIC_` is inlined into the browser bundle. `NEXT_PUBLIC_MAP_KEY` would leak. `DATABASE_URL` stays on the server if you only read it there.

Result: the browser can see `NEXT_PUBLIC_MAP_KEY`. It cannot see `DATABASE_URL` from a server-only read.

### Solution 4

The practice solution in section 7. The page reads with `await cookies()` from `next/headers`, then `cookieStore.get`. Middleware reads `request.cookies` on the `NextRequest`, and `matcher` decides which paths run that function.

Result: page render uses `await cookies()` and `await headers()`. Middleware uses `request.cookies` and `request.headers` before the route renders. `cookieStore.set` during render throws.
