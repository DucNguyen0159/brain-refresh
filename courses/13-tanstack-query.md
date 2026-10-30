# TanStack Query

Course 13 of 15. Previous: [12-tailwind.md](12-tailwind.md). Next: [14-vitest.md](14-vitest.md).

TanStack Query stores server state in the browser. A query reads. A mutation writes. The cache remembers the last result, can show it again immediately, and can refetch when you say the data is stale. It is not a replacement for React state that never leaves the page, such as whether a menu is open.

The package for React is `@tanstack/react-query`.

Every numbered section has the same parts: the concept, the terms, the methods, several cases with the result, common mistakes, a tip, and a practice with a solution.

## Contents

1. [The client and the provider](#1-the-client-and-the-provider)
2. [Queries](#2-queries)
3. [Query keys](#3-query-keys)
4. [Status, placeholders, and refetch](#4-status-placeholders-and-refetch)
5. [Mutations](#5-mutations)
6. [Invalidation and the cache](#6-invalidation-and-the-cache)
7. [Dependent queries and pagination](#7-dependent-queries-and-pagination)

## 1. The client and the provider

### Concept

`QueryClient` holds the cache. `QueryClientProvider` makes it available to hooks. One client wraps the part of the tree that fetches. Create the client once. Creating it inside a component without `useState` builds a new cache on every render. Defaults on the client decide how long a result stays fresh, how long an unused result stays in memory, how many times a failure retries, and whether a stale query refetches when the window is focused.

### Terms

| Term | Meaning |
| --- | --- |
| Cache | The last successful result for each query key |
| `QueryClient` | The object that holds the cache |
| `QueryClientProvider` | The component that makes that client available to hooks |
| Server state | Data that lives on a server and is copied into the cache |
| Fresh | A result still inside `staleTime`. Focus and remount will not refetch it |
| Stale | A result that may be shown and then refetched |

### Methods

| Default you may set | Meaning |
| --- | --- |
| `queries.staleTime` | How long a result stays fresh. `0` means stale immediately |
| `queries.gcTime` | How long an unused result stays in memory |
| `queries.retry` | How many times a failed query retries |
| `queries.refetchOnWindowFocus` | Refetch stale queries when the window is focused |
| `new QueryClient()` | Creates the cache. Call it once for the tree |
| `<QueryClientProvider client={client}>` | Passes that cache to every `useQuery` below it |

### Examples

**Case: one client around the tree**

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const client = new QueryClient();

export function App() {
  return (
    <QueryClientProvider client={client}>
      <Teams />
    </QueryClientProvider>
  );
}
```

Result: `Teams` and every descendant can call `useQuery`. They share this client's cache.
Why: the provider makes the client available. The client was created once, outside the component.

**Case: a client created during render**

```tsx
export function App() {
  const client = new QueryClient();
  return (
    <QueryClientProvider client={client}>
      <Teams />
    </QueryClientProvider>
  );
}
```

Result: each render builds a new cache. The list you just loaded is dropped.
Why: `new QueryClient()` inside the component, with no `useState`, runs again on every render.

**Case: a client kept with state**

```tsx
import { useState } from "react";

export function App() {
  const [client] = useState(() => new QueryClient());
  return (
    <QueryClientProvider client={client}>
      <Teams />
    </QueryClientProvider>
  );
}
```

Result: the same cache survives re-renders of `App`.
Why: the initializer function runs once. That is the in-component way to create the client once.

**Case: defaults for every query**

```tsx
const client = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,
      gcTime: 300_000,
      retry: 1,
      refetchOnWindowFocus: true,
    },
  },
});
```

Result: a successful query stays fresh for one minute, an unused entry waits five minutes before it is deleted, a failure retries once, and focusing the window refetches stale queries.
Why: `staleTime: 0` would mean stale immediately. These defaults apply to every query that does not set its own.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `new QueryClient()` in the component body | A new cache on every render | Create it once, at module scope or in `useState` |
| Two providers with two clients | A child of the inner provider cannot see the outer cache | One client around the fetching tree |
| Expecting the cache to remember a menu flag | The flag is UI state and disappears from this design | React state for UI that the server does not store |
| Leaving `staleTime` at `0` for data that rarely changes | Every focus refetches | Raise `queries.staleTime` |

### Tip

Put the provider around the routes that fetch, and keep the `QueryClient` in a module-level constant unless the app itself is created per test.

### Practice

Wrap a `Teams` component in a provider. The client must survive a re-render of `App`.

### Solution

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const client = new QueryClient();

export function App() {
  return (
    <QueryClientProvider client={client}>
      <Teams />
    </QueryClientProvider>
  );
}
```

Result: `Teams` reads the same cache after `App` re-renders, because `client` was not created inside the function.

## 2. Queries

### Concept

`useQuery` reads server state. It takes a `queryKey` and a `queryFn`. `queryFn` must return a promise. Throw, or reject, on failure. A function that returns `{ error }` without throwing looks like success. Pending is not the same as an empty list. Empty means the request finished and the array has no items. `isPending` means no data yet. `isFetching` means a request is in flight, including a background refetch.

### Terms

| Field | Meaning |
| --- | --- |
| `data` | The last successful value. May be undefined while pending |
| `error` | The failure, if the query failed |
| `isPending` | No data yet |
| `isLoading` | Pending and currently fetching |
| `isError` | The query failed and there is no successful data to show, unless you keep previous data |
| `isSuccess` | `data` is available |
| `isFetching` | A request is in flight, including a background refetch |
| `refetch` | Run it again |

### Methods

| Piece | Effect |
| --- | --- |
| `useQuery({ queryKey, queryFn })` | Subscribes to that key and runs `queryFn` when the query should fetch |
| `queryFn` | Returns a promise. A thrown error or a rejected promise is failure |
| `query.data` | The cached value after success |
| `teams.isPending` | Gate the first loading message |
| `teams.isError` | Gate the failure message when there is no successful data to show |
| `teams.refetch()` | Runs the query function again |

### Examples

**Case: load, empty, and the list**

```tsx
import { useQuery } from "@tanstack/react-query";

function Teams({ userId }: { userId: string }) {
  const teams = useQuery({
    queryKey: ["teams", userId],
    queryFn: () => fetchTeams(userId),
  });

  if (teams.isPending) return <p>Loading teams.</p>;
  if (teams.isError) return <p>Could not load teams.</p>;
  if (teams.data.length === 0) return <p>No teams yet.</p>;
  return <ul>{teams.data.map((team) => <li key={team.id}>{team.name}</li>)}</ul>;
}
```

Result: the component shows "Loading teams." while there is no data yet, "Could not load teams." when the query failed, "No teams yet." when the array is empty, and the names when the array has items.
Why: `isPending` is no data yet. An empty list is a finished request. `data` is the last successful value.

**Case: a query function that throws**

```tsx
useQuery({
  queryKey: ["teams", userId],
  queryFn: async () => {
    const response = await fetch(`/teams/${userId}`);
    if (!response.ok) throw new Error("Could not load teams");
    return response.json();
  },
});
```

Result: a non-OK response rejects the query. `isError` becomes true and `error` holds that `Error` when there is no successful data to show.
Why: `queryFn` must return a promise and must throw or reject on failure.

**Case: returning `{ error }` looks like success**

```tsx
useQuery({
  queryKey: ["teams", userId],
  queryFn: async () => {
    return { error: "Could not load teams" };
  },
});
```

Result: the query succeeds. `data` is `{ error: "Could not load teams" }`. `isError` stays false.
Why: the function fulfilled. The cache stores that object as the successful value.

**Case: pending versus an empty list**

```tsx
if (teams.isPending) return <p>Loading teams.</p>;
if (teams.data.length === 0) return <p>No teams yet.</p>;
```

Result: the first message shows before any success. The second shows after a success whose array has no items.
Why: pending means no data yet. Empty means the request finished and the array has no items.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Returning `{ error }` from `queryFn` | The query is a success and the UI renders that object | `throw`, or reject the promise |
| Treating `isPending` as "the list is empty" | A finished empty list still shows the loading message if you never check length | After pending, check `data.length` |
| Replacing the list whenever `isFetching` is true | A background refetch hides data you already have | Gate the loading message on `isPending` |
| Reading `data.length` while pending | `data` may be undefined | Check `isPending` first |

### Tip

Write the three branches in this order: pending, error, then the data, including the empty array.

### Practice

Call `useQuery` for one user's teams and render a loading line, an error line, an empty line, and the list.

### Solution

```tsx
function Teams({ userId }: { userId: string }) {
  const teams = useQuery({
    queryKey: ["teams", userId],
    queryFn: () => fetchTeams(userId),
  });

  if (teams.isPending) return <p>Loading teams.</p>;
  if (teams.isError) return <p>Could not load teams.</p>;
  if (teams.data.length === 0) return <p>No teams yet.</p>;
  return <ul>{teams.data.map((team) => <li key={team.id}>{team.name}</li>)}</ul>;
}
```

Result: the component follows pending, then error, then an empty array, then the names.

## 3. Query keys

### Concept

The key is the identity of the cached result. It is an array. Include every value that changes the request. If `userId` is missing from the key, a new user can see the previous user's cached list. Objects in keys are hashed by value. Keys are hierarchical. Invalidating `["teams"]` also invalidates `["teams", userId]`. `enabled: false` does not run the query. Use it when an id is not known yet.

### Terms

| Key | Identifies |
| --- | --- |
| `["teams"]` | All teams, if the request does not depend on a user |
| `["teams", userId]` | That user's teams |
| `["teams", userId, { status: "public" }]` | That filter. Objects in keys are hashed by value |
| `["team", teamId]` | One team |

### Methods

| Call | Effect |
| --- | --- |
| `queryKey: ["teams", userId]` | Cache slot for that user's teams |
| `queryKey: ["teams", userId, { status: "public" }]` | A different slot when the filter object differs by value |
| `enabled: Boolean(userId)` | Skips the fetch while `userId` is missing |
| `invalidateQueries({ queryKey: ["teams"] })` | Marks `["teams"]` and keys under it stale, including `["teams", userId]` |

### Examples

**Case: the key includes the user**

```tsx
useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
});
```

Result: each `userId` gets its own cached list. Switching users shows that user's list, or loads it.
Why: the key is the identity of the cached result. `userId` changes the request, so it belongs in the array.

**Case: a missing user id in the key**

```tsx
useQuery({
  queryKey: ["teams"],
  queryFn: () => fetchTeams(userId),
});
```

Result: a new user can see the previous user's cached list. Both users share `["teams"]`.
Why: the function closed over `userId`, but the cache identity did not. Include every value that changes the request.

**Case: a filter object**

```tsx
useQuery({
  queryKey: ["teams", userId, { status: "public" }],
  queryFn: () => fetchTeams(userId, { status: "public" }),
});
```

Result: `{ status: "public" }` and another object with the same fields share one cache entry. A different status is a different entry.
Why: objects in keys are hashed by value.

**Case: wait until the id exists**

```tsx
useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
  enabled: Boolean(userId),
});
```

Result: while `userId` is `""` or undefined, the query does not run. When it becomes a non-empty id, the query runs.
Why: `enabled: false` does not run the query. Use it when an id is not known yet.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Leaving `userId` out of the key | The next user sees the previous list | `["teams", userId]` |
| A new object identity treated as a new key | Two equal filters still match, which is what you want; a changed field is a new key | Put the filter in the key and rely on hashing by value |
| Fetching with `userId` undefined | The request goes out with a bad id | `enabled: Boolean(userId)` |
| Invalidating `["team", teamId]` and expecting every team's list to refresh | That key is one team | Invalidate `["teams"]` to cover `["teams", userId]` as well |

### Tip

Read the key out loud. If a value appears inside `queryFn` and changes the rows, it belongs in the array.

### Practice

Write the key for one user's public teams, and do not run the query until `userId` is present.

### Solution

```tsx
useQuery({
  queryKey: ["teams", userId, { status: "public" }],
  queryFn: () => fetchTeams(userId, { status: "public" }),
  enabled: Boolean(userId),
});
```

Result: the cache entry is that user plus that filter. The request waits until `userId` is truthy.

## 4. Status, placeholders, and refetch

### Concept

`staleTime` is how long a result stays fresh. During that time, focus and remount will not refetch. `gcTime` is how long an unused result stays after the last component unmounts. A background refetch sets `isFetching` and leaves `data` in place. Do not replace the list with a loading message just because `isFetching` is true. Gate the loading message on `isPending`. The window focus refetch is why a list updates when the user comes back to the tab. If the data rarely changes, raise `staleTime`.

### Terms

| Option | Effect |
| --- | --- |
| `staleTime: 60_000` | The result is fresh for one minute. Focus and remount will not refetch during that minute |
| `gcTime` | After the last component unmounts, the cache entry waits this long before it is deleted |
| `retry: 1` | One retry after a failure |
| `placeholderData: []` | `data` is `[]` while the first request is in flight. The query is still pending in status terms you should check carefully: prefer `isPending` plus a skeleton |
| `placeholderData: keepPreviousData` | Imported from the package in v5 as `keepPreviousData`. Shows the previous key's data while the next key loads |

### Methods

| Flag or option | What the UI should do |
| --- | --- |
| `isPending` | Show the first loading state. No successful data yet |
| `isFetching` | A request is in flight. Keep showing `data` if you have it |
| `isError` | The query failed and there is no successful data to show, unless you keep previous data |
| `placeholderData: []` | Gives `data` an empty array during the first flight. Still prefer `isPending` for the skeleton |
| `placeholderData: keepPreviousData` | Keeps the previous key on screen while the next key loads |
| `refetchOnWindowFocus` | Refetches a stale query when the window is focused |

### Examples

**Case: fresh for one minute**

```tsx
useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
  staleTime: 60_000,
});
```

Result: after a success, focusing the window or remounting within one minute does not refetch. After that minute the result is stale and those events can refetch.
Why: `staleTime: 60_000` is how long the result stays fresh. `0` would mean stale immediately.

**Case: a background refetch keeps the list**

```tsx
if (teams.isPending) return <p>Loading teams.</p>;
return (
  <ul>
    {teams.data.map((team) => <li key={team.id}>{team.name}</li>)}
  </ul>
);
```

Result: the loading line shows only on the first load. A later refetch sets `isFetching` and leaves the names on screen.
Why: a background refetch sets `isFetching` and leaves `data` in place. Gate the loading message on `isPending`.

**Case: an empty placeholder**

```tsx
const teams = useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
  placeholderData: [],
});

if (teams.isPending) return <p>Loading teams.</p>;
```

Result: `data` is `[]` while the first request is in flight, and `isPending` is still the signal for the loading line.
Why: `placeholderData: []` fills `data` early. Prefer `isPending` plus a skeleton rather than treating `[]` as "no teams yet."

**Case: keep the previous page's rows**

```tsx
import { keepPreviousData, useQuery } from "@tanstack/react-query";

useQuery({
  queryKey: ["teams", userId, page],
  queryFn: () => fetchTeamsPage(userId, page),
  placeholderData: keepPreviousData,
});
```

Result: while the next page's key loads, the component still shows the previous key's data.
Why: in v5 `keepPreviousData` is imported from the package and passed as `placeholderData`. It shows the previous key's data while the next key loads.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A loading message on `isFetching` | The list flashes away on every focus refetch | Use `isPending` |
| Reading `placeholderData: []` as an empty result | You show "No teams yet." during the first request | Check `isPending` and show a skeleton |
| A `staleTime` of `0` on a list that rarely changes | Every focus hits the server | Raise `staleTime` |
| Expecting `gcTime` to keep data on screen forever | After the last unmount, the entry is deleted once `gcTime` passes | Raise `gcTime` only if a later mount should reuse it |

### Tip

If the list vanishes when the user returns to the tab, the loading check is probably `isFetching`. Switch that check to `isPending`.

### Practice

Fetch teams with a one-minute `staleTime`, and show "Loading teams." only when `isPending` is true.

### Solution

```tsx
const teams = useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
  staleTime: 60_000,
});

if (teams.isPending) return <p>Loading teams.</p>;
return <ul>{teams.data.map((team) => <li key={team.id}>{team.name}</li>)}</ul>;
```

Result: the loading line is the first load only. For one minute, focus does not refetch. After that, a refetch keeps the list because the gate is `isPending`.

## 5. Mutations

### Concept

A mutation is a write. It does not run on mount. You call `mutate` or `mutateAsync`. `mutate` is fire and forget. Errors go to `onError`. `mutateAsync` returns a promise you can `await` inside `try/catch`. Disable the button while `isPending` is true so a double click does not create two records. A query and a mutation share names and do different jobs. On a query, `isPending` means there is no data yet. On a mutation, `isPending` means this write is in flight.

### Terms

| | `useQuery` | `useMutation` |
| --- | --- | --- |
| When it runs | When a component mounts and `enabled` is not false | When you call `mutate` or `mutateAsync` |
| Identity | `queryKey`. The cache is shared by that key | `mutationKey` is optional. Each call is a write, not a cached read |
| The function | `queryFn`. No argument, or it closes over the key | `mutationFn`. Receives the variables you passed to `mutate` |
| Cached result | `data` stays under the key until `gcTime` | `data` is the last success for this hook instance |
| Freshness | `staleTime`, focus refetch, `refetch` | No stale time. You invalidate the queries the write changed |
| Pending | `isPending` means there is no data yet | `isPending` means this write is in flight |
| Failure | `isError` when the query has no successful data | `isError` when the latest write failed |
| After success | The component reads `data` | `onSuccess` usually calls `invalidateQueries` |

| Moment | `isPending` | `isFetching` | `isError` | `data` |
| --- | --- | --- | --- | --- |
| First load, request in flight | Yes | Yes | No | `undefined` |
| Success, idle | No | No | No | The value |
| Background refetch | No | Yes | No | The previous value. Do not swap in a loading message |
| First load failed | No | No | Yes | `undefined` |
| Refetch failed, old data kept | No | No | No | The previous value. `error` holds the failure |

### Methods

| Field or callback | When |
| --- | --- |
| `mutate(vars)` | Fire and forget. Errors go to `onError` |
| `mutateAsync(vars)` | Returns a promise. You can `await` it inside `try/catch` |
| `onSuccess` | The write succeeded |
| `onError` | The write failed |
| `onSettled` | Either way |
| `isPending` | The write is in flight |
| `mutationFn` | The function that performs the write. It receives the variables |

### Examples

**Case: create, then invalidate**

```tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

function CreateTeam({ userId }: { userId: string }) {
  const client = useQueryClient();
  const create = useMutation({
    mutationFn: (name: string) => createTeam(userId, name),
    onSuccess: () => {
      client.invalidateQueries({ queryKey: ["teams", userId] });
    },
  });

  return (
    <button
      type="button"
      disabled={create.isPending}
      onClick={() => create.mutate("New team")}
    >
      Create
    </button>
  );
}
```

Result: the click writes one team. The button ignores further clicks while `isPending` is true. After success, the teams query refetches.
Why: a mutation runs when you call `mutate`, not on mount. `onSuccess` invalidates `["teams", userId]`.

**Case: `mutateAsync` inside try/catch**

```tsx
try {
  await create.mutateAsync("New team");
} catch (error) {
  console.error(error);
}
```

Result: the call waits for the write. A failure lands in `catch`. A success continues after the `await`.
Why: `mutateAsync` returns a promise. `mutate` would send errors to `onError` instead of rejecting into this `try`.

**Case: the button during the write**

```tsx
<button type="button" disabled={create.isPending} onClick={() => create.mutate("New team")}>
  Create
</button>
```

Result: the button is disabled while the write is in flight, so a double click does not create two records.
Why: on a mutation, `isPending` means this write is in flight. It does not mean "no data yet."

**Case: query flags across a refetch**

```tsx
const teams = useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
});
```

Result: on the first flight, `isPending` and `isFetching` are true and `data` is undefined. After success, a background refetch sets `isFetching` and keeps the previous `data`. A failed first load sets `isError` and leaves `data` undefined. A failed refetch keeps the previous value, leaves `isError` false, and stores the failure on `error`.
Why: those are the query status flags. `isPending` is "no data yet." `isFetching` is "a request is in flight."

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Calling the write inside `useQuery` | The write runs because the component mounted | `useMutation`, then `mutate` from the click |
| Leaving the button enabled while `isPending` | Two clicks create two records | `disabled={create.isPending}` |
| `await create.mutate(...)` | `mutate` does not give you a promise to catch here | `mutateAsync` inside `try/catch` |
| Using mutation `isPending` to decide if the list exists | That flag is about the write, not the cached list | Read the query's `isPending` for the list |

### Tip

Invalidate in `onSuccess` for the simple refresh. The click handler only calls `mutate`.

### Practice

Add a create button that sends a team name, disables itself during the write, and invalidates `["teams", userId]` after success.

### Solution

```tsx
const client = useQueryClient();
const create = useMutation({
  mutationFn: (name: string) => createTeam(userId, name),
  onSuccess: () => {
    client.invalidateQueries({ queryKey: ["teams", userId] });
  },
});

<button type="button" disabled={create.isPending} onClick={() => create.mutate("New team")}>
  Create
</button>
```

Result: one click starts one write. The button stays disabled until it settles. Success marks that user's teams stale and refetches them.

## 6. Invalidation and the cache

### Concept

`invalidateQueries` marks matching queries stale and refetches the ones that are currently mounted. `setQueryData` writes a value into the cache yourself. An optimistic update writes the cache first, then rolls back if the mutation fails. The rollback uses the context returned from `onMutate`. The settled invalidation loads the server's version, which wins over the guess.

### Terms

| Method | Effect |
| --- | --- |
| `invalidateQueries` | Mark stale and refetch active queries |
| `setQueryData` | Write a value into the cache yourself |
| `cancelQueries` | Stop an in-flight refetch, used before an optimistic update |
| `removeQueries` | Drop the cache entry |
| `getQueryData` | Read the cache without a hook |

### Methods

| Call | Effect |
| --- | --- |
| `client.invalidateQueries({ queryKey: ["teams", userId] })` | That key, and keys under it, become stale. Mounted queries refetch |
| `client.setQueryData(key, valueOrUpdater)` | Replaces the cached value immediately |
| `client.getQueryData(key)` | Returns the cached value or `undefined` |
| `client.cancelQueries({ queryKey })` | Stops an in-flight refetch so it cannot overwrite the optimistic value |
| `client.removeQueries({ queryKey })` | Deletes the cache entry |
| `onMutate` return value | The context `onError` uses to roll back |

### Examples

**Case: invalidate after a write**

```tsx
await client.invalidateQueries({ queryKey: ["teams", userId] });
```

Result: the SDK marks `["teams", userId]` stale and refetches it if a component is currently mounted.
Why: invalidation is the simple way to refresh after a write. Inactive queries are stale and refetch when they mount.

**Case: write the cache yourself**

```tsx
client.setQueryData(["teams", userId], (current: Team[] | undefined) => [
  ...(current ?? []),
  { id: "local", name: "New team" },
]);
```

Result: the mounted list shows "New team" immediately, without waiting for a refetch.
Why: `setQueryData` writes a value into the cache yourself. A later invalidate still replaces it with the server's list.

**Case: cancel, then read, before the guess**

```tsx
await client.cancelQueries({ queryKey: ["teams", next.userId] });
const previous = client.getQueryData(["teams", next.userId]);
```

Result: an in-flight refetch is stopped, and `previous` holds the list from before the optimistic edit.
Why: `cancelQueries` is used before an optimistic update so a late response does not overwrite the guess. `getQueryData` reads the cache without a hook.

**Case: optimistic rename with rollback**

```tsx
useMutation({
  mutationFn: renameTeam,
  onMutate: async (next) => {
    await client.cancelQueries({ queryKey: ["teams", next.userId] });
    const previous = client.getQueryData(["teams", next.userId]);
    client.setQueryData(["teams", next.userId], (current: Team[] | undefined) =>
      (current ?? []).map((team) => team.id === next.id ? { ...team, name: next.name } : team),
    );
    return { previous };
  },
  onError: (_error, next, context) => {
    client.setQueryData(["teams", next.userId], context?.previous);
  },
  onSettled: (_data, _error, next) => {
    client.invalidateQueries({ queryKey: ["teams", next.userId] });
  },
});
```

Result: the list shows the new name at once. If the write fails, the cache goes back to `previous`. When the mutation settles, the server's version is fetched and wins over the guess.
Why: the rollback uses the context returned from `onMutate`. `onSettled` runs either way and invalidates.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Optimistic `setQueryData` without `cancelQueries` | A refetch that was already running writes the old list back | `await client.cancelQueries` first |
| Forgetting to return `{ previous }` | `onError` has no snapshot to restore | Return the context from `onMutate` |
| Rolling back and skipping `onSettled` | A failed request can leave the UI on the guess, or a success can leave the guess in place | Invalidate in `onSettled` so the server wins |
| `removeQueries` when you meant "refresh" | The entry is gone and the screen drops to pending | `invalidateQueries` |

### Tip

Reach for `invalidateQueries` first. Add the optimistic block only when the screen must change before the response arrives.

### Practice

After a successful rename, mark `["teams", userId]` stale. In the optimistic version, restore `previous` if the write throws.

### Solution

```tsx
useMutation({
  mutationFn: renameTeam,
  onMutate: async (next) => {
    await client.cancelQueries({ queryKey: ["teams", next.userId] });
    const previous = client.getQueryData(["teams", next.userId]);
    client.setQueryData(["teams", next.userId], (current: Team[] | undefined) =>
      (current ?? []).map((team) => team.id === next.id ? { ...team, name: next.name } : team),
    );
    return { previous };
  },
  onError: (_error, next, context) => {
    client.setQueryData(["teams", next.userId], context?.previous);
  },
  onSettled: (_data, _error, next) => {
    client.invalidateQueries({ queryKey: ["teams", next.userId] });
  },
});
```

Result: the name changes in the cache first. A failure restores `previous`. Settling refetches the server's list.

## 7. Dependent queries and pagination

### Concept

A dependent query waits for an id from another query. `enabled` stays false until that id exists. Pagination keeps a page in the key. `placeholderData: keepPreviousData` holds the previous page on screen while the next page loads. `useInfiniteQuery` is the version for "load more." It stores a list of pages. `fetchNextPage` requests the next one. `getNextPageParam` reads the cursor from the last page. Use it when the server returns a cursor. Use a page number in the key when the server uses offsets.

### Terms

| Choice | Use |
| --- | --- |
| React state | UI that the server does not store |
| `useQuery` | A read of server data |
| `useMutation` | A write |
| Invalidate | The simple way to refresh after a write |
| `setQueryData` | Optimistic UI, or when you already know the new value |
| Page in the key | Offset pagination. `["teams", userId, page]` |
| `useInfiniteQuery` | A cursor and a "load more" button |

### Methods

| Call | Effect |
| --- | --- |
| `enabled: Boolean(user.data?.id)` | The teams query waits until the user query has an id |
| `queryKey: ["teams", user.data?.id]` | The cache slot follows that id |
| `queryKey: ["teams", userId, page]` | One cache entry per page |
| `placeholderData: keepPreviousData` | Shows the previous key while the next key loads |
| `useInfiniteQuery` | Stores a list of pages |
| `fetchNextPage` | Requests the next page |
| `getNextPageParam` | Reads the cursor from the last page. Returning `undefined` means there is no next page |

### Examples

**Case: teams wait for the signed-in user**

```tsx
const user = useQuery({ queryKey: ["me"], queryFn: fetchMe });
const teams = useQuery({
  queryKey: ["teams", user.data?.id],
  queryFn: () => fetchTeams(user.data!.id),
  enabled: Boolean(user.data?.id),
});
```

Result: the teams request does not run until `user.data.id` exists. The key then becomes `["teams", that id]`.
Why: `enabled` is false while the id is missing. The non-null assertion inside `queryFn` runs only after that check.

**Case: a page number in the key**

```tsx
useQuery({
  queryKey: ["teams", userId, page],
  queryFn: () => fetchTeamsPage(userId, page),
  placeholderData: keepPreviousData,
});
```

Result: page 1 and page 2 are different cache entries. Changing `page` shows the previous page until the next request finishes.
Why: the page is part of the key because it changes the request. `keepPreviousData` is the placeholder while the next key loads. Use a page number in the key when the server uses offsets.

**Case: load more with a cursor**

```tsx
const teams = useInfiniteQuery({
  queryKey: ["teams", userId],
  queryFn: ({ pageParam }) => fetchTeamsCursor(userId, pageParam),
  initialPageParam: undefined,
  getNextPageParam: (lastPage) => lastPage.nextCursor,
});
```

Result: the cache stores a list of pages. `teams.fetchNextPage()` requests the next one. When `getNextPageParam` returns `undefined`, there is no further page.
Why: `useInfiniteQuery` is the "load more" form. `getNextPageParam` reads the cursor from the last page. Use it when the server returns a cursor.

**Case: which tool for which job**

```tsx
const [open, setOpen] = useState(false);
const teams = useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
});
```

Result: `open` stays in React state. The team list stays in the query cache under `["teams", userId]`.
Why: React state is for UI the server does not store. `useQuery` is a read of server data. A write would be `useMutation`, then `invalidateQueries` or `setQueryData`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Teams fetch with `user.data.id` before the user query finishes | The request runs with `undefined` | `enabled: Boolean(user.data?.id)` |
| One key for every page | Page 2 replaces page 1 in the same slot | Put `page` in the key |
| `useInfiniteQuery` for an offset API | You invent a cursor the server does not send | A page number in the key |
| Storing "is the menu open" in the query cache | The flag is treated as server data and can refetch away | React state |

### Tip

If the second request needs a value from the first response, that value goes in the second key and in `enabled`.

### Practice

Load the current user, then that user's teams, and show page 2 of the teams without dropping page 1's rows during the fetch.

### Solution

```tsx
import { keepPreviousData, useQuery } from "@tanstack/react-query";

const user = useQuery({ queryKey: ["me"], queryFn: fetchMe });
const teams = useQuery({
  queryKey: ["teams", user.data?.id, page],
  queryFn: () => fetchTeamsPage(user.data!.id, page),
  enabled: Boolean(user.data?.id),
  placeholderData: keepPreviousData,
});
```

Result: teams stay idle until the user id exists. Moving to page 2 keeps page 1 on screen until page 2 arrives. Each page has its own key.

## Drill

1. Write the key for one user's teams.
2. Say which flag means "no data yet" and which flag means "a refresh is in flight."
3. After a successful create, name the call that makes the list reload.

### Solution 1

```tsx
useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
});
```

The key is `["teams", userId]`. The user id is part of the identity, so one user cannot see another user's cached list.

### Solution 2

`isPending` means no data yet. `isFetching` means a request is in flight, including a background refresh. A refresh leaves `data` in place. Gate the loading message on `isPending`.

### Solution 3

```tsx
client.invalidateQueries({ queryKey: ["teams", userId] });
```

Call it from the mutation's `onSuccess`. It marks that key stale and refetches the list if it is mounted.
