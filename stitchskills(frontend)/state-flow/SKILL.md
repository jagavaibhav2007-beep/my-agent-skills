---
name: state-flow
description: Expert state architecture for production React apps. Separates server-state (TanStack Query v5) from client-state (Zustand v5) following the Iron Rule. Covers query patterns, optimistic updates, slice composition with devtools, persistence, and the complete feature folder structure used by top-tier teams.
allowed-tools:
  - "Read"
  - "Write"
  - "Bash"
  - "mcp_context7_resolve-library-id"
  - "mcp_context7_query-docs"
---

# State Flow Skill

You are a **Principal Frontend Architect** specializing in state management for large-scale React applications. You know when to use which tool, how to structure stores for teams of 10+ engineers, and how to avoid the most common production pitfalls (stale data, race conditions, prop-drilling, re-render storms).

**Source Repos (all 10k+ ⭐ on GitHub — all APIs verified via Context7):**
- `pmndrs/zustand` ~54k ⭐ → Zustand v5 slice pattern, middleware, persist
- `tanstack/query` ~46k ⭐ → TanStack Query v5 full API
- `alan2207/bulletproof-react` ~32k ⭐ → queryOptions factory, QueryConfig types, feature folder structure
- `t3-oss/create-t3-app` ~27k ⭐ → tRPC integration, state management philosophy

---

## The Iron Rule of State in React

> **If the data lives on a server, it belongs in TanStack Query.  
> If the data only lives on the client, it belongs in Zustand.  
> Never put server data in Zustand. Never use Zustand as a data fetching layer.**

```
State Decision Tree:
━━━━━━━━━━━━━━━━━━━

Q: Does this data come from an API / database?
   → YES: TanStack Query (useQuery, useMutation)
   → NO: Continue ↓

Q: Does this state need to persist across a page refresh?
   → YES: Zustand with persist middleware → localStorage / sessionStorage
   → NO: Continue ↓

Q: Is this state shared by more than 2 unrelated components?
   → YES: Zustand store (global client-state)
   → NO: React useState / useReducer (local state)

Q: Is this derived from other state? (calculated values)
   → computed selector in Zustand store using get()
   → Never duplicate derived state into a separate variable
```

---

## Installation

```bash
# TanStack Query v5 (the data layer)
npm install @tanstack/react-query @tanstack/react-query-devtools

# Zustand v5 (the client-state layer)
npm install zustand immer
```

---

## Part 1 — TanStack Query v5 (Server State)

### 1A. Global Setup — `QueryClient` Configuration

Configure once in your app root. These defaults define your entire data-freshness strategy.

```tsx
// src/lib/queryClient.ts — configure ONCE, import everywhere
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // staleTime: how long data is considered "fresh" before a background refetch
      staleTime: 1000 * 60 * 5,    // 5 minutes — good default for most apps
      // gcTime (was cacheTime in v4): how long unused data stays in cache before garbage collection
      gcTime: 1000 * 60 * 10,      // 10 minutes
      // Retry failed requests 2 times with exponential backoff (built-in)
      retry: 2,
      // Don't refetch on window focus in production apps with frequent mutations
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 1, // Retry failed mutations once
    },
  },
})
```

```tsx
// src/main.tsx — wrap the whole app
import { QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { queryClient } from './lib/queryClient'

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      {/* Only renders in development — no bundle cost in prod */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

### 1B. Query Keys + `queryOptions` Factory — The Foundation of Caching

```tsx
// src/lib/queryKeys.ts — ALWAYS centralize query keys to avoid cache key typos
// This is the most underrated production pattern. Never use magic strings inline.
// Pattern sourced from: alan2207/bulletproof-react (~32k ⭐)

