---
id: STD-011
name: "Form State & Validation Patterns"
version: "1.1.0"
category: nextjs
status: active
last_reviewed: "2026-07-18"
discovery_layer: "generation-extracted"
---

# Form State & Validation Patterns

> Frontend services MUST manage form state with a single source of truth for values and errors, validate client-side before submission with user-facing messages, and handle server errors by merging field-level errors back into the form without clearing validation state prematurely.

> **Styling & realization.** Design *values* — colour, spacing, radius, typography — MUST bind to design-system tokens as CSS custom properties (`var(--token)`), never hardcoded. The styling *mechanism* (the Tailwind utility classes in the `FormField` example below) is a realization decision owned by the design system, not mandated here, and is **under active redesign** — a hub using CSS Modules bound to `var(--token)` (the current generation default) is equally compliant. This standard governs form **state, validation, and accessibility**, not the styling approach.

## Motivation

Inconsistent form handling introduces bugs: state scattered across multiple useState calls, validation logic duplicated across features, unclear error messaging, and server errors overwriting client validation. A standard form pattern ensures predictable state management, consistent validation, proper error recovery, and accessibility compliance for error display.

## Rules

### Form State

1. Form values MUST be typed as `Partial<{RequestType}>` to allow gradual updates:
   ```typescript
   type FormValues = Partial<CreateOrderRequest>;
   const [values, setValues] = useState<FormValues>({});
   ```

2. Form errors MUST be stored in a separate state as `Record<string, string>`:
   ```typescript
   const [errors, setErrors] = useState<Record<string, string>>({});
   ```

3. Field updates MUST clear the error for that field:
   ```typescript
   const updateField = (field: string, value: any) => {
     setValues(prev => ({ ...prev, [field]: value }));
     setErrors(prev => {
       const newErrors = { ...prev };
       delete newErrors[field];
       return newErrors;
     });
   };
   ```

4. Form state setup MUST use a single `updateField` callback, not separate handlers per field.

### Validation

5. Validation functions MUST return `FieldError[]` (array of field + reason pairs), not booleans or error strings:
   ```typescript
   export interface FieldError {
     field: string;
     reason: string;
   }

   export function validateCreateOrderForm(values: Partial<CreateOrderRequest>): FieldError[]
   ```

6. Validation MUST follow this order per field:
   1. Check required (if no value, return error immediately)
   2. Check format/pattern (e.g., email regex, phone format)
   3. Check range or length constraints
   4. Check business logic constraints (e.g., VAT registration rules) — ONLY if prior checks pass

7. Validation messages MUST be user-facing, not technical codes:
   ```typescript
   // Compliant
   { field: 'email', reason: 'Please enter a valid email address' }
   // Non-compliant
   { field: 'email', reason: 'INVALID_EMAIL_FORMAT' }
   ```

8. Provide a `fieldErrorMap()` utility function that converts `FieldError[]` to `Record<string, string>`:
   ```typescript
   export function fieldErrorMap(errors: FieldError[]): Record<string, string> {
     return errors.reduce((acc, err) => {
       acc[err.field] = err.reason;
       return acc;
     }, {} as Record<string, string>);
   }
   ```

9. Business logic validation (e.g., VAT checks, duplicate detection) MUST be separate from form validation. These are server-side concerns that happen after client validation passes.

### Submission Flow

10. Form submission MUST follow this sequence:
    1. `preventDefault()` on the form event
    2. Run client-side validation, get `FieldError[]`
    3. If errors exist, convert to `Record<string, string>` via `fieldErrorMap()`, call `setErrors()`, and return early
    4. Call `mutateAsync(values)` from the mutation hook
    5. On success: call `toast.success()`, then `reset()` on the hook, then `navigate()` if needed
    6. On error: parse server field errors and merge into `setErrors()` (do NOT clear client errors)

11. Confirmation dialogs for destructive actions (delete, clear) MUST appear BEFORE the API call is made, not after.

12. After successful submission, the hook's `reset()` method MUST be called to clear both `data` and `fieldErrors`.

### Form Field Wrapper Component

