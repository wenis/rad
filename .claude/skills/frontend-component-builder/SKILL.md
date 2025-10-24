---
name: frontend-component-builder
description: Builds React/Vue/Svelte components with proper state management, accessibility, and responsive design. Use when creating UI components OR implementing design systems OR building forms OR adding interactive features.
allowed-tools: Read, Write, Edit, Bash
---

# Frontend Component Builder

You build production-ready frontend components with accessibility, responsive design, and best practices baked in.

## When to use
- Creating new UI components from designs or specs
- Implementing design system components
- Building forms with validation
- Adding interactive features (modals, dropdowns, tabs)
- Converting Figma/design files to code
- Refactoring legacy components

## Supported Frameworks

### React/Next.js
- Functional components with hooks
- TypeScript for type safety
- Tailwind CSS for styling (or CSS modules, styled-components)
- Component composition patterns

### Vue/Nuxt
- Composition API (Vue 3)
- TypeScript support
- Scoped styles
- Composables for reusable logic

### Svelte/SvelteKit
- Reactive declarations
- Component props and events
- Scoped styles by default
- Stores for state management

## Component Principles

### 1. Accessibility First
- Semantic HTML elements (`<button>`, `<nav>`, `<main>`)
- ARIA attributes when needed
- Keyboard navigation support
- Screen reader friendly
- Focus management

### 2. Responsive Design
- Mobile-first approach
- Breakpoints: sm (640px), md (768px), lg (1024px), xl (1280px)
- Touch-friendly targets (min 44x44px)
- Flexible layouts (flexbox, grid)

### 3. Performance
- Lazy loading for heavy components
- Memoization for expensive computations
- Virtualization for long lists
- Optimized images (next/image, lazy loading)

### 4. Reusability
- Single responsibility
- Props over hardcoded values
- Composition over inheritance
- Variants via props

## Component Examples

### React Button Component

**TypeScript + Tailwind:**
```typescript
// components/Button.tsx
import { ButtonHTMLAttributes, forwardRef } from 'react';
import { cn } from '@/lib/utils';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  (
    {
      variant = 'primary',
      size = 'md',
      isLoading = false,
      leftIcon,
      rightIcon,
      children,
      className,
      disabled,
      ...props
    },
    ref
  ) => {
    const baseStyles = 'inline-flex items-center justify-center font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50';

    const variants = {
      primary: 'bg-blue-600 text-white hover:bg-blue-700 focus-visible:ring-blue-600',
      secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus-visible:ring-gray-400',
      outline: 'border border-gray-300 bg-transparent hover:bg-gray-100 focus-visible:ring-gray-400',
      ghost: 'hover:bg-gray-100 focus-visible:ring-gray-400',
      danger: 'bg-red-600 text-white hover:bg-red-700 focus-visible:ring-red-600',
    };

    const sizes = {
      sm: 'h-8 px-3 text-sm rounded',
      md: 'h-10 px-4 text-base rounded-md',
      lg: 'h-12 px-6 text-lg rounded-lg',
    };

    return (
      <button
        ref={ref}
        className={cn(baseStyles, variants[variant], sizes[size], className)}
        disabled={disabled || isLoading}
        {...props}
      >
        {isLoading && (
          <svg
            className="mr-2 h-4 w-4 animate-spin"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
            />
          </svg>
        )}
        {leftIcon && !isLoading && <span className="mr-2">{leftIcon}</span>}
        {children}
        {rightIcon && <span className="ml-2">{rightIcon}</span>}
      </button>
    );
  }
);

Button.displayName = 'Button';

export { Button };
```

**Usage:**
```tsx
import { Button } from '@/components/Button';
import { ChevronRight } from 'lucide-react';

function Example() {
  return (
    <>
      <Button variant="primary" size="md">
        Click me
      </Button>

      <Button variant="outline" size="lg" rightIcon={<ChevronRight />}>
        Next
      </Button>

      <Button variant="danger" isLoading>
        Deleting...
      </Button>
    </>
  );
}
```

### React Form Component with Validation

**Using React Hook Form + Zod:**
```typescript
// components/ContactForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from './Button';

const contactSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  message: z.string().min(10, 'Message must be at least 10 characters'),
});

type ContactFormData = z.infer<typeof contactSchema>;

interface ContactFormProps {
  onSubmit: (data: ContactFormData) => Promise<void>;
}

export function ContactForm({ onSubmit }: ContactFormProps) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset,
  } = useForm<ContactFormData>({
    resolver: zodResolver(contactSchema),
  });

  const handleFormSubmit = async (data: ContactFormData) => {
    await onSubmit(data);
    reset();
  };

  return (
    <form onSubmit={handleSubmit(handleFormSubmit)} className="space-y-4">
      <div>
        <label htmlFor="name" className="block text-sm font-medium text-gray-700 mb-1">
          Name
        </label>
        <input
          id="name"
          type="text"
          {...register('name')}
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          aria-invalid={errors.name ? 'true' : 'false'}
          aria-describedby={errors.name ? 'name-error' : undefined}
        />
        {errors.name && (
          <p id="name-error" className="mt-1 text-sm text-red-600" role="alert">
            {errors.name.message}
          </p>
        )}
      </div>

      <div>
        <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-1">
          Email
        </label>
        <input
          id="email"
          type="email"
          {...register('email')}
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <p id="email-error" className="mt-1 text-sm text-red-600" role="alert">
            {errors.email.message}
          </p>
        )}
      </div>

      <div>
        <label htmlFor="message" className="block text-sm font-medium text-gray-700 mb-1">
          Message
        </label>
        <textarea
          id="message"
          rows={4}
          {...register('message')}
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          aria-invalid={errors.message ? 'true' : 'false'}
          aria-describedby={errors.message ? 'message-error' : undefined}
        />
        {errors.message && (
          <p id="message-error" className="mt-1 text-sm text-red-600" role="alert">
            {errors.message.message}
          </p>
        )}
      </div>

      <Button type="submit" isLoading={isSubmitting} className="w-full">
        Send Message
      </Button>
    </form>
  );
}
```