export const queryKeys = {
  // User
  users: {
    all: ['users'] as const,
    lists: () => [...queryKeys.users.all, 'list'] as const,
    list: (filters: Record<string, unknown>) => [...queryKeys.users.lists(), filters] as const,
    details: () => [...queryKeys.users.all, 'detail'] as const,
    detail: (id: string) => [...queryKeys.users.details(), id] as const,
  },
  // Posts
  posts: {
    all: ['posts'] as const,
    lists: () => [...queryKeys.posts.all, 'list'] as const,
    list: (filters: Record<string, unknown>) => [...queryKeys.posts.all, 'list', filters] as const,
    detail: (id: string) => [...queryKeys.posts.all, 'detail', id] as const,
  },
} as const
```

**Critical Rule (verified — TanStack Query docs):** Always include variables used in `queryFn` inside the `queryKey`. If a variable changes, the key changes → query refetches automatically. Missing this causes stale data bugs.

```tsx
// ❌ WRONG — page variable changes but key doesn't → stale data
useQuery({ queryKey: ['posts'], queryFn: () => fetchPosts(page) })

// ✅ CORRECT — page is in the key, new page = new cache entry
useQuery({ queryKey: queryKeys.posts.list({ page }), queryFn: () => fetchPosts(page) })
```

### 1B2. `queryOptions` Factory — Type-Safe Queries (Bulletproof React Pattern)

The `queryOptions()` helper from TanStack Query v5 makes queries fully type-safe and reusable as both hooks AND prefetch functions — the pattern used by `alan2207/bulletproof-react`:

```tsx
// src/features/posts/api/get-posts.ts
// ✅ VERIFIED — bulletproof-react pattern (~32k ⭐)
import { queryOptions, useQuery, QueryClient } from '@tanstack/react-query'
import type { UseQueryOptions } from '@tanstack/react-query'
import { queryKeys } from '@/lib/queryKeys'

// 1. Define query options as a factory function — usable everywhere
export const getPostsQueryOptions = (filters?: { page?: number }) =>
  queryOptions({
    queryKey: queryKeys.posts.list(filters ?? {}),
    queryFn: () => fetchPosts(filters),
  })

// 2. Type-safe config: allows callers to override staleTime, enabled, etc.
export type QueryConfig<T extends (...args: unknown[]) => unknown> =
  Omit<ReturnType<T>, 'queryKey' | 'queryFn'>

type UsePostsOptions = {
  page?: number
  queryConfig?: Partial<UseQueryOptions<Post[]>>
}

// 3. The hook — spreads base options, then applies caller overrides
export function usePosts({ page = 1, queryConfig }: UsePostsOptions = {}) {
  return useQuery({
    ...getPostsQueryOptions({ page }),
    ...queryConfig,  // ← caller can override staleTime, enabled, etc.
  })
}

// 4. Reuse the same factory for server-side prefetching (SSR/RSC)
export async function prefetchPosts(queryClient: QueryClient, page = 1) {
  await queryClient.prefetchQuery(getPostsQueryOptions({ page }))
  return queryClient
}
```

**Why `queryOptions` factory > raw object:** You define the query key + fn ONCE. Then both `useQuery` and `queryClient.prefetchQuery` use the same definition — zero duplication, zero drift.

### 1C. The Query Hook Pattern — Custom Hooks Per Feature

```tsx
// src/features/posts/api/usePosts.ts
// ✅ VERIFIED — TanStack Query v5 pattern
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { queryKeys } from '@/lib/queryKeys'

// --- Types ---
interface Post {
  id: string
  title: string
  body: string
}

// --- API functions (plain async functions — NOT hooks) ---
const fetchPosts = async (): Promise<Post[]> => {
  const res = await fetch('/api/posts')
  if (!res.ok) throw new Error('Failed to fetch posts')
  return res.json()
}

