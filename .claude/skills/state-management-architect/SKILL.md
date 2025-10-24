---
name: state-management-architect
description: Implements frontend state management using Redux, Zustand, Jotai, React Context, or Vue/Svelte stores with best practices for async state, persistence, and DevTools integration.
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

## State Management Options

### React

| Library | Complexity | Use Case |
|---------|------------|----------|
| **Context API** | Low | Simple global state, theme, auth |
| **Zustand** | Low | Modern, simple global state |
| **Jotai** | Low-Med | Atomic state management |
| **Redux Toolkit** | Medium | Complex apps, time-travel debugging |
| **Recoil** | Medium | Facebook's atomic state |
| **MobX** | Medium | Observable state |
| **TanStack Query** | Medium | Async/server state only |

**Recommendation**: Zustand for most apps, TanStack Query for server state

### Vue

- **Pinia** (recommended): Vue 3 official state management
- **Vuex**: Vue 2 (legacy)
- **Composition API**: Built-in reactive state

### Svelte

- **Stores**: Built-in reactive stores
- **Context API**: Component tree state

## Zustand (React - Recommended)

### Installation
```bash
npm install zustand
```

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
  fetchUser: () => Promise<void>;
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
        body: JSON.stringify({ email, password }),
      });
      const user = await response.json();
      set({ user, isLoading: false });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },

  logout: () => {
    set({ user: null });
  },

  fetchUser: async () => {
    set({ isLoading: true });
    try {
      const response = await fetch('/api/users/me');
      const user = await response.json();
      set({ user, isLoading: false });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },
}));
```

**Usage:**
```typescript
// components/UserProfile.tsx
import { useUserStore } from '../stores/userStore';

