---
id: STD-013
name: "Page & Routing Structure"
version: "1.1.0"
category: nextjs
status: active
last_reviewed: "2026-07-18"
discovery_layer: "generation-extracted"
---

# Page & Routing Structure

> Frontend services using Next.js App Router MUST structure pages as client-side components, delegate business logic to feature components, organize supporting code by domain and concern, and use responsive patterns that adapt layouts from desktop to mobile.

> **Styling & realization.** Design *values* — colour, spacing, radius, typography — MUST bind to design-system tokens as CSS custom properties (`var(--token)`), never hardcoded. The styling *mechanism* (the Tailwind utility classes and `md:`/`max-sm:` responsive prefixes shown in the examples below) is a realization decision owned by the design system, not mandated here, and is **under active redesign** — a hub using CSS Modules with `@media` queries bound to `var(--token)` (the current generation default) is equally compliant. Read the examples for the **structural** patterns (page composition, feature-component delegation, file organization, responsive behaviour), not for the styling approach.

## Motivation

Without a consistent page structure, routes become hard to trace back to specifications, business logic scatters across components, and team members struggle to find related code. A standard page structure ensures specs are discoverable, clear boundaries between page entry points and feature logic, and responsive behavior is explicit and testable.

## Rules

### Page Components (Next.js App Router)

1. Page components MUST be marked with the `"use client"` directive to enable React state and interactivity:
   ```typescript
   "use client";
   ```

2. Page components MUST NOT open with spec-mapping comment blocks (Page ID / Route / Sections). The route is already expressed by the file path; the screen-spec mapping is owned by the Traceability Matrix (STD-032).

3. Page components MUST follow this structure:
   - Page header (back link for detail pages, title + action button for list pages)
   - Content container with responsive max-width
   - Suspense boundary wrapping async content
   - Feature component (never business logic directly in the page)

4. Page headers on detail pages (e.g., edit, view) MUST include a back button/link:
   ```typescript
   <Link href="/orders" className="text-blue-600 hover:underline">
     ← Back to Orders
   </Link>
   <h1>Edit Order</h1>
   ```

5. Page headers on list pages MUST include the resource title and a primary action button:
   ```typescript
   <div className="flex items-center justify-between mb-6">
     <h1>Orders</h1>
     <button onClick={() => navigate('/orders/new')}>Add Order</button>
   </div>
   ```

6. Page content MUST be wrapped in a container with responsive max-width:
   - Form pages: `max-w-2xl`
   - List pages: `max-w-7xl`
   - Detail pages (view-only): `max-w-4xl`

7. Async content MUST be wrapped in a Suspense boundary:
   ```typescript
   <Suspense fallback={<LoadingSpinner />}>
     <OrderDetailFeature orderId={params.id} />
   </Suspense>
   ```

8. Page components MUST NOT contain business logic (data fetching, mutations, validation). All logic MUST be delegated to feature components.

### File Organization

9. Feature components MUST be organized in `src/components/{domain}/{component-name}/` with an `index.ts` barrel export:
   ```
   src/components/
     orders/
       order-list/
         order-list.tsx
         order-list.test.tsx
         index.ts
       order-form/
         order-form.tsx
         order-form.test.tsx
         index.ts
   ```

10. Types MUST be organized by domain in `src/lib/types/{domain}.ts`:
    ```typescript
    // src/lib/types/order.ts
    export interface Order { ... }
    export interface CreateOrderRequest { ... }
    export interface UpdateOrderRequest { ... }
    ```

11. API functions MUST be organized by domain in `src/lib/api/{domain}.ts`:
    ```typescript
    // src/lib/api/orders.ts
    export async function getOrders() { ... }
    export async function createOrder() { ... }
    export async function updateOrder() { ... }
    ```

12. Hooks MUST be organized in `src/lib/hooks/use-{operation}.ts`:
    ```
    src/lib/hooks/
      use-orders.ts
      use-order.ts
      use-create-order.ts
      use-update-order.ts
      use-delete-order.ts
    ```

13. Validation functions MUST be organized by domain in `src/lib/validation/{domain}-validation.ts`:
    ```typescript
    // src/lib/validation/order-validation.ts
    export function validateCreateOrderForm() { ... }
    export function validateUpdateOrderForm() { ... }
    ```

### Responsive Design