13. A `FormField` wrapper component MUST be provided that encapsulates:
    - Label (with required indicator if applicable)
    - Input or other control
    - Error message (with `role="alert"` and `aria-live="assertive"`)
    - Help text (hidden when error is present)

14. Error messages MUST have `role="alert"` and `aria-live="assertive"` to be announced to screen readers.

15. Help text MUST be hidden when an error is present, using CSS or conditional rendering.

16. Required indicators MUST be applied via the Label component, not duplicated in each FormField.

## Examples

### Compliant: Form state with updateField callback

```typescript
// src/components/orders/create-order/create-order.tsx

interface CreateOrderFormValues extends Partial<CreateOrderRequest> {}

export function CreateOrderForm() {
  const [values, setValues] = useState<CreateOrderFormValues>({});
  const [errors, setErrors] = useState<Record<string, string>>({});
  const { mutateAsync, isPending, reset } = useCreateOrder();

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

    // Client-side validation
    const validationErrors = validateCreateOrderForm(values);
    if (validationErrors.length > 0) {
      setErrors(fieldErrorMap(validationErrors));
      return;
    }

    try {
      await mutateAsync(values as CreateOrderRequest);
      toast.success('Order created successfully');
      reset();
      navigate('/orders');
    } catch (error) {
      // Merge server errors into form
      if (error instanceof ApiClientError) {
        const serverErrors = extractErrorInfo(error);
        setErrors(prev => ({
          ...prev,
          ...fieldErrorMap(serverErrors),
        }));
      }
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <FormField
        label="Customer"
        required
        error={errors.customer}
        helpText="Account the order is placed for"
      >
        <Input
          value={values.customer || ''}
          onChange={e => updateField('customer', e.target.value)}
        />
      </FormField>

      <FormField
        label="Product"
        required
        error={errors.product}
      >
        <Input
          value={values.product || ''}
          onChange={e => updateField('product', e.target.value)}
        />
      </FormField>

      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Order'}
      </button>
    </form>
  );
}
```

### Compliant: Validation function returning FieldError[]

```typescript
// src/lib/validation/order-validation.ts

export interface FieldError {
  field: string;
  reason: string;
}

export function validateCreateOrderForm(
  values: Partial<CreateOrderRequest>
): FieldError[] {
  const errors: FieldError[] = [];

  // Required checks
  if (!values.customer || values.customer.trim() === '') {
    errors.push({ field: 'customer', reason: 'Customer is required' });
  }
  if (!values.product || values.product.trim() === '') {
    errors.push({ field: 'product', reason: 'Product is required' });
  }
  if (!values.quantity) {
    errors.push({ field: 'quantity', reason: 'Quantity is required' });
  }

  // Format checks (only if value provided)
  if (values.reference && !/^[A-Z0-9]{10}$/.test(values.reference)) {
    errors.push({ field: 'reference', reason: 'Reference must be 10 characters (A-Z, 0-9)' });
  }

  // Range checks
  if (values.quantity && (values.quantity < 1 || values.quantity > 999)) {
    errors.push({
      field: 'quantity',
      reason: 'Quantity must be between 1 and 999',
    });
  }

  return errors;
}

export function fieldErrorMap(errors: FieldError[]): Record<string, string> {
  return errors.reduce((acc, err) => {
    acc[err.field] = err.reason;
    return acc;
  }, {} as Record<string, string>);
}
```

### Compliant: FormField wrapper component

```typescript
// src/components/shared/ui/form-field/form-field.tsx

interface FormFieldProps {
  label: string;
  required?: boolean;
  error?: string;
  helpText?: string;
  children: React.ReactNode;
}

export function FormField({
  label,
  required = false,
  error,
  helpText,
  children,
}: FormFieldProps) {
  return (
    <div className="mb-4">
      <label className="block text-sm font-medium mb-1">
        {label}
        {required && <span className="text-red-500 ml-1">*</span>}
      </label>
      <div className={error ? 'ring-2 ring-red-500 rounded' : ''}>
        {children}
      </div>
      {error && (
        <div role="alert" aria-live="assertive" className="text-red-600 text-sm mt-1">
          {error}
        </div>
      )}
      {!error && helpText && (
        <div className="text-gray-500 text-sm mt-1">{helpText}</div>
      )}
    </div>
  );
}
```