export function UserProfile() {
  const { user, isLoading, fetchUser } = useUserStore();

  useEffect(() => {
    fetchUser();
  }, []);

  if (isLoading) return <div>Loading...</div>;
  if (!user) return <div>Not logged in</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// Accessing outside components
const user = useUserStore.getState().user;
```

### Middleware (Persistence, DevTools)

```typescript
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

export const useUserStore = create<UserState>()(
  devtools(
    persist(
      (set, get) => ({
        // Store definition
      }),
      {
        name: 'user-storage', // LocalStorage key
        partialize: (state) => ({ user: state.user }), // Only persist user
      }
    )
  )
);
```

### Immer for Immutable Updates

```typescript
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

interface TodoState {
  todos: Todo[];
  addTodo: (todo: Todo) => void;
  toggleTodo: (id: string) => void;
}

export const useTodoStore = create<TodoState>()(
  immer((set) => ({
    todos: [],

    addTodo: (todo) => set((state) => {
      state.todos.push(todo); // Mutate draft directly!
    }),

    toggleTodo: (id) => set((state) => {
      const todo = state.todos.find(t => t.id === id);
      if (todo) {
        todo.completed = !todo.completed;
      }
    }),
  }))
);
```

## Redux Toolkit (React)

### Installation
```bash
npm install @reduxjs/toolkit react-redux
```

### Store Setup

```typescript
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './slices/userSlice';
import todosReducer from './slices/todosSlice';

export const store = configureStore({
  reducer: {
    user: userReducer,
    todos: todosReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: false, // If using non-serializable data
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// app/store.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch: () => AppDispatch = useDispatch;
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Slice

```typescript
// store/slices/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  email: string;
  name: string;
}

interface UserState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: UserState = {
  user: null,
  isLoading: false,
  error: null,
};

// Async thunk
export const loginUser = createAsyncThunk(
  'user/login',
  async ({ email, password }: { email: string; password: string }) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });
    return response.json();
  }
);

export const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
    },
    logout: (state) => {
      state.user = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.user = action.payload;
        state.isLoading = false;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.error = action.error.message || 'Login failed';
        state.isLoading = false;
      });
  },
});

export const { setUser, logout } = userSlice.actions;
export default userSlice.reducer;
```

**Usage:**
```typescript
import { useAppDispatch, useAppSelector } from '../store/hooks';
import { loginUser, logout } from '../store/slices/userSlice';

export function LoginForm() {
  const dispatch = useAppDispatch();
  const { user, isLoading, error } = useAppSelector((state) => state.user);

  const handleLogin = async (email: string, password: string) => {
    await dispatch(loginUser({ email, password }));
  };

  // ...
}
```

## TanStack Query (React Server State)

```typescript
// hooks/useUser.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(res => res.json()),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (user: User) =>
      fetch(`/api/users/${user.id}`, {
        method: 'PUT',
        body: JSON.stringify(user),
      }),
    onSuccess: (data, variables) => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['user', variables.id] });
    },
  });
}
```

**Usage:**
```typescript
export function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useUser(userId);
  const updateUser = useUpdateUser();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  const handleUpdate = () => {
    updateUser.mutate({ ...user, name: 'New Name' });
  };

  return <div>{user.name}</div>;
}
```

## Pinia (Vue 3)

```typescript
// stores/user.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export const useUserStore = defineStore('user', () => {
  // State
  const user = ref<User | null>(null);
  const isLoading = ref(false);
  const error = ref<string | null>(null);

  // Getters
  const isLoggedIn = computed(() => user.value !== null);
  const userName = computed(() => user.value?.name ?? 'Guest');

  // Actions
  async function login(email: string, password: string) {
    isLoading.value = true;
    error.value = null;

    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        body: JSON.stringify({ email, password }),
      });
      user.value = await response.json();
    } catch (e) {
      error.value = e.message;
    } finally {
      isLoading.value = false;
    }
  }

  function logout() {
    user.value = null;
  }

  return {
    user,
    isLoading,
    error,
    isLoggedIn,
    userName,
    login,
    logout,
  };
});
```

**Usage:**
```vue
<script setup lang="ts">
import { useUserStore } from '@/stores/user';

const userStore = useUserStore();

async function handleLogin() {
  await userStore.login(email.value, password.value);
}
</script>

<template>
  <div v-if="userStore.isLoggedIn">
    <h1>Hello, {{ userStore.userName }}</h1>
    <button @click="userStore.logout">Logout</button>
  </div>
</template>
```

## Svelte Stores

```typescript
// stores/user.ts
import { writable, derived } from 'svelte/store';

interface User {
  id: string;
  email: string;
  name: string;
}

function createUserStore() {
  const { subscribe, set, update } = writable<User | null>(null);

  return {
    subscribe,
    login: async (email: string, password: string) => {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        body: JSON.stringify({ email, password }),
      });
      const user = await response.json();
      set(user);
    },
    logout: () => set(null),
    updateName: (name: string) => update(u => u ? { ...u, name } : null),
  };
}

export const user = createUserStore();
export const isLoggedIn = derived(user, $user => $user !== null);
```

**Usage:**
```svelte
<script lang="ts">
  import { user, isLoggedIn } from './stores/user';

  async function handleLogin() {
    await user.login(email, password);
  }
</script>

{#if $isLoggedIn}
  <h1>Hello, {$user.name}</h1>
  <button on:click={() => user.logout()}>Logout</button>
{/if}
```

## Patterns

### Separation of Client and Server State

**Client state**: UI state, form inputs, modals, themes
**Server state**: Data from API

```typescript
// Client state (Zustand)
const useUIStore = create((set) => ({
  theme: 'light',
  sidebarOpen: true,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
}));

// Server state (TanStack Query)
const { data: users } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
```

### Optimistic Updates

```typescript
const updateUser = useMutation({
  mutationFn: updateUserAPI,
  onMutate: async (newUser) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['user', newUser.id] });

    // Snapshot previous value
    const previousUser = queryClient.getQueryData(['user', newUser.id]);

    // Optimistically update
    queryClient.setQueryData(['user', newUser.id], newUser);

    return { previousUser };
  },
  onError: (err, newUser, context) => {
    // Rollback on error
    queryClient.setQueryData(['user', newUser.id], context.previousUser);
  },
  onSettled: (newUser) => {
    // Refetch after error or success
    queryClient.invalidateQueries({ queryKey: ['user', newUser.id] });
  },
});
```

### Normalized State (Redux)

```typescript
interface NormalizedState {
  users: {
    byId: Record<string, User>;
    allIds: string[];
  };
}

// Add user
state.users.byId[user.id] = user;
state.users.allIds.push(user.id);

// Selectors
const selectUser = (state: RootState, userId: string) =>
  state.users.byId[userId];

const selectAllUsers = (state: RootState) =>
  state.users.allIds.map(id => state.users.byId[id]);
```

### Form State

```typescript
// Complex forms with Zustand
interface FormState {
  values: Record<string, any>;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;

  setValue: (field: string, value: any) => void;
  setError: (field: string, error: string) => void;
  setTouched: (field: string) => void;
  reset: () => void;
  submit: () => Promise<void>;
}

// Or use React Hook Form for forms specifically
```

## Performance Optimization

### Selectors (prevent unnecessary re-renders)

```typescript
// Bad: Re-renders on any user state change
const { user, isLoading, error } = useUserStore();

// Good: Only re-renders when user changes
const user = useUserStore((state) => state.user);
const isLoading = useUserStore((state) => state.isLoading);
```

### Shallow Equality

```typescript
import shallow from 'zustand/shallow';

const { user, isLoading } = useUserStore(
  (state) => ({ user: state.user, isLoading: state.isLoading }),
  shallow
);
```

### Computed/Derived State

```typescript
// Zustand (compute in component)
const user = useUserStore((state) => state.user);
const isAdmin = user?.role === 'admin';

// Redux (use selectors with Reselect)
import { createSelector } from '@reduxjs/toolkit';

const selectUser = (state: RootState) => state.user.user;
const selectIsAdmin = createSelector(
  [selectUser],
  (user) => user?.role === 'admin'
);
```

## DevTools

### Redux DevTools

```typescript
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({
  reducer,
  devTools: process.env.NODE_ENV !== 'production',
});
```

### Zustand DevTools

```typescript
import { devtools } from 'zustand/middleware';

const useStore = create(
  devtools((set) => ({
    // store
  }), { name: 'MyStore' })
);
```

## Best Practices

✅ **DO:**
- Separate client and server state
- Use TypeScript for type safety
- Minimize global state (prefer local when possible)
- Use selectors to prevent re-renders
- Persist critical state (auth, theme)
- Use DevTools for debugging
- Test state logic

❌ **DON'T:**
- Put everything in global state
- Mutate state directly (unless using Immer)
- Store derived data (compute from state)
- Ignore performance (unnecessary re-renders)
- Mix UI and server state
- Store non-serializable data (unless necessary)

## Instructions

1. **Analyze requirements**: What state is needed?
2. **Choose solution**: Zustand (simple), Redux (complex), TanStack Query (server)
3. **Define state shape**: Types, initial values
4. **Implement actions**: How state changes
5. **Add persistence**: LocalStorage for auth, settings
6. **Optimize selectors**: Prevent unnecessary re-renders
7. **Add DevTools**: Enable debugging
8. **Test**: Unit tests for state logic

## Constraints

- Must use TypeScript
- Must separate client and server state
- Must not mutate state directly
- Must minimize global state
- Must optimize for performance
- Must enable DevTools in development