const createPost = async (data: Omit<Post, 'id'>): Promise<Post> => {
  const res = await fetch('/api/posts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  })
  if (!res.ok) throw new Error('Failed to create post')
  return res.json()
}

// --- Query hooks ---
export function usePosts() {
  return useQuery({
    queryKey: queryKeys.posts.lists(),
    queryFn: fetchPosts,
    // Override defaults for this specific query if needed
    staleTime: 1000 * 60 * 2, // Posts go stale faster (2 min)
  })
}

export function usePost(id: string) {
  return useQuery({
    queryKey: queryKeys.posts.detail(id),
    queryFn: () => fetch(`/api/posts/${id}`).then(r => r.json()),
    enabled: !!id, // Don't fire if id is empty/undefined
  })
}

// --- Mutation hooks ---
export function useCreatePost() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      // ✅ VERIFIED v5 API — all methods now take object arguments
      queryClient.invalidateQueries({ queryKey: queryKeys.posts.lists() })
    },
    onError: (error) => {
      // Handle error (toast notification, logging, etc.)
      console.error('Create post failed:', error)
    },
  })
}
```

**Usage in component:**
```tsx
function PostsList() {
  const { data: posts, isLoading, isError, error } = usePosts()
  const createPost = useCreatePost()

  if (isLoading) return <PostsSkeleton />
  if (isError) return <ErrorState message={error.message} />

  return (
    <>
      {posts?.map(post => <PostCard key={post.id} post={post} />)}
      <button
        onClick={() => createPost.mutate({ title: 'New', body: 'Body' })}
        disabled={createPost.isPending}
      >
        {createPost.isPending ? 'Creating...' : 'Add Post'}
      </button>
    </>
  )
}
```

### 1D. Optimistic Updates — Instant UI Feedback

Use when you're confident the mutation will succeed. Snaps the UI immediately, rolls back on failure.

```tsx
// ✅ VERIFIED — TanStack Query v5 optimistic update pattern
// Source: github.com/tanstack/query docs/framework/react/guides/optimistic-updates.md

export function useUpdatePost() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (post: Post) =>
      fetch(`/api/posts/${post.id}`, {
        method: 'PUT',
        body: JSON.stringify(post),
      }).then(r => r.json()),

    onMutate: async (newPost) => {
      // 1. Cancel outgoing refetches — prevent them from overwriting our optimistic update
      await queryClient.cancelQueries({ queryKey: queryKeys.posts.detail(newPost.id) })

      // 2. Snapshot the current data for rollback
      const previousPost = queryClient.getQueryData<Post>(queryKeys.posts.detail(newPost.id))

      // 3. Optimistically update the cache immediately
      queryClient.setQueryData(queryKeys.posts.detail(newPost.id), newPost)

      // 4. Return snapshot so onError can roll back
      return { previousPost }
    },

    onError: (_err, newPost, context) => {
      // Roll back to what we had before the mutation
      if (context?.previousPost) {
        queryClient.setQueryData(queryKeys.posts.detail(newPost.id), context.previousPost)
      }
    },

    onSettled: (_, __, newPost) => {
      // Always refetch to sync with server truth, regardless of success or error
      queryClient.invalidateQueries({ queryKey: queryKeys.posts.detail(newPost.id) })
    },
  })
}
```

### 1E. Simplified Optimistic Updates (v5 Pattern — Variables)

For simpler cases, use `mutation.variables` to show optimistic state without touching the cache:

```tsx
// ✅ VERIFIED v5 pattern — from TanStack Query v5 migration guide
const queryInfo = usePosts()
const addPostMutation = useCreatePost()

// Simply show the pending item using mutation.variables
return (
  <ul>
    {queryInfo.data?.map(post => <li key={post.id}>{post.title}</li>)}
    {/* Show optimistic item while mutation is in flight */}
    {addPostMutation.isPending && (
      <li style={{ opacity: 0.5 }}>
        {addPostMutation.variables?.title} (saving...)
      </li>
    )}
  </ul>
)
```

### 1F. Prefetching — Eliminate Loading States

```tsx
// ✅ VERIFIED — queryClient.prefetchQuery (v5 object-argument API)
// Prefetch on hover so data is ready when user navigates

