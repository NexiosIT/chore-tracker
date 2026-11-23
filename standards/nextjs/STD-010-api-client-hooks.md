---
id: STD-010
name: "API Client & React Query Hooks"
version: "1.0.0"
category: nextjs
status: active
last_reviewed: "2026-06-03"
discovery_layer: "generation-extracted"
---

# API Client & React Query Hooks

> Frontend services MUST structure API clients with automatic request/response conversion, use React Query for data fetching and mutations with consistent error handling, and export typed hook results with utility functions for error parsing.

## Motivation

Inconsistent API communication patterns introduce bugs: missing snake_case/camelCase conversion, unhandled errors, cache invalidation issues, and hook interfaces that leak implementation details. A standard client and hook structure ensures all API calls are type-safe, errors are consistently parsed and exposed to UI components, cache behavior is predictable, and mutations properly invalidate related queries.

## Rules

### API Client

1. The `apiClient<T>` function MUST accept a URL and options, perform the request, and convert response body from snake_case to camelCase:
   ```typescript
   async function apiClient<T>(
     url: string,
     options: RequestInit = {}
   ): Promise<T>
   ```

2. Request bodies MUST be converted from camelCase to snake_case before sending:
   ```typescript
   const body = convertKeysToSnakeCase(options.body);
   ```

3. Response bodies MUST be converted from snake_case to camelCase after receiving:
   ```typescript
   const data = convertKeysFromSnakeCaseToJS(responseJson);
   ```

4. Non-200 responses MUST throw `ApiClientError`:
   ```typescript
   if (!response.ok) {
     throw new ApiClientError(status, errorData);
   }
   ```

5. Service functions MUST be organized by domain (e.g., `orders.ts`, `contacts.ts`) and export individual functions for each operation:
   ```typescript
   export async function getOrders(params: OrderListParams): Promise<Order[]>
   export async function createOrder(body: CreateOrderRequest): Promise<Order>
   export async function updateOrder(id: string, body: UpdateOrderRequest): Promise<Order>
   ```

### Mutation Hooks

6. Mutation hooks MUST expose these properties on the result object:
   - `mutate(variables, options?)` — synchronous fire-and-forget
   - `mutateAsync(variables)` — async, returns promise
   - `isPending` — true while request in flight
   - `isSuccess` — true after successful mutation
   - `data` — the response data
   - `reset()` — clear state (called after navigation)

7. Export the hook result as a TypeScript interface:
   ```typescript
   export interface Use{Operation}Result {
     mutate: (variables: Variables) => void;
     mutateAsync: (variables: Variables) => Promise<Data>;
     isPending: boolean;
     isSuccess: boolean;
     data?: Data;
     fieldErrors: ErrorDetail[];
     reset: () => void;
   }
   ```

8. Field-level errors MUST be extracted via `fieldErrors: ErrorDetail[]` property on the result:
   ```typescript
   export interface ErrorDetail {
     field: string;
     reason: string;
   }
   ```

9. Provide a static `extractErrorInfo(error: ApiClientError): ErrorDetail[]` helper function that parses the API error response and returns an array of field errors.

10. Mutations MUST invalidate related queries on success (e.g., after creating an order, invalidate the orders list query):
    ```typescript
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['orders'] });
    }
    ```

11. Mutation hooks MUST NOT handle toast notifications. The feature component consuming the hook is responsible for calling `toast()`.

### Query Hooks

12. Query hooks MUST use stable queryKeys as arrays with the resource name first, followed by optional parameters:
    ```typescript
    const queryKey = ['orders', { sortBy: 'created_at', status: 'active' }];
    const { data, isLoading, error } = useQuery({
      queryKey,
      queryFn: () => getOrders(queryKey[1])
    });
    ```

13. List queries MUST use `placeholderData: keepPreviousData` to avoid flickering during re-fetch:
    ```typescript
    useQuery({
      queryKey: ['orders', filters],
      queryFn: () => getOrders(filters),
      placeholderData: keepPreviousData
    })
    ```

14. Query hooks MUST set `staleTime: 30_000` (30 seconds) by default for all query types.

15. Export a separate error message function with signature `get{Resource}ErrorMessage(error: Error | null): string` that returns a user-facing error description.

### Search Hooks

16. Search hooks MUST debounce the query string at 300ms before sending:
    ```typescript
    const debouncedQuery = useDebounce(query, 300);
    ```