14. Tables (data-heavy components) MUST have:
    - Desktop: semantic `<table>` (visible by default)
    - Mobile: card layout (visible only on mobile, hidden above `md` breakpoint)

    ```typescript
    <table className="hidden md:table w-full">
      <thead><tr>...</tr></thead>
      <tbody>{rows}</tbody>
    </table>

    <div className="md:hidden space-y-4">
      {orders.map(v => <OrderCard key={v.id} order={v} />)}
    </div>
    ```

15. Dialogs and modals MUST be full-screen on mobile:
    ```typescript
    <dialog className="max-sm:fixed max-sm:inset-0 md:max-w-lg">
      {/* content */}
    </dialog>
    ```

16. Progressive disclosure: less important columns on tables MUST be hidden below certain breakpoints:
    ```typescript
    <th className="hidden lg:table-cell">Created At</th>
    <th className="hidden md:table-cell">Status</th>
    ```

17. Content spacing MUST adapt: use responsive padding and margins for smaller screens:
    ```typescript
    <div className="p-4 md:p-6 lg:p-8">
      {/* content */}
    </div>
    ```

## Examples

### Compliant: List page with feature component delegation

```typescript
// app/orders/page.tsx

"use client";

import { useState } from "react";
import Link from "next/link";
import { OrderListFeature } from "@/components/orders/order-list";
import { Button } from "@/components/shared/ui/button";

export default function OrdersPage() {
  return (
    <div className="min-h-screen bg-gray-50 p-4 md:p-8">
      <div className="max-w-7xl mx-auto">
        <div className="flex items-center justify-between mb-8">
          <h1 className="text-3xl font-bold">Orders</h1>
          <Link href="/orders/new">
            <Button>Add Order</Button>
          </Link>
        </div>

        <OrderListFeature />
      </div>
    </div>
  );
}
```

### Compliant: Detail page with back link and Suspense

```typescript
// app/orders/[id]/page.tsx

"use client";

import { Suspense } from "react";
import Link from "next/link";
import { EditOrderFeature } from "@/components/orders/edit-order";
import { LoadingSpinner } from "@/components/shared/ui/loading-spinner";

interface EditOrderPageProps {
  params: { id: string };
}

export default function EditOrderPage({ params }: EditOrderPageProps) {
  return (
    <div className="min-h-screen bg-gray-50 p-4 md:p-8">
      <div className="max-w-2xl mx-auto">
        <Link href="/orders" className="inline-block text-blue-600 hover:underline mb-6">
          ← Back to Orders
        </Link>

        <h1 className="text-3xl font-bold mb-8">Edit Order</h1>

        <Suspense fallback={<LoadingSpinner />}>
          <EditOrderFeature orderId={params.id} />
        </Suspense>
      </div>
    </div>
  );
}
```

### Compliant: File organization structure

```
src/
  app/
    orders/
      page.tsx                 # List page
      layout.tsx
      [id]/
        page.tsx               # Detail page
  components/
    orders/                  # Domain: orders
      order-list/
        order-list.tsx       # Feature component
        order-list.test.tsx
        index.ts
      order-form/
        order-form.tsx       # Feature component
        order-form.test.tsx
        index.ts
      order-card/
        order-card.tsx       # UI component
        order-card.test.tsx
        index.ts
    shared/
      ui/
        button/
        dialog/
        form-field/
  lib/
    api/
      orders.ts              # API functions for orders domain
    hooks/
      use-orders.ts          # Query hook
      use-order.ts           # Single order query hook
      use-create-order.ts    # Mutation hook
      use-update-order.ts    # Mutation hook
      use-delete-order.ts    # Mutation hook
    types/
      order.ts               # Order types
    validation/
      order-validation.ts    # Order validation functions
```

### Compliant: Responsive table with mobile card layout

