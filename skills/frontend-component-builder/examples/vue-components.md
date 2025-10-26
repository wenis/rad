# Vue 3 Components (Composition API)

## Card Component

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

## Button Component

```vue
<!-- components/Button.vue -->
<script setup lang="ts">
import { computed } from 'vue';

interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  disabled?: boolean;
}

const props = withDefaults(defineProps<ButtonProps>(), {
  variant: 'primary',
  size: 'md',
  isLoading: false,
  disabled: false,
});

const emit = defineEmits<{
  click: [event: MouseEvent];
}>();

const buttonClasses = computed(() => {
  const base = 'inline-flex items-center justify-center font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50';

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

  return [base, variants[props.variant], sizes[props.size]];
});

function handleClick(event: MouseEvent) {
  if (!props.disabled && !props.isLoading) {
    emit('click', event);
  }
}
</script>

<template>
  <button
    :class="buttonClasses"
    :disabled="disabled || isLoading"
    @click="handleClick"
  >
    <svg
      v-if="isLoading"
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
    <slot />
  </button>
</template>
```

## Form Component with Validation

```vue
<!-- components/ContactForm.vue -->
<script setup lang="ts">
import { ref, reactive } from 'vue';
import { useForm } from 'vee-validate';
import * as yup from 'yup';

interface ContactFormData {
  name: string;
  email: string;
  message: string;
}

const emit = defineEmits<{
  submit: [data: ContactFormData];
}>();

const schema = yup.object({
  name: yup.string().required('Name is required').min(2, 'Name must be at least 2 characters'),
  email: yup.string().required('Email is required').email('Invalid email address'),
  message: yup.string().required('Message is required').min(10, 'Message must be at least 10 characters'),
});

const { errors, defineField, handleSubmit, resetForm } = useForm({
  validationSchema: schema,
});

const [name, nameAttrs] = defineField('name');
const [email, emailAttrs] = defineField('email');
const [message, messageAttrs] = defineField('message');

const isSubmitting = ref(false);

const onSubmit = handleSubmit(async (values) => {
  isSubmitting.value = true;
  try {
    emit('submit', values as ContactFormData);
    resetForm();
  } finally {
    isSubmitting.value = false;
  }
});
</script>

<template>
  <form @submit="onSubmit" class="space-y-4">
    <div>
      <label for="name" class="block text-sm font-medium text-gray-700 mb-1">
        Name
      </label>
      <input
        id="name"
        v-model="name"
        v-bind="nameAttrs"
        type="text"
        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        :aria-invalid="!!errors.name"
        :aria-describedby="errors.name ? 'name-error' : undefined"
      />
      <p v-if="errors.name" id="name-error" class="mt-1 text-sm text-red-600" role="alert">
        {{ errors.name }}
      </p>
    </div>

    <div>
      <label for="email" class="block text-sm font-medium text-gray-700 mb-1">
        Email
      </label>
      <input
        id="email"
        v-model="email"
        v-bind="emailAttrs"
        type="email"
        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        :aria-invalid="!!errors.email"
        :aria-describedby="errors.email ? 'email-error' : undefined"
      />
      <p v-if="errors.email" id="email-error" class="mt-1 text-sm text-red-600" role="alert">
        {{ errors.email }}
      </p>
    </div>

    <div>
      <label for="message" class="block text-sm font-medium text-gray-700 mb-1">
        Message
      </label>
      <textarea
        id="message"
        v-model="message"
        v-bind="messageAttrs"
        rows="4"
        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        :aria-invalid="!!errors.message"
        :aria-describedby="errors.message ? 'message-error' : undefined"
      />
      <p v-if="errors.message" id="message-error" class="mt-1 text-sm text-red-600" role="alert">
        {{ errors.message }}
      </p>
    </div>

    <Button type="submit" :is-loading="isSubmitting" class="w-full">
      Send Message
    </Button>
  </form>
</template>
```

## Composables

### useToggle

```typescript
// composables/useToggle.ts
import { ref } from 'vue';

export function useToggle(initialValue = false) {
  const value = ref(initialValue);

  function toggle() {
    value.value = !value.value;
  }

  function setTrue() {
    value.value = true;
  }

  function setFalse() {
    value.value = false;
  }

  return {
    value,
    toggle,
    setTrue,
    setFalse,
  };
}
```

### useClickOutside

```typescript
// composables/useClickOutside.ts
import { onMounted, onUnmounted, Ref } from 'vue';

export function useClickOutside(
  elementRef: Ref<HTMLElement | null>,
  callback: () => void
) {
  function handleClick(event: MouseEvent) {
    if (elementRef.value && !elementRef.value.contains(event.target as Node)) {
      callback();
    }
  }

  onMounted(() => {
    document.addEventListener('click', handleClick);
  });

  onUnmounted(() => {
    document.removeEventListener('click', handleClick);
  });
}
```