17. Search hooks MUST enforce a minimum of 2 characters before querying:
    ```typescript
    const MIN_QUERY_LENGTH = 2;
    enabled: debouncedQuery.length >= MIN_QUERY_LENGTH
    ```

18. Search query keys MUST be `[resource, debouncedQuery]` with the debounced string as the second element.

19. Search queries MUST use `staleTime: 30_000` and include the `enabled` condition checking minimum length.

### Contract Types

20. Request/response types MUST be **generated from the consumed contract, not hand-written**. Run `openapi-typescript` (a prebuild step) over each upstream contract in the materialized snapshot `.flowforge/contracts/{UC}/contract.openapi.yml` to emit camelCase TypeScript types + a typed `paths` map. Treat the output as generated — do not hand-edit it.

    The prebuild script (`scripts/gen-types.mjs`) MUST **dynamically scan** `.flowforge/contracts/` to discover which UCs have contracts present, rather than maintain a hardcoded list. Required pattern:

    ```js
    import { execSync } from "child_process";
    import { existsSync, mkdirSync, readdirSync } from "fs";

    const contractsDir = ".flowforge/contracts";
    const ucs = existsSync(contractsDir)
      ? readdirSync(contractsDir).filter(d =>
          existsSync(`${contractsDir}/${d}/contract.openapi.yml`)
        )
      : [];

    mkdirSync("lib/types/generated", { recursive: true });

    for (const uc of ucs) {
      const input = `${contractsDir}/${uc}/contract.openapi.yml`;
      const output = `lib/types/generated/${uc.toLowerCase()}.ts`;
      execSync(`npx openapi-typescript ${input} --output ${output}`, { stdio: "inherit" });
    }
    ```

21. The `apiClient<T>` (Rule 1) and the per-domain service functions (Rule 5) MUST be typed against the generated types — request body, query params, and the `Promise<T>` return reference the generated request/response types (and the `paths` map for path/method/status where used). This makes the consumer's API surface conform to the contract **at compile time**: a drifted field or a call to a non-existent operation is a `tsc` error. This compile-time check **is** the frontend's code-to-contract conformance (STD-024) — the producer owns runtime validation.

