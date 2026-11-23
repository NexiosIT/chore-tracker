---
id: STD-004
name: "Component Structure"
version: "1.0.0"
category: nextjs
status: active
last_reviewed: "2026-04-06"
---

# Component Structure

> Frontend services MUST organize components by domain and tier, enforce design token usage, and maintain a clear separation between feature logic and presentational UI.

## Motivation

Without a consistent component structure, generated UI code scatters components unpredictably. Business logic bleeds into presentational components. Hardcoded colors and spacing values prevent theme changes. A standard structure makes generated code predictable, supports the Component Tree YAML spec format, and ensures design tokens are the single source of visual truth.

## Rules

1. Components MUST be organized by domain:
   ```
   components/{domain}/{component-name}/
   ```
   Shared components used across domains MUST be placed in `components/shared/`.

2. Each component MUST have its own directory containing at minimum:
   - The component file (e.g., `product-card.tsx`, `product-card.vue`)
   - A co-located test file (e.g., `product-card.test.tsx`)
   - Optionally: a styles file, a stories file, and an `index` barrel file

3. Component names MUST use PascalCase in code and kebab-case for directories/files:
   - Component: `ProductCard`
   - Directory: `product-card/`
   - File: `product-card.tsx`

4. Components MUST be categorized into one of four tiers:

   | Tier | Purpose | May Import |
   |------|---------|------------|
   | `page` | Route-level entry point. One per route. | layout, feature, ui |
   | `layout` | Structural wrappers (shells, sidebars, grids) | ui |
   | `feature` | Domain-specific. Contains business logic, data fetching, state. | ui, services, state |
   | `ui` | Pure presentational. No business logic. | other ui components only |

5. `ui` tier components MUST NOT import domain services, API clients, global state stores, or other feature components. They receive all data and callbacks via props.

6. `feature` tier components MUST encapsulate their data fetching and state management. They compose `ui` components and pass data down via props.

7. Components MUST use design tokens for all visual properties:
   - Colors: `token('color.primary.500')`, not `#3B82F6`
   - Spacing: `token('spacing.4')`, not `16px`
   - Typography: `token('font.body.md')`, not `font-size: 14px`
   - Shadows: `token('shadow.md')`, not `box-shadow: 0 4px 6px rgba(0,0,0,0.1)`

   Hardcoded visual values are prohibited.

8. Components MUST declare their props interface/type explicitly. No `any` types, no untyped props, no implicit prop spreading without a defined type.

9. Components that represent a list item MUST be extracted as a separate `ui` component. The list container is a `feature` component; the item is a `ui` component.

## Examples

### Compliant: Directory structure

```
src/
  components/
    orders/                          # domain
      order-list/                    # feature tier
        order-list.tsx
        order-list.test.tsx
        index.ts
      order-card/                    # ui tier
        order-card.tsx
        order-card.test.tsx
        index.ts
    shared/
      ui/
        button/                      # shared ui tier
          button.tsx
          button.test.tsx
          index.ts
        data-table/
          data-table.tsx
          data-table.test.tsx
          index.ts
    layouts/
      main-layout/                   # layout tier
        main-layout.tsx
        index.ts
  pages/
    orders/
      orders-page.tsx                # page tier
```

### Compliant: Feature component composing UI components (pseudocode)

```
// feature tier — owns data fetching and state
component OrderList:
  props: { customerId: string }

  state:
    orders = fetchOrders(props.customerId)
    sortField = "created_at"

  render:
    if orders.loading:
      return <LoadingSpinner />           // shared ui
    if orders.error:
      return <ErrorMessage error={orders.error} />  // shared ui
    return (
      <DataTable                          // shared ui
        columns={ORDER_COLUMNS}
        data={orders.data}
        sortField={sortField}
        onSort={setSortField}
        renderRow={(order) =>
          <OrderCard                      // domain ui
            order={order}
            onSelect={handleSelect}
          />
        }
      />
    )
```

### Compliant: UI component with design tokens (pseudocode)

```
// ui tier — pure presentational, all data via props
component OrderCard:
  props:
    order: Order
    onSelect: (id: string) => void

  styles:
    container:
      padding: token('spacing.4')
      background: token('color.surface.primary')
      borderRadius: token('radius.md')
      boxShadow: token('shadow.sm')
    title:
      font: token('font.heading.sm')
      color: token('color.text.primary')
    amount:
      font: token('font.body.md')
      color: token('color.text.secondary')

  render:
    <div style={styles.container} onClick={() => props.onSelect(props.order.id)}>
      <h3 style={styles.title}>{props.order.title}</h3>
      <span style={styles.amount}>{formatCurrency(props.order.total)}</span>
    </div>
```

### Non-Compliant: UI component with business logic

```
// BAD: ui tier component fetches data and uses hardcoded values
component OrderCard:
  state:
    order = fetchOrder(props.orderId)     // ui tier must NOT fetch data

  styles:
    container:
      padding: "16px"                      // hardcoded, must use token
      background: "#ffffff"                // hardcoded, must use token
      color: "#1a1a1a"                     // hardcoded, must use token

  render:
    <div style={styles.container}>
      <h3>{order.title}</h3>
    </div>
```

### Non-Compliant: Flat component structure

```
// BAD: Components dumped flat without domain grouping
src/
  components/
    OrderList.tsx
    OrderCard.tsx
    ProductGrid.tsx
    Button.tsx
    UserAvatar.tsx
```

## Applicability

- **Service types:** frontend
- **Domains:** all
- **Exceptions:** Third-party component library wrappers MAY live in `components/shared/vendor/` and follow the library's conventions internally, but MUST expose a props interface that follows this standard.

## References

- STD-003: Naming Conventions (file and directory naming)
- STD-005: Performance Budgets (virtualization and rendering thresholds affect component design)
