---
id: STD-012
name: "Component Composition Patterns"
version: "1.1.0"
category: nextjs
status: active
last_reviewed: "2026-07-18"
discovery_layer: "generation-extracted"
---

# Component Composition Patterns

> Frontend services MUST use TypeScript for type-safe component props, expose variants as a type-safe union prop, expose DOM elements via forwardRef, separate feature logic from presentational UI, and follow accessibility patterns for dialogs, comboboxes, and toasts.

> **Styling & realization.** Design *values* — colour, spacing, radius, typography — MUST bind to design-system tokens as CSS custom properties (`var(--color-surface)`, `var(--spacing-md)`), never hardcoded. The styling *mechanism* itself — the Tailwind utility classes + `class-variance-authority` + `clsx`/`twMerge` shown in the examples below — is a **realization decision owned by the design-system component inventory** (`impl`), not mandated by this standard. A hub whose tech-stack uses CSS Modules bound to `var(--token)` (the current generation default) is equally compliant. **This mechanism is under active redesign — read the examples below for the *coding patterns* (typed props, `forwardRef`, accessibility, feature/UI-tier separation), not for the styling approach.** The token-binding rule is the invariant; the class-authoring tool is not.

## Motivation

Inconsistent component composition leads to unpredictable prop APIs, difficulty in customizing styles, hard-to-test component boundaries, and accessibility violations. A standard pattern ensures components are type-safe, composable, testable, and accessible. Feature components clearly own business logic while UI tier components remain pure and reusable.

## Rules

### Shared UI Components

1. Components that expose internal DOM elements MUST use `forwardRef`:
   ```typescript
   export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
     (props, ref) => <button ref={ref} {...props} />
   );
   ```

2. Components MUST expose variants as a type-safe union prop. The requirement is type-safety, not the tool — `class-variance-authority` (CVA) is one realization (shown here); a CSS-Module class map keyed by the same union is another:
   ```typescript
   const buttonVariants = cva('px-4 py-2 rounded font-medium', {
     variants: {
       variant: {
         primary: 'bg-blue-600 text-white',
         secondary: 'bg-gray-200 text-black',
       },
       size: {
         sm: 'text-sm px-2 py-1',
         md: 'text-base px-4 py-2',
       },
     },
     defaultVariants: { variant: 'primary', size: 'md' },
   });
   ```

3. Class names MUST merge without conflicts. When the styling mechanism is Tailwind, use a `cn()` helper (`clsx` + `twMerge`) as shown; CSS-Module class composition has no conflict problem to solve and needs no such helper:
   ```typescript
   export function cn(...inputs: (string | undefined | false)[]): string {
     return twMerge(clsx(inputs));
   }
   ```

4. Components MUST export a `Props` interface extending the native HTML element interface:
   ```typescript
   export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
     variant?: 'primary' | 'secondary';
     size?: 'sm' | 'md' | 'lg';
     isLoading?: boolean;
   }
   ```

5. Components MUST set `displayName` for debugging in React DevTools:
   ```typescript
   Button.displayName = 'Button';
   ```

6. Error states MUST be props that modify className, not separate error components:
   ```typescript
   // Compliant: error state as prop
   <Input error={!!errors.email} aria-invalid={!!errors.email} />

   // Non-compliant: separate error component
   <Input />
   <ErrorComponent message={errors.email} />
   ```

### Feature Components (Orchestrators)

7. Feature components MUST own React state for form values and UI flags (isOpen, isPending):
   ```typescript
   export function EditOrderDialog() {
     const [isOpen, setIsOpen] = useState(false);
     const [values, setValues] = useState<FormValues>({});
     // ...
   }
   ```

8. Feature components MUST own all business logic hooks (mutations, queries):
   ```typescript
   export function EditOrderDialog() {
     const { mutateAsync, isPending, fieldErrors } = useUpdateOrder();
     const { data: order } = useOrder(orderId);
     // ...
   }
   ```

9. Feature components MUST compose UI-tier components exclusively. They MUST NOT manipulate the DOM directly.

10. Feature components MUST handle side effects (toast, navigation):
    ```typescript
    try {
      await mutateAsync(values);
      toast.success('Order updated');
      navigate('/orders');
    } catch (error) {
      toast.error('Failed to update order');
    }
    ```

11. Feature components MUST map API responses to child component props, making the data transformation explicit:
    ```typescript
    export function OrderListFeature() {
      const { data: orders } = useOrders();

      return (
        <OrderListUI
          orders={orders?.map(o => ({
            id: o.id,
            displayName: `${o.reference} — ${o.customer}`,
            status: o.status,
          }))}
        />
      );
    }
    ```