### Vue 3 Component (Composition API)

```vue
<!-- components/Card.vue -->
<script setup lang="ts">
import { computed } from 'vue';

interface CardProps {
  variant?: 'default' | 'elevated' | 'outlined';
  padding?: 'none' | 'sm' | 'md' | 'lg';
}

const props = withDefaults(defineProps<CardProps>(), {
  variant: 'default',
  padding: 'md',
});

const cardClasses = computed(() => {
  const base = 'rounded-lg transition-shadow';

  const variants = {
    default: 'bg-white shadow',
    elevated: 'bg-white shadow-lg hover:shadow-xl',
    outlined: 'bg-white border border-gray-200',
  };

  const paddings = {
    none: 'p-0',
    sm: 'p-4',
    md: 'p-6',
    lg: 'p-8',
  };

  return [base, variants[props.variant], paddings[props.padding]];
});
</script>

<template>
  <div :class="cardClasses">
    <div v-if="$slots.header" class="mb-4 pb-4 border-b border-gray-200">
      <slot name="header" />
    </div>

    <div>
      <slot />
    </div>

    <div v-if="$slots.footer" class="mt-4 pt-4 border-t border-gray-200">
      <slot name="footer" />
    </div>
  </div>
</template>
```

**Usage:**
```vue
<Card variant="elevated" padding="lg">
  <template #header>
    <h2 class="text-xl font-bold">Card Title</h2>
  </template>

  <p>Card content goes here...</p>

  <template #footer>
    <button>Action</button>
  </template>
</Card>
```

### Svelte Component

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

## Accessibility Checklist

For every component:
- [ ] Use semantic HTML elements
- [ ] Add ARIA labels for interactive elements
- [ ] Support keyboard navigation (Tab, Enter, Escape, Arrow keys)
- [ ] Manage focus properly (trap focus in modals, restore after close)
- [ ] Provide text alternatives for images/icons
- [ ] Ensure color contrast meets WCAG AA (4.5:1 for text)
- [ ] Test with screen reader (VoiceOver, NVDA)
- [ ] Add loading/error states
- [ ] Make touch targets at least 44x44px

## Responsive Design Patterns

### Mobile-First Breakpoints (Tailwind)
```typescript
// Breakpoints
sm: 640px   // Small tablets
md: 768px   // Tablets
lg: 1024px  // Small desktops
xl: 1280px  // Desktops
2xl: 1536px // Large desktops

// Example: Stack on mobile, grid on desktop
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* Cards */}
</div>

// Example: Hide on mobile, show on desktop
<div className="hidden lg:block">
  {/* Desktop-only sidebar */}
</div>
```

## Component Documentation

Document each component with:

```typescript
/**
 * Button component with multiple variants and sizes
 *
 * @example
 * ```tsx
 * <Button variant="primary" size="md" onClick={handleClick}>
 *   Click me
 * </Button>
 * ```
 */
```

Or use Storybook:
```typescript
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Button',
  },
};

export const Loading: Story = {
  args: {
    variant: 'primary',
    isLoading: true,
    children: 'Loading...',
  },
};
```

## Testing Components

**React Testing Library:**
```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders with children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when loading', () => {
    render(<Button isLoading>Loading</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('is accessible', () => {
    render(<Button>Accessible button</Button>);
    const button = screen.getByRole('button');
    expect(button).toHaveAccessibleName('Accessible button');
  });
});
```

## Instructions

1. **Read requirements**: Understand component purpose, variants, states
2. **Check design system**: Use existing colors, spacing, typography
3. **Choose framework**: React/Vue/Svelte based on project
4. **Build component**:
   - Start with semantic HTML
   - Add TypeScript types/interfaces
   - Implement variants via props
   - Add accessibility attributes
   - Style with Tailwind/CSS
5. **Test component**: Write unit tests, check accessibility
6. **Document**: Add JSDoc comments, Storybook stories

## Best Practices

✅ **DO:**
- Use TypeScript for type safety
- Implement keyboard navigation
- Add loading and error states
- Make components responsive
- Test with screen readers
- Document with examples
- Keep components focused (single responsibility)

❌ **DON'T:**
- Use divs for buttons (use `<button>`)
- Forget focus management
- Hardcode colors/spacing
- Ignore mobile users
- Skip accessibility
- Create god components

## Constraints

- Must be accessible (WCAG 2.1 AA minimum)
- Must be responsive (mobile-first)
- Must have TypeScript types
- Must handle loading/error states
- Must be tested
- Must be documented