function PostLink({ postId }: { postId: string }) {
  const queryClient = useQueryClient()

  return (
    <Link
      href={`/posts/${postId}`}
      onMouseEnter={() => {
        // Prefetch when user hovers — data is ready by the time they click
        // ✅ Reuse queryOptions factory — no duplication
        queryClient.prefetchQuery(getPostQueryOptions(postId))
      }}
    >
      View Post
    </Link>
  )
}
```

### 1G. Parallel Queries — Eliminate Request Waterfalls

```tsx
// ✅ VERIFIED — useSuspenseQueries (TanStack Query v5)
// Source: tanstack.com/query/v5/docs/framework/react/guides/request-waterfalls.md
// PROBLEM: Sequential useQuery calls create waterfalls — each waits for the previous
// SOLUTION: useSuspenseQueries fires all queries in parallel, suspends once for all

import { useSuspenseQueries } from '@tanstack/react-query'

function DashboardPage({ userId }: { userId: string }) {
  // ❌ BAD — waterfall: profile fetches, then posts, then stats (3 round-trips)
  // const profile = useQuery(getUserQueryOptions(userId))
  // const posts = useQuery(getPostsQueryOptions({ userId }))  // waits for profile
  // const stats = useQuery(getStatsQueryOptions(userId))      // waits for posts

  // ✅ GOOD — all 3 fire in parallel, component suspends until ALL are ready
  const [profileQuery, postsQuery, statsQuery] = useSuspenseQueries({
    queries: [
      getUserQueryOptions(userId),
      getPostsQueryOptions({ userId }),
      getStatsQueryOptions(userId),
    ],
  })

  // data is guaranteed defined here (Suspense handles loading/error states)
  return (
    <>
      <ProfileCard user={profileQuery.data} />
      <PostsList posts={postsQuery.data} />
      <StatsWidget stats={statsQuery.data} />
    </>
  )
}
```

### 1H. Infinite Scroll / Pagination

```tsx
// ✅ VERIFIED — useSuspenseInfiniteQuery (TanStack Query v5)
// Source: tanstack.com/query/v5/docs/framework/react/reference/useSuspenseInfiniteQuery

import { useSuspenseInfiniteQuery } from '@tanstack/react-query'

export function useInfinitePosts() {
  return useSuspenseInfiniteQuery({
    queryKey: queryKeys.posts.all,     // ← key must include any filter variables
    queryFn: ({ pageParam }) => fetchPosts({ cursor: pageParam }),
    initialPageParam: 0,              // ← REQUIRED in v5 (was automatic in v4)
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    // data.pages is always defined — no undefined checks needed with Suspense
  })
}

function InfinitePostsList() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfinitePosts()

  return (
    <>
      {data.pages.flatMap(page => page.items).map(post => (
        <PostCard key={post.id} post={post} />
      ))}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage ? 'Loading...' : hasNextPage ? 'Load more' : 'All loaded'}
      </button>
    </>
  )
}
```

---

## Part 2 — Zustand v5 (Client State)

### 2A. The Slice Pattern — Scalable Store Architecture

For any store with more than 3-4 fields, use the **Slice Pattern**. Each feature owns its own slice. All slices share one store.

```tsx
// ✅ VERIFIED — Zustand slice pattern
// Source: github.com/pmndrs/zustand docs/learn/guides/advanced-typescript.md

import { create, StateCreator } from 'zustand'
import { devtools, persist } from 'zustand/middleware'
import { immer } from 'zustand/middleware/immer'

// ── Slice Definitions ────────────────────────────────────────────────────────

interface AuthSlice {
  user: { id: string; name: string; role: 'admin' | 'user' } | null
  isAuthenticated: boolean
  login: (user: AuthSlice['user']) => void
  logout: () => void
}

interface UISlice {
  sidebarOpen: boolean
  theme: 'light' | 'dark'
  toggleSidebar: () => void
  setTheme: (theme: UISlice['theme']) => void
}

interface CartSlice {
  items: Array<{ id: string; qty: number; price: number }>
  addItem: (item: CartSlice['items'][0]) => void
  removeItem: (id: string) => void
  total: () => number
}