12. Feature components MUST NOT carry screen-spec mapping comment blocks (`// Maps SCR-…`). The Traceability Matrix owns the spec→component mapping (STD-032); the component's name and composed children make the structure readable on their own.

### Dialog/Modal Components

13. Dialogs MUST have `role="dialog"`, `aria-modal="true"`, and `aria-labelledby`:
    ```typescript
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="dialog-title"
    >
      <h2 id="dialog-title">Edit Order</h2>
      {/* content */}
    </div>
    ```

14. Dialogs MUST close on Escape key, but MUST check `isPending` first to prevent closing during submission:
    ```typescript
    const handleEscape = (e: React.KeyboardEvent) => {
      if (e.key === 'Escape' && !isPending) {
        onClose();
      }
    };
    ```

15. Dialogs MUST implement focus trap: focus the first focusable element on open.

16. Dialogs MUST close when clicking outside (backdrop click), unless a request is pending:
    ```typescript
    const handleBackdropClick = (e: React.MouseEvent) => {
      if (e.target === e.currentTarget && !isPending) {
        onClose();
      }
    };
    ```

17. Dialogs MUST be full-screen on mobile using responsive classes:
    ```typescript
    <div className="max-sm:fixed max-sm:inset-0 max-sm:z-50 md:max-w-lg">
      {/* dialog content */}
    </div>
    ```

### Combobox Components

18. Comboboxes MUST have `role="combobox"` with `aria-expanded`, `aria-controls`, and `aria-activedescendant`:
    ```typescript
    <div role="combobox" aria-expanded={isOpen} aria-controls="results">
      <input aria-activedescendant={activeId} />
      <ul id="results" role="listbox">{/* options */}</ul>
    </div>
    ```

19. Comboboxes MUST support keyboard navigation:
    - ArrowUp / ArrowDown to move focus between options
    - Enter to select the focused option
    - Escape to close the list

20. Comboboxes MUST close when clicking outside.

21. Comboboxes MUST render either the selected label OR the search input (dual render):
    ```typescript
    {selectedValue ? (
      <button onClick={() => setIsOpen(true)}>{selectedLabel}</button>
    ) : (
      <input
        value={searchQuery}
        onChange={e => setSearchQuery(e.target.value)}
        placeholder="Search..."
      />
    )}
    ```

22. Comboboxes MUST include a clear button when a selection exists:
    ```typescript
    {selectedValue && (
      <button onClick={() => { setSelectedValue(null); setSearchQuery(''); }}>
        Clear
      </button>
    )}
    ```

23. Comboboxes MUST display "Searching..." when loading and "No results" when the list is empty.

### Toast/Notification Components

> **Realization is pinned by the design-system inventory, not here.** This section defines the toast/notification **behaviour contract** — the API shape (`toast.success` / `toast.error`), accessibility, and the default 4000ms duration. Whether that contract is realized by a hand-built shared module or a library wrapper is the `impl` decision recorded once on the `toast` entry in the design-system component inventory (`inventory.yml`), which references this standard via `contract: STD-012`. Do NOT hand-roll a toast to avoid a dependency — the inventory's `impl` is the single source for build-vs-buy. The local `@/lib/toast` module referenced in the examples below is the *custom* form of that decision; a `dependency` form wires a package to the same API. Either way the rules in this section hold.

24. Toasts MUST have a default duration of 4000ms for both success and error variants:
    ```typescript
    toast.success('Order created', { duration: 4000 });
    toast.error('Failed to save', { duration: 4000 });
    ```

25. Toast descriptions MUST be user-facing, not technical error codes:
    ```typescript
    // Compliant
    toast.error('Please enter a valid email address');
    // Non-compliant
    toast.error('INVALID_EMAIL_ERR');
    ```

26. Toasts MUST be called from feature components, not from hooks or utilities.

### Config-Driven Components

27. Components displaying badges, banners, or status indicators based on enum values MUST use a static configuration object:
    ```typescript
    const STATUS_CONFIG: Record<OrderStatus, { label: string; className: string }> = {
      active: { label: 'Active', className: 'bg-green-100 text-green-800' },
      inactive: { label: 'Inactive', className: 'bg-gray-100 text-gray-800' },
      archived: { label: 'Archived', className: 'bg-red-100 text-red-800' },
    };

    export function StatusBadge({ status }: { status: OrderStatus }) {
      const config = STATUS_CONFIG[status];
      return <span className={config.className}>{config.label}</span>;
    }
    ```

28. Config-driven components MUST handle missing keys gracefully:
    ```typescript
    const config = STATUS_CONFIG[status] ?? { label: 'Unknown', className: 'bg-gray-100' };
    ```

## Examples

### Compliant: Shared UI component with forwardRef, CVA, and Props