### Compliant: Destructive action with confirmation

```typescript
// src/components/orders/delete-order/delete-order.tsx

export function DeleteOrderDialog({ orderId, onClose }: Props) {
  const [confirmed, setConfirmed] = useState(false);
  const { mutateAsync, isPending } = useDeleteOrder();

  const handleDelete = async () => {
    if (!confirmed) {
      setConfirmed(true);
      return;
    }

    try {
      await mutateAsync(orderId);
      toast.success('Order deleted');
      navigate('/orders');
    } catch (error) {
      toast.error('Failed to delete order');
    }
  };

  return (
    <Dialog open onOpenChange={onClose}>
      <DialogContent>
        {!confirmed ? (
          <>
            <DialogTitle>Delete Order?</DialogTitle>
            <DialogDescription>This cannot be undone.</DialogDescription>
            <DialogFooter>
              <button onClick={onClose}>Cancel</button>
              <button onClick={handleDelete}>Continue</button>
            </DialogFooter>
          </>
        ) : (
          <>
            <DialogTitle>Confirm Deletion</DialogTitle>
            <DialogDescription>
              Are you absolutely sure? This will permanently delete the order.
            </DialogDescription>
            <DialogFooter>
              <button onClick={onClose}>Cancel</button>
              <button onClick={handleDelete} disabled={isPending}>
                {isPending ? 'Deleting...' : 'Delete'}
              </button>
            </DialogFooter>
          </>
        )}
      </DialogContent>
    </Dialog>
  );
}
```

### Non-Compliant: State scattered across multiple useState calls

```typescript
// BAD: Each field has its own state — hard to manage
const [customer, setCustomer] = useState('');
const [product, setProduct] = useState('');
const [customerError, setCustomerError] = useState('');
const [productError, setProductError] = useState('');
// ... repeats for every field
```

### Non-Compliant: Validation with technical error codes

```typescript
// BAD: Error messages are codes, not user-facing
function validateForm(values) {
  const errors = [];
  if (!values.customer) {
    errors.push('CUSTOMER_REQUIRED');  // Not user-facing
  }
  if (!/^[0-9]+$/.test(values.quantity)) {
    errors.push('QUANTITY_FORMAT_ERR');  // Technical code
  }
  return errors;
}
```

### Non-Compliant: Business logic in form validation

```typescript
// BAD: VAT checks belong on the server, not in form validation
export function validateOrderForm(values) {
  const errors = [];
  // ... required and format checks ...

  // WRONG: This is a server-side concern
  const vatValid = await checkVatRegistration(values.vatId);
  if (!vatValid) {
    errors.push({ field: 'vatId', reason: 'VAT not registered' });
  }

  return errors;
}
```

### Non-Compliant: Form submission clearing errors on server error

```typescript
// BAD: Clearing errors before attempting submission
const handleSubmit = async (e) => {
  e.preventDefault();
  setErrors({});  // WRONG: cleared too early

  const validationErrors = validateForm(values);
  if (validationErrors.length > 0) {
    setErrors(fieldErrorMap(validationErrors));
    return;
  }

  try {
    await mutateAsync(values);
  } catch (error) {
    // Server errors override validation errors instead of merging
    setErrors(fieldErrorMap(extractErrorInfo(error)));
  }
};
```

### Non-Compliant: Toast in hook instead of feature component

```typescript
// BAD: Hook handles the toast, not the feature component
export function useCreateOrder() {
  return useMutation({
    mutationFn: createOrder,
    onSuccess: () => {
      toast.success('Created!');  // WRONG: belongs in component
    },
  });
}
```

## Applicability

- **Service types:** frontend
- **Domains:** all domains with user forms
- **Exceptions:** Server-side rendering scenarios may validate on the server and return errors directly; client-side validation still MUST follow this pattern for progressive enhancement.

## References

- STD-010: API Client & React Query Hooks (mutations expose fieldErrors)
- STD-012: Component Patterns (feature components own form state)
- WCAG 2.1 AA: Error Identification, Error Suggestion (aria-live, role="alert")