```typescript
// src/components/orders/order-list/order-list.tsx

"use client";

import { Order } from "@/lib/types/order";
import { OrderCard } from "../order-card";

interface OrderListUIProps {
  orders: Order[];
  isLoading: boolean;
  onEdit: (id: string) => void;
  onDelete: (id: string) => void;
}

export function OrderListUI({
  orders,
  isLoading,
  onEdit,
  onDelete,
}: OrderListUIProps) {
  if (isLoading) return <div>Loading...</div>;

  return (
    <>
      <table className="hidden md:table w-full border-collapse border border-gray-300">
        <thead>
          <tr className="bg-gray-100">
            <th className="border p-3 text-left">Quantity</th>
            <th className="border p-3 text-left">Customer</th>
            <th className="border p-3 text-left">Product</th>
            <th className="border p-3 text-left hidden lg:table-cell">Reference</th>
            <th className="border p-3 text-left">Status</th>
            <th className="border p-3 text-center w-24">Actions</th>
          </tr>
        </thead>
        <tbody>
          {orders.map(o => (
            <tr key={o.id} className="hover:bg-gray-50">
              <td className="border p-3">{o.quantity}</td>
              <td className="border p-3">{o.customer}</td>
              <td className="border p-3">{o.product}</td>
              <td className="border p-3 hidden lg:table-cell">{o.reference}</td>
              <td className="border p-3">{o.status}</td>
              <td className="border p-3 text-center">
                <button onClick={() => onEdit(o.id)} className="text-blue-600">
                  Edit
                </button>
                <button onClick={() => onDelete(o.id)} className="text-red-600 ml-2">
                  Delete
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      <div className="md:hidden space-y-4">
        {orders.map(o => (
          <OrderCard
            key={o.id}
            order={o}
            onEdit={() => onEdit(o.id)}
            onDelete={() => onDelete(o.id)}
          />
        ))}
      </div>
    </>
  );
}
```

### Compliant: Responsive dialog with mobile full-screen

```typescript
// src/components/orders/edit-order/edit-order-dialog.tsx

interface EditOrderDialogProps {
  open: boolean;
  orderId: string;
  onClose: () => void;
}

export function EditOrderDialog({
  open,
  orderId,
  onClose,
}: EditOrderDialogProps) {
  return (
    <dialog
      open={open}
      className="max-sm:fixed max-sm:inset-0 max-sm:z-50 md:max-w-lg md:rounded-lg"
      onClick={e => {
        if (e.target === e.currentTarget) onClose();
      }}
    >
      <div className="max-sm:p-4 md:p-6">
        <h2 className="text-2xl font-bold mb-4">Edit Order</h2>
        <EditOrderFeature orderId={orderId} onClose={onClose} />
      </div>
    </dialog>
  );
}
```

### Non-Compliant: Business logic in page component

```typescript
// BAD: Page component doing data fetching and mutations
export default function OrdersPage() {
  const [orders, setOrders] = useState([]);
  const [filters, setFilters] = useState({});

  useEffect(() => {
    // WRONG: data fetching in page
    fetch('/api/orders').then(r => r.json()).then(setOrders);
  }, [filters]);

  const handleDelete = async (id: string) => {
    // WRONG: mutations in page
    await fetch(`/api/orders/${id}`, { method: 'DELETE' });
    setOrders(orders.filter(o => o.id !== id));
    toast.success('Deleted');  // WRONG: side effects in page
  };

  return (
    <table>
      {orders.map(o => (
        <tr key={o.id}>
          <td>{o.customer}</td>
          <td><button onClick={() => handleDelete(o.id)}>Delete</button></td>
        </tr>
      ))}
    </table>
  );
}
```

### Non-Compliant: Flat file organization

```typescript
// BAD: No domain organization, files scattered
src/
  Order.tsx               # What type of component?
  OrderList.tsx
  OrderForm.tsx
  orderApi.ts
  useOrders.ts
  validateOrder.ts
```

### Non-Compliant: No responsive table implementation

```typescript
// BAD: Desktop-only table, no mobile layout
export function OrderTable({ orders }: Props) {
  return (
    <table>
      {/* only renders on desktop, unusable on mobile */}
      <thead><tr><th>...</th></tr></thead>
      <tbody>
        {orders.map(o => <tr key={o.id}><td>{o.customer}</td></tr>)}
      </tbody>
    </table>
  );
}
```

### Non-Compliant: Dialog without mobile responsiveness

```typescript
// BAD: No mobile-specific styling
<dialog className="max-w-lg">
  {/* Fixed width on mobile, may overflow screen */}
  <EditOrderForm />
</dialog>
```

## Applicability

- **Service types:** frontend (Next.js App Router projects)
- **Domains:** all
- **Exceptions:** Static pages (e.g., landing pages) may omit feature components and manage minimal state. SSR-only pages may delegate differently; still MUST follow the file organization and responsive patterns.

## References

- STD-004: Component Structure (page tier definition)
- STD-012: Component Patterns (feature vs. UI tier composition)
- Next.js App Router documentation (routing, layouts, Suspense)