// Full store type = union of all slices
type AppStore = AuthSlice & UISlice & CartSlice

// ── Slice Creators ──────────────────────────────────────────────────────────

// Each slice gets [['zustand/devtools', never]] in mutators to enable debug names
const createAuthSlice: StateCreator<
  AppStore,
  [['zustand/devtools', never], ['zustand/immer', never]],
  [],
  AuthSlice
> = (set) => ({
  user: null,
  isAuthenticated: false,
  login: (user) =>
    set(
      (state) => { state.user = user; state.isAuthenticated = true },
      undefined,
      'auth/login',        // ← Named action shows in Redux DevTools
    ),
  logout: () =>
    set(
      (state) => { state.user = null; state.isAuthenticated = false },
      undefined,
      'auth/logout',
    ),
})

const createUISlice: StateCreator<
  AppStore,
  [['zustand/devtools', never], ['zustand/immer', never]],
  [],
  UISlice
> = (set) => ({
  sidebarOpen: false,
  theme: 'light',
  toggleSidebar: () =>
    set((state) => { state.sidebarOpen = !state.sidebarOpen }, undefined, 'ui/toggleSidebar'),
  setTheme: (theme) =>
    set((state) => { state.theme = theme }, undefined, 'ui/setTheme'),
})

const createCartSlice: StateCreator<
  AppStore,
  [['zustand/devtools', never], ['zustand/immer', never]],
  [],
  CartSlice
> = (set, get) => ({
  items: [],
  addItem: (item) =>
    set((state) => {
      const existing = state.items.find(i => i.id === item.id)
      if (existing) existing.qty += item.qty
      else state.items.push(item)
    }, undefined, 'cart/addItem'),
  removeItem: (id) =>
    set(
      (state) => { state.items = state.items.filter(i => i.id !== id) },
      undefined,
      'cart/removeItem',
    ),
  // Derived value as a function — computed at call time, never stored
  total: () => get().items.reduce((sum, i) => sum + i.qty * i.price, 0),
})

// ── Composed Store ───────────────────────────────────────────────────────────

export const useStore = create<AppStore>()(
  devtools(
    immer((...args) => ({
      ...createAuthSlice(...args),
      ...createUISlice(...args),
      ...createCartSlice(...args),
    })),
    { name: 'AppStore' }, // Shows as "AppStore" in Redux DevTools
  ),
)
```

### 2B. Selectors — Prevent Re-Render Storms

**Critical:** Never select the whole store. Always select only what the component needs.

```tsx
// ❌ BAD — component re-renders on ANY store change
const { user, sidebarOpen } = useStore()

// ✅ GOOD — component only re-renders when user.name changes
const userName = useStore((state) => state.user?.name)

// ✅ GOOD — stable shallow-compare for multiple primitives
import { useShallow } from 'zustand/react/shallow'

const { sidebarOpen, theme } = useStore(
  useShallow((state) => ({ sidebarOpen: state.sidebarOpen, theme: state.theme }))
)
```

### 2C. Persist Middleware — Survive Page Refresh

```tsx
// ✅ VERIFIED — persist middleware
// Source: github.com/pmndrs/zustand docs/learn/guides/beginner-typescript.md

import { create } from 'zustand'
import { persist, createJSONStorage } from 'zustand/middleware'

interface ThemeStore {
  theme: 'light' | 'dark'
  setTheme: (theme: 'light' | 'dark') => void
}

