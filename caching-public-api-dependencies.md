---
trigger: always_on
---

Caching, Public APIs & Dependency Rules

---

## 1. Cache Tag Taxonomy

All cache tags are defined in a **single source of truth**. The location is up to the project, but a consistent convention is:

```
src/shared/lib/cache/tags.ts
```

### Pattern

```ts
// src/shared/lib/cache/tags.ts
export const CACHE_TAGS = {
  // Define tags for any entity your project actually has
  product: (id: string) => `product:${id}`,
  catalog: () => 'catalog',
  user: (id: string) => `user:${id}`,
  cart: (userId: string) => `cart:user:${userId}`,
} as const;
```

Adapt the tag names to your domain. Only define tags for entities that exist in the project.

### Rules

- **Every entity that can mutate must have a cache tag.**
- **Tags are always imported from the central `tags.ts` file** — never hardcoded as inline strings.
- **The feature that mutates owns the revalidation.** Call `revalidateTag()` inside the Server Action, not from the UI.
- **Prefer `revalidateTag` over `revalidatePath`** for domain-level freshness. Use `revalidatePath` only for broad layout-level resets.

### Tag lifecycle example

```ts
// 1. Query registers the tag when fetching
export const getProducts = unstable_cache(
  async () => {
    /* ... */
  },
  ['products'],
  { tags: [CACHE_TAGS.catalog()] }
);

// 2. Server Action invalidates after mutation
export async function deleteProductAction(id: string) {
  'use server';
  // perform deletion...
  revalidateTag(CACHE_TAGS.product(id));
  revalidateTag(CACHE_TAGS.catalog());
}
```

---

## 2. Public API (index.ts) — Non-Negotiable

Every slice must expose a **stable public API** through its `index.ts`. External consumers must only import from this file.

```ts
// src/features/cart/add-to-cart/index.ts
export { AddToCartButton } from './ui/AddToCartButton';
export { addToCartAction } from './api/addToCart.action';
// Internal helpers, selectors, and private types are NOT exported
```

```ts
// src/entities/product/index.ts
export type { Product, ProductFilters } from './model/types';
export { ProductCard } from './ui/ProductCard';
export { listProducts, getProductById } from './api/product.queries';
```

### Rules

- **Import only from the slice's `index.ts`**, never from deep internal paths.
- Internal files are implementation details — they can be refactored freely without breaking consumers.

```ts
// ✅ Correct
import { AddToCartButton } from '@/features/cart/add-to-cart';

// ❌ Wrong — breaks encapsulation
import { AddToCartButton } from '@/features/cart/add-to-cart/ui/AddToCartButton';
```

---

## 3. Dependency Direction Rules

FSD enforces **unidirectional dependency flow**. Higher layers import from lower layers — never the reverse. Slices at the same layer cannot import from each other.

```
app → pages → widgets → features → entities → shared
```

| Layer      | Can import from                             | Cannot import from                          |
| ---------- | ------------------------------------------- | ------------------------------------------- |
| `pages`    | `widgets`, `features`, `entities`, `shared` | `app`                                       |
| `widgets`  | `features`, `entities`, `shared`            | `pages`, `app`                              |
| `features` | `entities`, `shared`                        | `widgets`, `pages`, `app`, other `features` |
| `entities` | `shared`                                    | everything above, other `entities`          |
| `shared`   | nothing internal                            | everything above                            |

### Cross-feature communication

Features cannot import from each other. If two features share logic, extract it to `entities/` or `shared/`.

```ts
// ❌ Wrong — feature importing from another feature
import { useAuthUser } from '@/features/auth/login';

// ✅ Correct — shared concern extracted to entities
import { getCurrentUser } from '@/entities/user';
```

---

## 4. `shared/` Purity

`shared/` must remain **business-agnostic**. It should only contain things that could belong in any Next.js project.

**Allowed in `shared/`:**

- UI kit components (Button, Input, Modal, etc.)
- Generic utilities (formatDate, cn, etc.)
- Fetcher abstraction
- Cache tag definitions
- Environment config
- TypeScript utility types

**Not allowed in `shared/`:**

- Domain types (User, Product, Appointment, etc.)
- Business rules or domain logic
- API calls specific to your application
- Components that reference your domain

---

## 5. Architecture Health Checklist

Use this on any PR that touches `src/` structure:

**Folder structure**

- [ ] New business logic is in `features/` or `entities/`, not in `shared/`
- [ ] Route files (`app/**/page.tsx`) are thin — only import and return
- [ ] New slices have an `index.ts` with explicit, intentional exports

**Dependencies**

- [ ] No feature imports from another feature
- [ ] No entity imports from a feature or widget
- [ ] `shared/` has no domain knowledge

**Data & caching**

- [ ] Fetchable data has a cache tag in `shared/lib/cache/tags.ts`
- [ ] Server Actions call `revalidateTag` after successful mutations
- [ ] New Route Handlers are for integration purposes, not internal UI calls

**Server/Client boundary**

- [ ] No `"use client"` at page or layout level without a clear reason
- [ ] Client components are leaf-like — not wrapping large subtrees
- [ ] No server-only objects passed as props to client components
