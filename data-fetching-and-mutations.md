---
trigger: always_on
---

Data Fetching, Server Actions & Route Handlers

## Core Principles

- **Server-side fetching is the default.** Fetch data in Server Components whenever possible.
- **Data queries belong to `entities/`.** Keep domain reads close to the domain model.
- **Mutations belong to `features/`.** The slice that changes data owns the Server Action.
- **Route Handlers are for integration boundaries only**, not for internal UI mutations.

---

## Server-Side Fetching

Data queries live inside the entity slice that owns that domain:

```
src/entities/
  product/
    api/
      product.queries.ts   ← getProductById, listProducts
  user/
    api/
      user.queries.ts      ← getCurrentUser, getUserById
```

### Pattern

```ts
// src/entities/product/api/product.queries.ts
import { unstable_cache } from 'next/cache';
import { CACHE_TAGS } from '@/shared/lib/cache/tags';

export const listProducts = unstable_cache(
  async (filters?: ProductFilters) => {
    // fetch from DB or external API
  },
  ['products-list'],
  { tags: [CACHE_TAGS.catalog()] }
);
```

---

## Server Actions (Mutations)

Server Actions live inside the **feature slice** that owns that user scenario. The action performs the mutation **and** triggers cache revalidation.

```
src/features/
  cart/
    add-to-cart/
      api/
        addToCart.action.ts   ← owns the mutation + revalidation
  auth/
    login/
      api/
        login.action.ts
```

### Pattern

```ts
// src/features/cart/add-to-cart/api/addToCart.action.ts
'use server';

import { revalidateTag } from 'next/cache';
import { CACHE_TAGS } from '@/shared/lib/cache/tags';

export async function addToCartAction(productId: string) {
  // validate input, perform mutation
  // ...

  // The feature that mutates owns the revalidation
  revalidateTag(CACHE_TAGS.cart(userId));
}
```

### Rules for Server Actions

- Always use the `"use server"` directive at the top of the file.
- Validate input before any DB or API operation.
- Always call `revalidateTag` (or `revalidatePath`) after a successful mutation.
- Handle errors explicitly — do not let unhandled exceptions reach the client.

---

## Route Handlers

Route Handlers (`app/api/**/route.ts`) are for **integration boundaries only**.

**Use Route Handlers when:**

- Receiving webhooks from external services (Stripe, etc.)
- Handling OAuth callbacks
- Exposing an HTTP endpoint for third-party consumers
- Custom request/response handling with Web APIs

**Do NOT use Route Handlers for:**

- Internal UI mutations → use Server Actions instead
- Fetching data for your own UI → fetch directly in Server Components

### Pattern

```ts
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const body = await req.text();
  const sig = req.headers.get('stripe-signature')!;
  // validate, process, respond
  return NextResponse.json({ received: true });
}
```

---

## Client-Side Fetching

Use client fetching **only when necessary**:

- Live updates independent of navigation
- User-driven polling or websockets
- Client-only auth context to call an external API

Client hooks live inside `features/` (they represent user scenarios):

```ts
// src/features/notifications/model/useNotifications.ts
'use client';

import useSWR from 'swr';

export function useNotifications() {
  return useSWR('/api/notifications', fetcher, { refreshInterval: 30000 });
}
```

---

## Decision Matrix

| Scenario                     | Solution                        |
| ---------------------------- | ------------------------------- |
| Display data on page load    | Server Component + entity query |
| Form submit or button action | Server Action in feature slice  |
| Webhook or OAuth callback    | Route Handler in `app/api/`     |
| Real-time or polling updates | Client hook in `features/`      |

---

## Streaming & Loading States

Use `loading.tsx` and `<Suspense>` intentionally:

- **Fast, cacheable data** → fetch directly in the server page, no boundary needed.
- **Slow or non-critical data** → wrap in `<Suspense>` with a skeleton fallback.

```tsx
// src/pages/home/ui/HomePage.tsx
import { Suspense } from 'react';
import { ProductGrid, ProductGridSkeleton } from '@/widgets/product-grid';

export async function HomePage() {
  return (
    <main>
      <Suspense fallback={<ProductGridSkeleton />}>
        <ProductGrid />
      </Suspense>
    </main>
  );
}
```