export const useThemeStore = create<ThemeStore>()(
  persist(
    (set) => ({
      theme: 'light',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'theme-storage',           // localStorage key
      storage: createJSONStorage(() => localStorage),
      // Only persist specific fields (skip transient UI state)
      partialize: (state) => ({ theme: state.theme }),
    },
  ),
)
```

### 2D. Accessing State Outside React

```tsx
// When you need store state outside components (utility functions, middleware, etc.)
import { useStore } from '@/stores/appStore'

// ✅ Access state outside React component
const currentUser = useStore.getState().user

// ✅ Subscribe to changes outside React
const unsubscribe = useStore.subscribe(
  (state) => state.user,
  (user) => console.log('User changed:', user),
)

// Clean up when done
unsubscribe()
```

---

## Part 3 — The Feature Folder Architecture

This structure is directly sourced from **`alan2207/bulletproof-react` (~32k ⭐)**. Every feature is fully self-contained — you can delete a feature folder and nothing outside breaks.

```
src/
├── lib/
│   ├── queryClient.ts        ← QueryClient + defaultOptions config
│   └── queryKeys.ts          ← ALL query key factories (single source of truth)
│
├── stores/
│   ├── appStore.ts           ← Composed Zustand store (all slices joined)
│   └── slices/
│       ├── authSlice.ts      ← Auth state slice
│       ├── uiSlice.ts        ← UI state slice (sidebar, modals, theme)
│       └── cartSlice.ts      ← Cart state slice
│
├── features/
│   └── posts/                ← One folder per domain feature
│       ├── api/
│       │   ├── get-posts.ts     ← queryOptions factory + usePosts hook
│       │   ├── get-post.ts      ← getPostQueryOptions + usePost hook
│       │   └── create-post.ts   ← useCreatePost mutation hook
│       ├── components/
│       │   ├── PostCard.tsx     ← Presentational, no data fetching
│       │   ├── PostsList.tsx    ← Connects useQuery to PostCard
│       │   └── CreatePostForm.tsx
│       ├── hooks/              ← Feature-specific non-query hooks
│       │   └── usePostFilters.ts
│       ├── stores/             ← Feature-scoped Zustand state (if needed)
│       │   └── postDraftStore.ts
│       └── types.ts            ← Post, CreatePostDTO, PostFilters etc.
│
├── components/
│   └── ui/                   ← Shared shadcn/ui components
│
└── types/
    └── api.ts                ← Shared ApiError, PaginatedResponse<T> etc.
```

**Rules (from bulletproof-react):**
- Feature folders are the primary unit of organization — not component type
- `api/` contains ONLY query/mutation hooks + their plain fetcher functions
- `components/` inside a feature are never imported by other features (use shared `components/ui/` for that)
- Never import from `features/A` into `features/B` — communicate through shared stores or URL state
- `queryClient.ts` and `queryKeys.ts` are app-level, never feature-level

### T3 Stack State Philosophy (from `t3-oss/create-t3-app` ~27k ⭐)

> *"tRPC's React Query hooks should take care of your server state. For client state, start with React's useState, and reach for Zustand only when you need more."*

Practical translation:
```
Start here:         useState / useReducer
↓ State is shared across distant components
Go here:            Zustand (global client state)
↓ Data comes from an API
Go here:            TanStack Query (server state)
↓ Full-stack type safety required
Go here:            tRPC + TanStack Query (end-to-end typed)
```

---

## Part 4 — Production Patterns

### Loading States — Show Skeleton, Never Spinners

```tsx
// ✅ PRODUCTION PATTERN — Skeleton > Spinner
function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading, isFetching, isError } = useUser(userId)

  // isLoading = first load, no cached data
  // isFetching = background refetch (has cached data already)

  if (isLoading) return <UserProfileSkeleton />  // First load
  if (isError) return <ErrorBoundary />

  return (
    <div>
      {/* Show a subtle loading indicator on background refetch */}
      {isFetching && <RefetchIndicator />}
      <h1>{data.name}</h1>
    </div>
  )
}
```

### Error Handling — Typed Errors

```tsx
// Wrap your API errors in a typed class
class ApiError extends Error {
  constructor(
    public status: number,
    public code: string,
    message: string,
  ) {
    super(message)
    this.name = 'ApiError'
  }
}

async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`)
  if (!res.ok) {
    const body = await res.json()
    throw new ApiError(res.status, body.code, body.message)
  }
  return res.json()
}

// In a component
const { error } = useUser(userId)
if (error instanceof ApiError && error.status === 404) {
  return <NotFound />
}
```

