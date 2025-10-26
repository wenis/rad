# Svelte Components

## Modal Component

```svelte
<!-- components/Modal.svelte -->
<script lang="ts">
  import { createEventDispatcher, onMount } from 'svelte';
  import { fade, scale } from 'svelte/transition';

  export let isOpen = false;
  export let title: string;
  export let size: 'sm' | 'md' | 'lg' = 'md';

  const dispatch = createEventDispatcher();

  const sizes = {
    sm: 'max-w-md',
    md: 'max-w-lg',
    lg: 'max-w-2xl',
  };

  function handleClose() {
    isOpen = false;
    dispatch('close');
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === 'Escape' && isOpen) {
      handleClose();
    }
  }

  onMount(() => {
    document.addEventListener('keydown', handleKeydown);
    return () => document.removeEventListener('keydown', handleKeydown);
  });
</script>

{#if isOpen}
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4"
    transition:fade={{ duration: 200 }}
    on:click={handleClose}
    role="dialog"
    aria-modal="true"
    aria-labelledby="modal-title"
  >
    <!-- Backdrop -->
    <div class="absolute inset-0 bg-black/50" />

    <!-- Modal -->
    <div
      class="relative bg-white rounded-lg shadow-xl w-full {sizes[size]}"
      transition:scale={{ duration: 200, start: 0.95 }}
      on:click|stopPropagation
    >
      <!-- Header -->
      <div class="flex items-center justify-between px-6 py-4 border-b">
        <h2 id="modal-title" class="text-xl font-semibold">
          {title}
        </h2>
        <button
          on:click={handleClose}
          class="text-gray-400 hover:text-gray-600 transition-colors"
          aria-label="Close modal"
        >
          <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- Content -->
      <div class="px-6 py-4">
        <slot />
      </div>

      <!-- Footer -->
      {#if $$slots.footer}
        <div class="px-6 py-4 border-t bg-gray-50">
          <slot name="footer" />
        </div>
      {/if}
    </div>
  </div>
{/if}
```

**Usage:**
```svelte
<script>
  import Modal from './Modal.svelte';
  let showModal = false;
</script>

<button on:click={() => showModal = true}>
  Open Modal
</button>

<Modal bind:isOpen={showModal} title="Confirm Action" size="md">
  <p>Are you sure you want to continue?</p>

  <div slot="footer" class="flex gap-2 justify-end">
    <button on:click={() => showModal = false}>Cancel</button>
    <button on:click={handleConfirm}>Confirm</button>
  </div>
</Modal>
```

## Button Component

```svelte
<!-- components/Button.svelte -->
<script lang="ts">
  export let variant: 'primary' | 'secondary' | 'outline' | 'danger' = 'primary';
  export let size: 'sm' | 'md' | 'lg' = 'md';
  export let isLoading = false;
  export let disabled = false;
  export let type: 'button' | 'submit' | 'reset' = 'button';

  const baseStyles = 'inline-flex items-center justify-center font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50';

  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus-visible:ring-blue-600',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus-visible:ring-gray-400',
    outline: 'border border-gray-300 bg-transparent hover:bg-gray-100 focus-visible:ring-gray-400',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus-visible:ring-red-600',
  };

  const sizes = {
    sm: 'h-8 px-3 text-sm rounded',
    md: 'h-10 px-4 text-base rounded-md',
    lg: 'h-12 px-6 text-lg rounded-lg',
  };

  $: buttonClass = `${baseStyles} ${variants[variant]} ${sizes[size]}`;
</script>

<button
  {type}
  class={buttonClass}
  disabled={disabled || isLoading}
  on:click
>
  {#if isLoading}
    <svg
      class="mr-2 h-4 w-4 animate-spin"
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
    >
      <circle
        class="opacity-25"
        cx="12"
        cy="12"
        r="10"
        stroke="currentColor"
        stroke-width="4"
      />
      <path
        class="opacity-75"
        fill="currentColor"
        d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
      />
    </svg>
  {/if}
  <slot />
</button>
```

## Tabs Component

```svelte
<!-- components/Tabs.svelte -->
<script lang="ts">
  import { setContext } from 'svelte';
  import { writable } from 'svelte/store';

  export let activeTab: string;

  const activeTabStore = writable(activeTab);
  setContext('tabs', activeTabStore);

  $: activeTabStore.set(activeTab);
</script>

<div class="tabs">
  <slot />
</div>

<style>
  .tabs {
    display: flex;
    flex-direction: column;
  }
</style>
```

```svelte
<!-- components/TabList.svelte -->
<script lang="ts">
  export let ariaLabel = 'Tabs';
</script>

<div role="tablist" aria-label={ariaLabel} class="flex border-b border-gray-200">
  <slot />
</div>
```

```svelte
<!-- components/Tab.svelte -->
<script lang="ts">
  import { getContext } from 'svelte';
  import type { Writable } from 'svelte/store';

  export let value: string;
  export let label: string;

  const activeTab = getContext<Writable<string>>('tabs');

  function handleClick() {
    activeTab.set(value);
  }

  $: isActive = $activeTab === value;
</script>

<button
  role="tab"
  aria-selected={isActive}
  class="px-4 py-2 text-sm font-medium transition-colors border-b-2 {isActive
    ? 'border-blue-600 text-blue-600'
    : 'border-transparent text-gray-600 hover:text-gray-900'}"
  on:click={handleClick}
>
  {label}
</button>
```

```svelte
<!-- components/TabPanel.svelte -->
<script lang="ts">
  import { getContext } from 'svelte';
  import type { Writable } from 'svelte/store';

  export let value: string;

  const activeTab = getContext<Writable<string>>('tabs');

  $: isActive = $activeTab === value;
</script>

{#if isActive}
  <div role="tabpanel" class="py-4">
    <slot />
  </div>
{/if}
```

**Usage:**
```svelte
<script>
  import Tabs from './Tabs.svelte';
  import TabList from './TabList.svelte';
  import Tab from './Tab.svelte';
  import TabPanel from './TabPanel.svelte';

  let activeTab = 'profile';
</script>

<Tabs bind:activeTab>
  <TabList>
    <Tab value="profile" label="Profile" />
    <Tab value="settings" label="Settings" />
    <Tab value="billing" label="Billing" />
  </TabList>

  <TabPanel value="profile">
    <p>Profile content</p>
  </TabPanel>

  <TabPanel value="settings">
    <p>Settings content</p>
  </TabPanel>

  <TabPanel value="billing">
    <p>Billing content</p>
  </TabPanel>
</Tabs>
```

## Stores

### Custom Writable Store

```typescript
// stores/theme.ts
import { writable } from 'svelte/store';

type Theme = 'light' | 'dark';

function createThemeStore() {
  const { subscribe, set, update } = writable<Theme>('light');

  return {
    subscribe,
    set,
    toggle: () => update(theme => theme === 'light' ? 'dark' : 'light'),
    setLight: () => set('light'),
    setDark: () => set('dark'),
  };
}

export const theme = createThemeStore();
```
