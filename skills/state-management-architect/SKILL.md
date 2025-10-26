---
name: state-management-architect
description: Chooses and implements optimal state management for React/Vue/Svelte applications based on app complexity and state type (client vs server). Implements Zustand, Redux Toolkit, TanStack Query (React Query), Pinia, or Svelte stores with TypeScript. Creates feature-based stores, implements persistence (localStorage), adds DevTools integration, and optimizes re-renders with selectors. Use when choosing state library, migrating from Redux, implementing global state, adding data fetching with caching, or optimizing component re-renders.
allowed-tools: Read, Write, Edit
---

# State Management Architect

You design and implement scalable, maintainable state management solutions for frontend applications.

## When to use
- Building new frontend application
- Refactoring unmanageable state
- Adding global state to app
- Implementing complex forms
- Managing async data (API calls)
- Adding state persistence
- Optimizing re-renders

## Choosing State Management

### Decision Tree

**1. Server/Async State?**
→ Use **TanStack Query** (React Query) or **SWR**
- API data, caching, refetching
- Don't put server data in global state!

**2. Simple Global State?**
→ React: **Zustand** | Vue: **Pinia** | Svelte: **Stores**
- User auth, theme, simple shared data
- Lightweight, minimal boilerplate

**3. Complex Application?**
→ React: **Redux Toolkit** | Vue: **Pinia**
- Large team, many features
- Time-travel debugging needed
- Strict patterns required

**4. Atomic/Granular State?**
→ React: **Jotai** or **Recoil**
- Fine-grained reactivity
- Derived state, complex dependencies

**5. Just Component State?**
→ **useState** + **Context API**
- Local to component tree
- No global state needed

## State Management Options

### React

| Library | Complexity | Bundle Size | Use Case |
|---------|------------|-------------|----------|
| **Zustand** | Low | 1.2kb | Simple global state (recommended) |
| **TanStack Query** | Low-Med | 13kb | Server/async state only |
| **Jotai** | Low-Med | 3kb | Atomic state management |
| **Redux Toolkit** | Medium | 11kb | Complex apps, time-travel |
| **Recoil** | Medium | 14kb | Facebook's atomic state |
| **Context API** | Low | 0kb | Built-in, simple cases |

**Recommendation**: Zustand + TanStack Query for most apps

### Vue

- **Pinia** (recommended): Vue 3 official state management
- **Composition API**: Built-in reactive state
- **Vuex**: Vue 2 (legacy, use Pinia instead)

### Svelte

- **Stores**: Built-in reactive stores (writable, readable, derived)
- **Context API**: Component tree state

## Complete Implementation Guides

For detailed examples with middleware, patterns, and best practices:

**Zustand (React)** → `examples/zustand-react.md`
- Basic store setup
- Middleware (persist, devtools, immer)
- Async actions
- Slices pattern
- Testing

**Redux Toolkit (React)** → `examples/redux-toolkit-react.md`
- Store configuration
- Slices with createSlice
- Async thunks (createAsyncThunk)
- RTK Query for API calls
- DevTools integration

**TanStack Query (React)** → `examples/tanstack-query-react.md`
- Queries and mutations
- Caching strategies
- Optimistic updates
- Infinite queries
- Prefetching

**Pinia (Vue)** → `examples/pinia-vue.md`
- Store definition
- Getters and actions
- Composition API style
- Persistence
- DevTools

**Svelte Stores** → `examples/svelte-stores.md`
- Writable stores
- Derived stores
- Custom stores
- Persistence

## Quick Start: Zustand (React)

Most recommended for new React applications.

### Basic Store

```typescript
// stores/userStore.ts
import { create } from 'zustand';

interface User {
  id: string;
  email: string;
  name: string;
}

interface UserState {
  user: User | null;
  isLoading: boolean;
  error: string | null;

  // Actions
  setUser: (user: User | null) => void;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

export const useUserStore = create<UserState>((set, get) => ({
  user: null,
  isLoading: false,
  error: null,

  setUser: (user) => set({ user }),

  login: async (email, password) => {
    set({ isLoading: true, error: null });
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });
      const user = await response.json();
      set({ user, isLoading: false });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },

  logout: () => set({ user: null }),
}));
```

### Usage in Components

```typescript
// components/UserProfile.tsx
import { useUserStore } from '../stores/userStore';

export function UserProfile() {
  // Subscribe to specific state
  const user = useUserStore((state) => state.user);
  const isLoading = useUserStore((state) => state.isLoading);
  const fetchUser = useUserStore((state) => state.fetchUser);

  // Or subscribe to multiple
  const { user, isLoading, logout } = useUserStore();

  if (isLoading) return <div>Loading...</div>;
  if (!user) return <div>Not logged in</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

// Access outside React components
const user = useUserStore.getState().user;
useUserStore.getState().logout();
```

### Persistence + DevTools

```typescript
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

export const useUserStore = create<UserState>()(
  devtools(
    persist(
      (set) => ({
        // Store definition
        user: null,
        login: async (email, password) => { /* ... */ },
        logout: () => set({ user: null }),
      }),
      {
        name: 'user-storage', // localStorage key
        partialize: (state) => ({ user: state.user }), // Only persist user
      }
    ),
    { name: 'UserStore' } // DevTools name
  )
);
```

## Server State (TanStack Query)

For API data, use TanStack Query instead of global state.

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Fetch data
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(res => res.json()),
    staleTime: 5 * 60 * 1000, // Cache for 5 minutes
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return <div>{user.name}</div>;
}