22. The generated types describe the **camelCase** `data` payload (the contract's shape). The snake↔camel conversion (Rules 2–3) maps the wire to/from these types at the client edge; assert that round-trip **once** (a single client-level test), not per call.

## Examples

### Compliant: API Client with conversion

```typescript
// src/lib/api/client.ts

export class ApiClientError extends Error {
  constructor(
    public status: number,
    public data: { error?: string; errors?: Record<string, string[]> }
  ) {
    super(data.error || 'API request failed');
  }
}

async function apiClient<T>(
  url: string,
  options: RequestInit = {}
): Promise<T> {
  const headers = {
    'Content-Type': 'application/json',
    ...options.headers,
  };

  const body = options.body
    ? JSON.stringify(convertKeysToSnakeCase(JSON.parse(options.body as string)))
    : undefined;

  const response = await fetch(url, { ...options, headers, body });

  if (!response.ok) {
    const errorData = await response.json();
    throw new ApiClientError(response.status, errorData);
  }

  const responseJson = await response.json();
  return convertKeysFromSnakeCaseToJS(responseJson) as T;
}

// src/lib/api/orders.ts

export async function getOrders(params: OrderListParams): Promise<Order[]> {
  return apiClient<Order[]>(`/api/v1/orders?${new URLSearchParams(params)}`);
}

export async function createOrder(body: CreateOrderRequest): Promise<Order> {
  return apiClient<Order>('/api/v1/orders', {
    method: 'POST',
    body: JSON.stringify(body),
  });
}

export async function updateOrder(
  id: string,
  body: UpdateOrderRequest
): Promise<Order> {
  return apiClient<Order>(`/api/v1/orders/${id}`, {
    method: 'PATCH',
    body: JSON.stringify(body),
  });
}
```

### Compliant: Mutation hook with error extraction and query invalidation

```typescript
// src/lib/hooks/use-create-order.ts

export interface ErrorDetail {
  field: string;
  reason: string;
}

export interface UseCreateOrderResult {
  mutate: (variables: CreateOrderRequest) => void;
  mutateAsync: (variables: CreateOrderRequest) => Promise<Order>;
  isPending: boolean;
  isSuccess: boolean;
  data?: Order;
  fieldErrors: ErrorDetail[];
  reset: () => void;
}

export function useCreateOrder(): UseCreateOrderResult {
  const queryClient = useQueryClient();
  const [fieldErrors, setFieldErrors] = useState<ErrorDetail[]>([]);

  const mutation = useMutation({
    mutationFn: createOrder,
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['orders'] });
      setFieldErrors([]);
    },
    onError: (error) => {
      if (error instanceof ApiClientError) {
        setFieldErrors(extractErrorInfo(error));
      }
    },
  });

  return {
    mutate: mutation.mutate,
    mutateAsync: mutation.mutateAsync,
    isPending: mutation.isPending,
    isSuccess: mutation.isSuccess,
    data: mutation.data,
    fieldErrors,
    reset: () => {
      mutation.reset();
      setFieldErrors([]);
    },
  };
}

export function extractErrorInfo(error: ApiClientError): ErrorDetail[] {
  if (error.data.errors) {
    return Object.entries(error.data.errors).map(([field, messages]) => ({
      field,
      reason: messages[0] || 'Invalid input',
    }));
  }
  return [];
}
```

### Compliant: Query hook with placeholderData

```typescript
// src/lib/hooks/use-orders.ts

export function useOrders(filters: OrderListParams) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['orders', filters],
    queryFn: () => getOrders(filters),
    placeholderData: keepPreviousData,
    staleTime: 30_000,
  });

  return { data, isLoading, error };
}

export function getOrdersErrorMessage(error: Error | null): string {
  if (!error) return '';
  if (error instanceof ApiClientError && error.status === 404) {
    return 'Orders not found.';
  }
  return 'Failed to load orders. Please try again.';
}
```

### Compliant: Search hook with debounce and minimum length

```typescript
// src/lib/hooks/use-search-orders.ts

const MIN_QUERY_LENGTH = 2;
const DEBOUNCE_MS = 300;

export function useSearchOrders(query: string) {
  const debouncedQuery = useDebounce(query, DEBOUNCE_MS);

  const { data, isLoading, error } = useQuery({
    queryKey: ['orders-search', debouncedQuery],
    queryFn: () => searchOrders(debouncedQuery),
    staleTime: 30_000,
    enabled: debouncedQuery.length >= MIN_QUERY_LENGTH,
  });

  return { data, isLoading, error };
}

async function searchOrders(query: string): Promise<Order[]> {
  return apiClient<Order[]>(`/api/v1/orders/search?q=${encodeURIComponent(query)}`);
}
```

### Non-Compliant: Manual request/response conversion

```typescript
// BAD: no automatic snake_case/camelCase conversion
export async function getOrders(): Promise<Order[]> {
  const response = await fetch('/api/v1/orders');
  const json = await response.json();
  // Response comes back as snake_case, no conversion applied
  // Field names mismatch: order_id instead of orderId
  return json;
}
```

### Non-Compliant: Mutation hook handling toast

```typescript
// BAD: mutation hook should NOT call toast
export function useCreateOrder() {
  return useMutation({
    mutationFn: createOrder,
    onSuccess: () => {
      toast.success('Order created!');  // WRONG: belongs in feature component
    },
  });
}
```

### Non-Compliant: Query without proper cache invalidation

```typescript
// BAD: creating an order doesn't invalidate list
export function useCreateOrder() {
  return useMutation({
    mutationFn: createOrder,
    onSuccess: () => {
      // No queryClient.invalidateQueries() — list becomes stale
    },
  });
}
```

### Non-Compliant: Search without debounce or minimum length

```typescript
// BAD: fires API call on every keystroke, no debounce
export function useSearchOrders(query: string) {
  const { data } = useQuery({
    queryKey: ['orders-search', query],  // non-stable key, unstable order
    queryFn: () => searchOrders(query),
    // No debounce, no minimum length check
  });
  return data;
}
```

## Applicability

- **Service types:** frontend
- **Domains:** all domains making API calls
- **Exceptions:** Server-side rendering scenarios may use direct `apiClient` calls outside of hooks; still MUST apply conversion rules.

## References

- STD-011: Form Patterns (mutation hooks validate form input)
- STD-012: Component Patterns (feature components own the hooks, UI tier receives data via props)
