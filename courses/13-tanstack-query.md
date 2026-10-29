# TanStack Query

Course 13 of 15. Previous: [12-tailwind.md](12-tailwind.md). Next: [14-vitest.md](14-vitest.md).

TanStack Query stores server state in the browser. A query reads. A mutation writes. The cache remembers the last result, can show it again immediately, and can refetch when you say the data is stale. It is not a replacement for React state that never leaves the page, such as whether a menu is open.

The package for React is `@tanstack/react-query`.

## Contents

1. [The client and the provider](#1-the-client-and-the-provider)
2. [Queries](#2-queries)
3. [Query keys](#3-query-keys)
4. [Status, placeholders, and refetch](#4-status-placeholders-and-refetch)
5. [Mutations](#5-mutations)
6. [Invalidation and the cache](#6-invalidation-and-the-cache)
7. [Dependent queries and pagination](#7-dependent-queries-and-pagination)

## 1. The client and the provider

`QueryClient` holds the cache. `QueryClientProvider` makes it available to hooks. One client wraps the part of the tree that fetches.

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

Create the client once. Creating it inside a component without `useState` builds a new cache on every render.

| Default you may set | Meaning |
| --- | --- |
| `queries.staleTime` | How long a result stays fresh. `0` means stale immediately |
| `queries.gcTime` | How long an unused result stays in memory |
| `queries.retry` | How many times a failed query retries |
| `queries.refetchOnWindowFocus` | Refetch stale queries when the window is focused |

## 2. Queries

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

`queryFn` must return a promise. Throw, or reject, on failure. A function that returns `{ error }` without throwing looks like success.

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

Pending is not the same as an empty list. Empty means the request finished and the array has no items.

## 3. Query keys

The key is the identity of the cached result. It is an array.

| Key | Identifies |
| --- | --- |
| `["teams"]` | All teams, if the request does not depend on a user |
| `["teams", userId]` | That user's teams |
| `["teams", userId, { status: "public" }]` | That filter. Objects in keys are hashed by value |
| `["team", teamId]` | One team |

Include every value that changes the request. If `userId` is missing from the key, a new user can see the previous user's cached list.

Keys are hierarchical. Invalidating `["teams"]` also invalidates `["teams", userId]`.

```tsx
useQuery({
  queryKey: ["teams", userId],
  queryFn: () => fetchTeams(userId),
  enabled: Boolean(userId),
});
```

`enabled: false` does not run the query. Use it when an id is not known yet.

## 4. Status, placeholders, and refetch

| Option | Effect |
| --- | --- |
| `staleTime: 60_000` | The result is fresh for one minute. Focus and remount will not refetch during that minute |
| `gcTime` | After the last component unmounts, the cache entry waits this long before it is deleted |
| `retry: 1` | One retry after a failure |
| `placeholderData: []` | `data` is `[]` while the first request is in flight. The query is still pending in status terms you should check carefully: prefer `isPending` plus a skeleton |
| `placeholderData: keepPreviousData` | Imported from the package in v5 as `keepPreviousData`. Shows the previous key's data while the next key loads |

A background refetch sets `isFetching` and leaves `data` in place. Do not replace the list with a loading message just because `isFetching` is true. Gate the loading message on `isPending`.

The window focus refetch is why a list updates when the user comes back to the tab. If the data rarely changes, raise `staleTime`.

## 5. Mutations

A mutation is a write. It does not run on mount. You call `mutate` or `mutateAsync`.

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

| Field or callback | When |
| --- | --- |
| `mutate(vars)` | Fire and forget. Errors go to `onError` |
| `mutateAsync(vars)` | Returns a promise. You can `await` it inside `try/catch` |
| `onSuccess` | The write succeeded |
| `onError` | The write failed |
| `onSettled` | Either way |
| `isPending` | The write is in flight |

Disable the button while `isPending` is true so a double click does not create two records.

A query and a mutation share names and do different jobs.

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

The status flags on a query, at each moment:

| Moment | `isPending` | `isFetching` | `isError` | `data` |
| --- | --- | --- | --- | --- |
| First load, request in flight | Yes | Yes | No | `undefined` |
| Success, idle | No | No | No | The value |
| Background refetch | No | Yes | No | The previous value. Do not swap in a loading message |
| First load failed | No | No | Yes | `undefined` |
| Refetch failed, old data kept | No | No | No | The previous value. `error` holds the failure |

## 6. Invalidation and the cache

`invalidateQueries` marks matching queries stale and refetches the ones that are currently mounted.

```tsx
await client.invalidateQueries({ queryKey: ["teams", userId] });
```

| Method | Effect |
| --- | --- |
| `invalidateQueries` | Mark stale and refetch active queries |
| `setQueryData` | Write a value into the cache yourself |
| `cancelQueries` | Stop an in-flight refetch, used before an optimistic update |
| `removeQueries` | Drop the cache entry |
| `getQueryData` | Read the cache without a hook |

An optimistic update writes the cache first, then rolls back if the mutation fails.

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

The rollback uses the context returned from `onMutate`. The settled invalidation loads the server's version, which wins over the guess.

## 7. Dependent queries and pagination

A dependent query waits for an id from another query.

```tsx
const user = useQuery({ queryKey: ["me"], queryFn: fetchMe });
const teams = useQuery({
  queryKey: ["teams", user.data?.id],
  queryFn: () => fetchTeams(user.data!.id),
  enabled: Boolean(user.data?.id),
});
```

Pagination keeps a page in the key.

```tsx
useQuery({
  queryKey: ["teams", userId, page],
  queryFn: () => fetchTeamsPage(userId, page),
  placeholderData: keepPreviousData,
});
```

`useInfiniteQuery` is the version for "load more." It stores a list of pages. `fetchNextPage` requests the next one. `getNextPageParam` reads the cursor from the last page. Use it when the server returns a cursor. Use a page number in the key when the server uses offsets.

| Choice | Use |
| --- | --- |
| React state | UI that the server does not store |
| `useQuery` | A read of server data |
| `useMutation` | A write |
| Invalidate | The simple way to refresh after a write |
| `setQueryData` | Optimistic UI, or when you already know the new value |

## Drill

1. Write the key for one user's teams.
2. Say which flag means "no data yet" and which flag means "a refresh is in flight."
3. After a successful create, name the call that makes the list reload.
