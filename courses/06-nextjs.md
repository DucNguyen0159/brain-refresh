# Next.js

Course 6 of 15. Previous: [05-react.md](05-react.md). Next: [07-node-express.md](07-node-express.md).

Next.js is a React framework. It turns files into routes, can render HTML on the server, and can host the result on a platform such as Vercel. This course is the App Router, which uses the `app` directory.

## Contents

1. [What a route is](#1-what-a-route-is)
2. [Layouts, loading, and errors](#2-layouts-loading-and-errors)
3. [Server and client components](#3-server-and-client-components)
4. [Navigation and dynamic routes](#4-navigation-and-dynamic-routes)
5. [Data, mutations, and metadata](#5-data-mutations-and-metadata)
6. [Images, fonts, and environment](#6-images-fonts-and-environment)
7. [Cookies, headers, and middleware](#7-cookies-headers-and-middleware)

## 1. What a route is

A folder under `app` is a URL segment. `page.tsx` is the UI for that URL.

| File | URL |
| --- | --- |
| `app/page.tsx` | `/` |
| `app/search/page.tsx` | `/search` |
| `app/reports/[city]/page.tsx` | `/reports/irvine` and any other city |
| `app/(marketing)/about/page.tsx` | `/about`. The `(marketing)` group does not appear in the URL |

```tsx
export default function HomePage() {
  return <h1>Homes</h1>;
}
```

The default export of `page.tsx` is the page. Named exports from that file are not the page.

| Also special | Role |
| --- | --- |
| `layout.tsx` | Wraps child pages. Stays mounted across navigations inside it |
| `template.tsx` | Like a layout, but remounts on navigation |
| `loading.tsx` | Fallback while the page's async work is pending |
| `error.tsx` | Error boundary for this segment. Must be a client component |
| `not-found.tsx` | Rendered when `notFound()` is called |
| `route.ts` | An HTTP handler instead of a page. No `page.tsx` beside it in the same folder |

## 2. Layouts, loading, and errors

The root layout wraps every page and holds `<html>` and `<body>`.

```tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

| Piece | Behavior |
| --- | --- |
| `children` | The matched page, or a nested layout |
| `loading.tsx` | Shown immediately, replaced when the page resolves |
| `error.tsx` | Receives `error` and `reset`. `reset` tries to render the segment again |
| Nested layouts | A layout in `app/search/layout.tsx` wraps only `/search` and below |

A layout should not fetch data that changes on every navigation if that work can live in the page. The layout does not remount between those pages, so its state survives.

## 3. Server and client components

By default, a component in the App Router is a server component. It runs on the server, can `await` data, and does not ship its own code as a client bundle. It cannot use `useState`, `useEffect`, or browser events.

A file that starts with `"use client"` is a client component. It can use state and events. It is still pre-rendered to HTML, then hydrated in the browser.

```tsx
"use client";

import { useState } from "react";

export function CityField() {
  const [city, setCity] = useState("");
  return <input value={city} onChange={(event) => setCity(event.target.value)} />;
}
```

| Need | Where |
| --- | --- |
| Read a database or a secret | Server component or a route handler |
| `useState`, clicks, effects | Client component |
| A client child inside a server page | Allowed. Import the client component into the server page |
| A server component imported into a client file | It becomes part of the client graph. Pass server-rendered nodes as `children` instead |

`"use client"` is a boundary, not a label on one function. Every module that file imports is client code too.

## 4. Navigation and dynamic routes

```tsx
import Link from "next/link";

export function Nav() {
  return <Link href="/search">Search</Link>;
}
```

`Link` prefetches and navigates without a full browser reload. An ordinary `<a>` does a full load. Use `Link` for routes in your app.

```tsx
export default async function CityPage({
  params,
}: {
  params: Promise<{ city: string }>;
}) {
  const { city } = await params;
  return <h1>{city}</h1>;
}
```

| API | Use |
| --- | --- |
| `params` | Dynamic segments. In current Next, the page receives it as a promise and you `await` it |
| `searchParams` | The query string. Also a promise in current page props |
| `notFound()` | Render the closest `not-found.tsx` |
| `redirect("/login")` | Send the browser to another URL |
| `useRouter` | Client navigation: `push`, `replace`, `back`. Requires a client component |
| `usePathname` | The current path, in a client component |

`generateStaticParams` lists which dynamic values to prebuild. Omit it and the route can still render on demand.

## 5. Data, mutations, and metadata

A server component may `await` a function that reads your database. That function runs on the server.

```tsx
export default async function ReportPage() {
  const rows = await getSales("Irvine");
  return <ul>{rows.map((row) => <li key={row.id}>{row.total}</li>)}</ul>;
}
```

Do not import server-only secrets into a client component. A variable is public in the browser when its name starts with `NEXT_PUBLIC_`.

| Caching idea | Meaning |
| --- | --- |
| Static | The result can be reused across requests |
| Dynamic | The result depends on the request, such as cookies or a search param |
| `revalidate` | Seconds before the cached result may be refreshed |
| `noStore` / uncached fetch | Read fresh data |

Three ways to load or change data:

| | Server component | Client `fetch` | Server Action |
| --- | --- | --- | --- |
| Runs | On the server, while the page renders | In the browser, usually inside an effect | On the server, when the client calls it |
| Starts | As the page is built | After the page is on screen | On submit, or when you call the function |
| Can read a secret | Yes | No | Yes |
| Can use `useState` | No | Yes | The function itself cannot. A client component may call it |
| Shape | `const rows = await getSales(city)` | `fetch("/api/sales")` then `setRows` | `"use server"` and a form `action` or a direct call |
| When it is the wrong tool | The UI needs a click handler | The data is a secret, or the page should arrive with the data | You only needed to read, and a server component can `await` it |

Route handlers live in `route.ts` and export functions named for HTTP methods.

```tsx
export async function GET() {
  return Response.json({ status: "ok" });
}
```

Server Actions are async functions that run on the server when a form submits. Mark the file or the function with `"use server"`. They are a way to change data without writing a separate API route. Validate the input on the server anyway.

Metadata:

```tsx
export const metadata = {
  title: "Irvine report",
  description: "Twelve-month sales summary.",
};
```

`generateMetadata` is the async version when the title depends on `params`. The root layout can set a title template: `{ default: "Homes", template: "%s · Homes" }`.

## 6. Images, fonts, and environment

`next/image` requires width and height, or `fill` inside a positioned parent. It reserves space and can serve a resized file.

```tsx
import Image from "next/image";

export function Photo() {
  return <Image src="/house.jpg" alt="Front door" width={800} height={600} />;
}
```

Remote image hosts must be listed in `next.config`. Local files in `public/` are served from `/`.

`next/font` loads a font without a layout jump. You apply the class it returns on `body`.

| Variable | Visible in the browser? |
| --- | --- |
| `DATABASE_URL` | No, if you only read it on the server |
| `NEXT_PUBLIC_MAP_KEY` | Yes. Anything that may be public |

A push to a host such as Vercel can build and publish the app. That publish step is not a test suite. Tests run where you configured them.

## 7. Cookies, headers, and middleware

A server component can read the incoming request. In current Next, `cookies()` and `headers()` are async.

```tsx
import { cookies, headers } from "next/headers";

export default async function Page() {
  const cookieStore = await cookies();
  const headerStore = await headers();
  const theme = cookieStore.get("theme")?.value ?? "light";
  const host = headerStore.get("host");
  return <p>{theme} on {host}</p>;
}
```

Reading is allowed while the page renders. Setting a cookie is not. `cookieStore.set` belongs in a Server Action or a route handler. Calling it during render throws.

`middleware.ts` sits next to the `app` directory, or inside `src` when the project uses `src`. It runs before the route renders.

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

| | Server component | Middleware |
| --- | --- | --- |
| Cookies | `await cookies()` from `next/headers` | `request.cookies` |
| Headers | `await headers()` from `next/headers` | `request.headers` |
| When | While that page renders | Before the matched route renders |
| Redirect | `redirect("/login")` from `next/navigation` | `NextResponse.redirect` |
| Set a cookie | In a Server Action or a route handler | On the `NextResponse` you return |

`matcher` limits which paths run the function. A middleware that matches every page runs on static files too unless you exclude them.

## Drill

1. Say which file is the URL `/search`.
2. Say why a click handler file needs `"use client"`.
3. Say which env name would leak into the browser.
4. Say where a page reads a cookie, and where middleware reads that same cookie.