```typescript
// src/components/shared/ui/button/button.tsx

import { forwardRef } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded font-medium transition-colors',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700 disabled:bg-blue-300',
        secondary: 'bg-gray-200 text-black hover:bg-gray-300 disabled:bg-gray-100',
        ghost: 'hover:bg-gray-100 disabled:opacity-50',
      },
      size: {
        sm: 'text-sm px-2 py-1 h-8',
        md: 'text-base px-4 py-2 h-10',
        lg: 'text-lg px-6 py-3 h-12',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, isLoading, disabled, ...props }, ref) => (
    <button
      ref={ref}
      className={cn(buttonVariants({ variant, size, className }))}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading && <span className="mr-2 inline-block animate-spin">⟳</span>}
      {props.children}
    </button>
  )
);

Button.displayName = 'Button';
```

### Compliant: Feature component orchestrating hooks and composing UI

```typescript
// src/components/orders/edit-order/edit-order.tsx

'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { useUpdateOrder } from '@/lib/hooks/use-update-order';
import { useOrder } from '@/lib/hooks/use-orders';
import { validateUpdateOrderForm, fieldErrorMap } from '@/lib/validation/order-validation';
import { toast } from '@/lib/toast';
import { EditOrderForm } from './edit-order-form';
import { ConfirmDialog } from '@/components/shared/ui/confirm-dialog/confirm-dialog';

export function EditOrderFeature({ orderId }: { orderId: string }) {
  const router = useRouter();
  const [values, setValues] = useState<Partial<UpdateOrderRequest>>({});
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [showConfirm, setShowConfirm] = useState(false);

  const { data: order, isLoading: orderLoading } = useOrder(orderId);
  const { mutateAsync, isPending, fieldErrors } = useUpdateOrder();

  const updateField = (field: string, value: any) => {
    setValues(prev => ({ ...prev, [field]: value }));
    setErrors(prev => {
      const newErrors = { ...prev };
      delete newErrors[field];
      return newErrors;
    });
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    const validationErrors = validateUpdateOrderForm(values);
    if (validationErrors.length > 0) {
      setErrors(fieldErrorMap(validationErrors));
      return;
    }

    setShowConfirm(true);
  };

  const handleConfirm = async () => {
    try {
      await mutateAsync({ id: orderId, ...values });
      toast.success('Order updated successfully');
      router.push('/orders');
    } catch (error) {
      if (fieldErrors.length > 0) {
        setErrors(prev => ({
          ...prev,
          ...fieldErrorMap(fieldErrors),
        }));
      }
      toast.error('Failed to update order');
    } finally {
      setShowConfirm(false);
    }
  };

  if (orderLoading) return <div>Loading...</div>;

  return (
    <>
      <EditOrderForm
        initialValues={order}
        values={values}
        errors={errors}
        onSubmit={handleSubmit}
        onFieldChange={updateField}
        isPending={isPending}
      />

      <ConfirmDialog
        open={showConfirm}
        title="Confirm Changes"
        description="Are you sure you want to update this order?"
        onConfirm={handleConfirm}
        onCancel={() => setShowConfirm(false)}
        isPending={isPending}
      />
    </>
  );
}
```

### Compliant: Dialog with accessibility and mobile responsiveness

```typescript
// src/components/shared/ui/dialog/dialog.tsx

interface DialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  title: string;
  children: React.ReactNode;
}

export function Dialog({ open, onOpenChange, title, children }: DialogProps) {
  if (!open) return null;

  const handleEscape = (e: React.KeyboardEvent) => {
    if (e.key === 'Escape') {
      onOpenChange(false);
    }
  };

  const handleBackdropClick = (e: React.MouseEvent) => {
    if (e.target === e.currentTarget) {
      onOpenChange(false);
    }
  };

  return (
    <div
      className="fixed inset-0 z-50 bg-black/50 flex items-center justify-center"
      onClick={handleBackdropClick}
    >
      <div
        role="dialog"
        aria-modal="true"
        aria-labelledby="dialog-title"
        className="max-sm:fixed max-sm:inset-0 max-sm:z-50 md:max-w-lg bg-white rounded-lg shadow-lg p-6"
        onKeyDown={handleEscape}
      >
        <h2 id="dialog-title" className="text-lg font-semibold mb-4">
          {title}
        </h2>
        {children}
      </div>
    </div>
  );
}
```

### Compliant: Combobox with keyboard navigation and accessibility