### Query Retry Strategy

```tsx
// Different retry strategies for different errors
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        // Don't retry on 4xx errors — they won't magically fix themselves
        if (error instanceof ApiError && error.status >= 400 && error.status < 500) {
          return false
        }
        // Retry up to 3 times for network errors and 5xx
        return failureCount < 3
      },
    },
  },
})
```

### Dependent Queries

```tsx
// Query B depends on data from Query A
function UserPosts({ userId }: { userId: string }) {
  const { data: user } = useUser(userId)

  const { data: posts } = useQuery({
    queryKey: queryKeys.posts.list({ userId }),
    queryFn: () => fetchPostsByUser(user!.id),
    enabled: !!user?.id,  // ← Don't run until user.id exists
  })
}
```

---

## Part 5 — Anti-Patterns (Red Flags)

```
Server State (Query) Violations:
❌ Storing API response data in Zustand — React Query handles this
❌ useEffect + fetch + setState — always use useQuery instead
❌ Calling queryClient.invalidateQueries without queryKey scoping — nukes unrelated caches
❌ Not centralizing queryKeys — typo in one place = cache miss in production
❌ Using staleTime: 0 globally — floods your API with requests
❌ v4 multi-argument queryClient methods in v5 — all methods now take a single object
❌ Using cacheTime (v4 name) instead of gcTime (v5 name) — silent default applies
❌ Missing variables from queryKey — data goes stale silently when filters change
❌ Sequential useQuery per data dependency — creates waterfall, use useSuspenseQueries
❌ Duplicating queryFn between hook and prefetch call — use queryOptions factory
❌ Not passing initialPageParam in useInfiniteQuery v5 — this is now required

Client State (Zustand) Violations:
❌ Selecting entire store with useStore() — every state change re-renders component
❌ Not using useShallow when selecting multiple values — same re-render storm
❌ Storing derived/computed data as state — always compute it in the selector or a getter fn
❌ Putting server data in Zustand — it will become stale with no refresh cycle
❌ Using useEffect to sync Zustand with React Query data — anti-pattern
❌ Forgetting immer for nested state updates — leads to mutation bugs

Architecture Violations (sourced from bulletproof-react):
❌ Importing from features/A into features/B — creates hidden coupling
❌ Flat folder structure with component-type folders (hooks/, components/, utils/) — doesn't scale
❌ Feature-specific types in global types/ — kills co-location and discoverability

General:
❌ Using Context API for frequently-changing state — causes entire subtree to re-render
❌ useState for data that should survive navigation — use Zustand persist
❌ Loading spinners over skeleton screens — hurts perceived performance
❌ Skipping Zustand entirely and using global Context for shared state — re-renders whole tree
```

---

## Part 6 — Execution Protocol

When asked to "add state" or "connect this to real data":

```
Step 1: READ THE COMPONENT
  → Identify what data it needs (server or client?)
  → Identify what actions it triggers (mutations?)

Step 2: APPLY THE IRON RULE
  → Server data → TanStack Query
  → Client-only state → Zustand

Step 3: BUILD THE API LAYER FIRST
  → Write the plain async fetch function in features/[name]/api/
  → Wrap it in useQuery or useMutation custom hook
  → Add the query key to queryKeys.ts

Step 4: BUILD THE STORE SLICE (if needed)
  → Add to existing appStore.ts using slice pattern
  → Name your actions: 'feature/actionName' for devtools
  → Use useShallow for any multi-field selector

Step 5: WIRE INTO THE COMPONENT
  → Replace useState/useEffect data fetching with useQuery hook
  → Replace prop-drilled handlers with Zustand actions
  → Add loading/error/skeleton states

Step 6: VALIDATE
  → Open React Query DevTools — verify staleTime, gcTime, cache hit/miss behavior
  → Open Redux DevTools — verify named Zustand actions are appearing
  → Run lighthouse — confirm no waterfall request chains
```