// Mutate data
function UpdateUserForm({ userId }: { userId: string }) {
  const queryClient = useQueryClient();

  const updateUser = useMutation({
    mutationFn: (userData) =>
      fetch(`/api/users/${userId}`, {
        method: 'PUT',
        body: JSON.stringify(userData),
      }),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['user', userId] });
    },
  });

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      updateUser.mutate({ name: 'New Name' });
    }}>
      <button type="submit">Update</button>
    </form>
  );
}
```

## State Organization Patterns

### Feature-Based Slices

```typescript
// stores/authStore.ts
export const useAuthStore = create(/* auth state */);

// stores/cartStore.ts
export const useCartStore = create(/* cart state */);

// stores/uiStore.ts
export const useUIStore = create(/* UI state */);
```

### Slices Pattern (Single Store)

```typescript
// stores/index.ts
import { create } from 'zustand';

interface AppState {
  auth: AuthSlice;
  cart: CartSlice;
  ui: UISlice;
}

const createAuthSlice = (set) => ({
  user: null,
  login: async () => { /* ... */ },
});

const createCartSlice = (set) => ({
  items: [],
  addItem: (item) => { /* ... */ },
});

export const useStore = create<AppState>()((...args) => ({
  auth: createAuthSlice(...args),
  cart: createCartSlice(...args),
  ui: createUISlice(...args),
}));

// Usage: const user = useStore((state) => state.auth.user);
```

## Best Practices

✅ **DO:**
- Separate server state (TanStack Query) from client state (Zustand)
- Use selectors to prevent unnecessary re-renders
- Keep actions in the store, not components
- Use middleware for persistence and DevTools
- Normalize complex nested data
- Test store logic separately from components
- Use TypeScript for type safety
- Keep stores focused (single responsibility)

❌ **DON'T:**
- Put server/API data in global state (use TanStack Query)
- Mutate state directly (use immutable updates)
- Create one massive global store
- Put derived state in store (compute in selectors)
- Ignore re-render optimization
- Mix UI state with domain state excessively
- Over-engineer simple local state (use useState)
- Forget to clean up subscriptions

## Common Patterns

### Derived State (Selectors)

```typescript
// Zustand
const completedTodos = useTodoStore((state) =>
  state.todos.filter(todo => todo.completed)
);

// Or create a selector
const selectCompletedTodos = (state) =>
  state.todos.filter(todo => todo.completed);

const completedTodos = useTodoStore(selectCompletedTodos);
```

### Optimistic Updates

```typescript
// TanStack Query
const deleteTodo = useMutation({
  mutationFn: (id) => fetch(`/api/todos/${id}`, { method: 'DELETE' }),
  onMutate: async (id) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['todos'] });

    // Snapshot previous value
    const previousTodos = queryClient.getQueryData(['todos']);

    // Optimistically update
    queryClient.setQueryData(['todos'], (old) =>
      old.filter(todo => todo.id !== id)
    );

    return { previousTodos };
  },
  onError: (err, id, context) => {
    // Rollback on error
    queryClient.setQueryData(['todos'], context.previousTodos);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

### Reset Store

```typescript
const initialState = {
  user: null,
  isLoading: false,
  error: null,
};

export const useUserStore = create<UserState>((set) => ({
  ...initialState,

  reset: () => set(initialState),

  logout: () => {
    // Clear state
    set(initialState);
    // Or use reset
    useUserStore.getState().reset();
  },
}));
```

## Testing State Management

```typescript
import { renderHook, act } from '@testing-library/react';
import { useUserStore } from './userStore';

describe('userStore', () => {
  beforeEach(() => {
    // Reset store before each test
    useUserStore.getState().reset();
  });

  it('logs in user', async () => {
    const { result } = renderHook(() => useUserStore());

    expect(result.current.user).toBeNull();

    await act(async () => {
      await result.current.login('test@example.com', 'password');
    });

    expect(result.current.user).toEqual({
      id: '1',
      email: 'test@example.com',
      name: 'Test User',
    });
    expect(result.current.isLoading).toBe(false);
  });

  it('handles login error', async () => {
    const { result } = renderHook(() => useUserStore());

    await act(async () => {
      await result.current.login('invalid', 'wrong');
    });

    expect(result.current.user).toBeNull();
    expect(result.current.error).toBeTruthy();
  });
});
```

## Performance Optimization

### Selector Optimization

```typescript
// Bad: Creates new object every render
const { user, isLoading } = useUserStore();

// Good: Subscribe to specific values
const user = useUserStore((state) => state.user);
const isLoading = useUserStore((state) => state.isLoading);

// Or use shallow equality
import shallow from 'zustand/shallow';
const { user, isLoading } = useUserStore(
  (state) => ({ user: state.user, isLoading: state.isLoading }),
  shallow
);
```

### Memoization

```typescript
import { useMemo } from 'react';

const expensiveComputation = useMemo(() => {
  return todos.filter(/* expensive filter */);
}, [todos]);
```

## Instructions

1. **Identify state type**: Server/async → TanStack Query | Simple global → Zustand/Pinia | Complex → Redux Toolkit
2. **Choose library**: React (Zustand + TanStack Query) | Vue (Pinia) | Svelte (Stores)
3. **Structure state**: Feature-based stores (recommended) or slices pattern, normalize nested data
4. **Implement store**: Define interface (TypeScript), create store with state/actions/selectors
5. **Add middleware**: Persistence (localStorage), DevTools, Immer if needed
6. **Optimize**: Use selectors wisely, prevent unnecessary re-renders, memoize computations
7. **Test**: Unit test store logic, integration test with components, test async actions

## Constraints

**Must:** Separate server/client state, use TypeScript, prevent re-renders, handle async state, immutable updates, cleanup subscriptions

**Should:** Use DevTools in dev, persist important state, normalize nested data
- Must test state management logic separately from UI