```typescript
// src/components/shared/ui/combobox/combobox.tsx

interface ComboboxProps {
  value: string | null;
  onValueChange: (value: string | null) => void;
  options: { value: string; label: string }[];
  placeholder?: string;
}

export function Combobox({
  value,
  onValueChange,
  options,
  placeholder = 'Search...',
}: ComboboxProps) {
  const [isOpen, setIsOpen] = useState(false);
  const [searchQuery, setSearchQuery] = useState('');
  const [activeIndex, setActiveIndex] = useState(0);

  const filtered = options.filter(opt =>
    opt.label.toLowerCase().includes(searchQuery.toLowerCase())
  );

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (!isOpen) return;

    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex(Math.min(activeIndex + 1, filtered.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex(Math.max(activeIndex - 1, 0));
        break;
      case 'Enter':
        e.preventDefault();
        onValueChange(filtered[activeIndex].value);
        setIsOpen(false);
        setSearchQuery('');
        break;
      case 'Escape':
        e.preventDefault();
        setIsOpen(false);
        break;
    }
  };

  const selectedLabel = options.find(opt => opt.value === value)?.label;

  return (
    <div
      role="combobox"
      aria-expanded={isOpen}
      aria-controls="combobox-listbox"
      className="relative"
    >
      {value ? (
        <div className="flex items-center justify-between p-2 border rounded">
          <span>{selectedLabel}</span>
          <button
            onClick={() => {
              onValueChange(null);
              setSearchQuery('');
            }}
          >
            ✕
          </button>
        </div>
      ) : (
        <input
          value={searchQuery}
          onChange={e => setSearchQuery(e.target.value)}
          onFocus={() => setIsOpen(true)}
          onKeyDown={handleKeyDown}
          placeholder={placeholder}
          className="w-full p-2 border rounded"
        />
      )}

      {isOpen && (
        <ul
          id="combobox-listbox"
          role="listbox"
          className="absolute top-full left-0 right-0 mt-1 border rounded bg-white shadow-lg"
          onBlur={() => setIsOpen(false)}
        >
          {filtered.length === 0 ? (
            <li className="p-2 text-gray-500">No results</li>
          ) : (
            filtered.map((opt, idx) => (
              <li
                key={opt.value}
                role="option"
                onClick={() => {
                  onValueChange(opt.value);
                  setIsOpen(false);
                  setSearchQuery('');
                }}
                className={cn(
                  'p-2 cursor-pointer',
                  idx === activeIndex && 'bg-blue-100'
                )}
              >
                {opt.label}
              </li>
            ))
          )}
        </ul>
      )}
    </div>
  );
}
```

### Compliant: Config-driven status badge

```typescript
// src/components/orders/status-badge/status-badge.tsx

type OrderStatus = 'active' | 'inactive' | 'archived';

const STATUS_CONFIG: Record<OrderStatus, { label: string; className: string }> = {
  active: { label: 'Active', className: 'bg-green-100 text-green-800 font-medium' },
  inactive: { label: 'Inactive', className: 'bg-gray-100 text-gray-800 font-medium' },
  archived: { label: 'Archived', className: 'bg-red-100 text-red-800 font-medium' },
};

export function StatusBadge({ status }: { status: OrderStatus }) {
  const config = STATUS_CONFIG[status] ?? { label: 'Unknown', className: 'bg-gray-100' };
  return <span className={`px-3 py-1 rounded-full text-sm ${config.className}`}>{config.label}</span>;
}
```

### Non-Compliant: Business logic in UI component

```typescript
// BAD: UI component fetching data and managing mutations
export function OrderCard({ orderId }: { orderId: string }) {
  const [order, setOrder] = useState(null);

  useEffect(() => {
    // WRONG: UI tier should not fetch data
    apiClient(`/orders/${orderId}`).then(setOrder);
  }, [orderId]);

  const handleDelete = async () => {
    // WRONG: mutation logic in UI component
    await apiClient(`/orders/${orderId}`, { method: 'DELETE' });
    toast.success('Deleted');  // WRONG: toast in UI tier
  };

  return <div>{order?.customer} - <button onClick={handleDelete}>Delete</button></div>;
}
```

### Non-Compliant: Dialog without accessibility attributes

```typescript
// BAD: Missing role, aria-modal, no focus trap
export function EditDialog({ isOpen, onClose }: Props) {
  return (
    <div onClick={onClose} className="fixed inset-0 bg-black/50">
      <div onClick={e => e.stopPropagation()}>
        <h2>Edit Order</h2>  {/* No aria-labelledby */}
        {/* form content */}
      </div>
    </div>
  );
}
```

## Applicability

- **Service types:** frontend
- **Domains:** all domains
- **Exceptions:** Third-party component library wrappers MAY have internal conventions that differ; the wrapper MUST expose a public API that follows this standard.

## References

- STD-004: Component Structure (directory organization and tier definitions)
- STD-010: API Client & React Query Hooks (feature components own the hooks)
- WCAG 2.1 AA: Dialog, Combobox, Alert patterns
