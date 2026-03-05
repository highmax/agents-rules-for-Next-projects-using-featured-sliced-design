---
trigger: model_decision
description: Apply this guide when you're adding any new service file that makes HTTP requests to the backend. (Creating a new entity that needs backend data, Adding new API endpoints to existing entities, entities/user/api/service.ts)
---

# API Architecture Style Guide - FSD (Condensed)

> Library-agnostic guide for Next.js + Feature-Sliced Design

**📖 Note**: This is a condensed version for quick reference and editor rules. For detailed explanations, additional examples, migration guides, and comprehensive documentation, see the full version: `API_STYLE_GUIDE.md`

## When to Apply

**✅ APPLY when:**

- Creating API services, data hooks, or route handlers
- Defining API types/schemas
- Setting up HTTP client infrastructure

**❌ DON'T APPLY when:**

- Static content, client-only state, build scripts
- Third-party SDKs (use as-is)

---

## Architecture Overview

```
App (app/api/)           → Thin route handlers
    ↓
Entities                 → Domain services + hooks
    ↓
Shared (shared/api/)     → HTTP client + utilities
    ↓
External API
```

**Import Rule**: Lower layers CANNOT import from higher layers.

---

## Directory Structure

```
src/
├── shared/api/
│   ├── client.ts              # HTTP client
│   ├── error-handler.ts       # Error utilities
│   └── types.ts               # Common types
│
├── entities/[entity]/
│   ├── api/
│   │   ├── service.ts         # API calls
│   │   └── endpoints.ts       # URL constants
│   ├── model/
│   │   ├── types.ts           # Domain types
│   │   ├── schemas.ts         # Validation
│   │   └── use[Entity].ts     # Data hooks
│   └── index.ts               # Public API
│
└── app/api/[...]/route.ts     # Next.js routes
```

---

## Naming Conventions

| Type      | Convention       | Example                          |
| --------- | ---------------- | -------------------------------- |
| Service   | `service.ts`     | `entities/auth/api/service.ts`   |
| Hook      | `use[Entity].ts` | `entities/auth/model/useAuth.ts` |
| Types     | `types.ts`       | `entities/auth/model/types.ts`   |
| Schemas   | `schemas.ts`     | `entities/auth/model/schemas.ts` |
| Endpoints | `endpoints.ts`   | `entities/auth/api/endpoints.ts` |
| Routes    | `route.ts`       | `app/api/auth/logout/route.ts`   |

---

## Layer Responsibilities

### Shared (`shared/api/`)

**Purpose**: Reusable infrastructure

```typescript
// shared/api/client.ts
export class ApiClient {
  async post<T>(endpoint: string, data?: unknown): Promise<Response> {
    return fetch(`${process.env.NEXT_PUBLIC_API_URL}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
  }
}

export const apiClient = new ApiClient();
```

### Entities (`entities/[entity]/`)

**Purpose**: Domain logic per entity

```typescript
// entities/appointment/api/service.ts
import { apiClient } from '@/shared/api';
import { ENDPOINTS } from './endpoints';

export const appointmentService = {
  async holdSlot(data: HoldSlotRequest): Promise<Response> {
    return apiClient.post(ENDPOINTS.HOLD, data);
  },
};
```

```typescript
// entities/appointment/model/useAppointments.ts
import useSWR from 'swr';
import { appointmentService } from '../api/service';

export function useAvailableSlots(date: string | null) {
  const { data, error, isLoading, mutate } = useSWR(date ? `/slots/${date}` : null, async () => {
    const res = await appointmentService.getSlots(date!);
    if (!res.ok) throw new Error('Failed');
    return res.json();
  });

  return { slots: data, error, isLoading, refresh: mutate };
}
```

### App (`app/api/`)

**Purpose**: Thin proxies only

```typescript
// app/api/bookings/hold/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { appointmentService } from '@/entities/appointment';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const response = await appointmentService.holdSlot(body);
    const data = await response.json();

    return NextResponse.json(data, {
      status: response.ok ? 201 : response.status,
    });
  } catch (error) {
    return NextResponse.json({ error: 'Internal error' }, { status: 500 });
  }
}
```

---

## Import Rules

### Order

```typescript
// 1. External
import { NextResponse } from 'next/server';

// 2. Shared
import { apiClient } from '@/shared/api';

// 3. Entities
import { appointmentService } from '@/entities/appointment';

// 4. Relative
import { helper } from './utils';
```

### Allowed Directions

```
✅ App → Entities → Shared
✅ Entities → Shared
❌ Shared → Entities (FORBIDDEN)
❌ Lower → Higher (FORBIDDEN)
```

### Public API Pattern

```typescript
// entities/appointment/index.ts
export { appointmentService } from './api/service';
export { useAvailableSlots } from './model/useAppointments';
export type { Appointment } from './model/types';

// Usage:
import { appointmentService, useAvailableSlots } from '@/entities/appointment';
```

---

## Code Organization

### Service File

```typescript
// 1. Imports
import { apiClient } from '@/shared/api';
import { ENDPOINTS } from './endpoints';
import type { Request, Response } from '../model/types';

// 2. Service object
export const entityService = {
  async method(data: Request): Promise<Response> {
    return apiClient.post(ENDPOINTS.PATH, data);
  },
};
```

### Hook File

```typescript
// 1. Imports
import useSWR from 'swr';
import { service } from '../api/service';

// 2. Main hook
export function useEntity(id: string | null) {
  const { data, error, isLoading } = useSWR(id ? `/entity/${id}` : null, () => fetcher(id!));

  return { data, error, isLoading };
}

// 3. Helpers
async function fetcher(id: string) {
  const res = await service.get(id);
  return res.json();
}
```

### Types File

```typescript
// 1. Request types
export interface CreateRequest { ... }

// 2. Response types
export interface CreateResponse { ... }

// 3. Domain types
export interface Entity { ... }

// 4. Utility types
export type EntityStatus = 'active' | 'inactive';
```

---

## Library-Agnostic Pattern

Same hook interface works with any library:

```typescript
// With SWR
export function useEntity(id: string | null) {
  const { data, error, isLoading, mutate } = useSWR(...);
  return { data, error, isLoading, refresh: mutate };
}

// With TanStack Query
export function useEntity(id: string | null) {
  const { data, error, isLoading, refetch } = useQuery(...);
  return { data, error, isLoading, refresh: refetch };
}
```

**Key**: Consistent return interface regardless of library.

---

## Complete Example

```typescript
// entities/appointment/api/endpoints.ts
export const ENDPOINTS = {
  HOLD: '/bookings/hold',
  SLOTS: '/slots',
};

// entities/appointment/api/service.ts
import { apiClient } from '@/shared/api';
import { ENDPOINTS } from './endpoints';

export const appointmentService = {
  async holdSlot(data: HoldSlotRequest) {
    return apiClient.post(ENDPOINTS.HOLD, data);
  },
};

// entities/appointment/model/types.ts
export interface HoldSlotRequest {
  clinic_id: string;
  date: string;
  time: string;
}

// entities/appointment/model/useAppointments.ts
import useSWR from 'swr';
import { appointmentService } from '../api/service';

export function useAvailableSlots(date: string | null) {
  const { data, error, isLoading } = useSWR(date ? `/slots/${date}` : null, async () => {
    const res = await appointmentService.getSlots(date!);
    return res.json();
  });

  return { slots: data, error, isLoading };
}

// entities/appointment/index.ts
export { appointmentService } from './api/service';
export { useAvailableSlots } from './model/useAppointments';
export type { HoldSlotRequest } from './model/types';

// Usage in component
import { useAvailableSlots } from '@/entities/appointment';

function Scheduler() {
  const { slots, error, isLoading } = useAvailableSlots('2024-03-15');
  // ...
}
```

---

## Anti-Patterns

```typescript
// ❌ Business logic in routes
export async function POST(req: NextRequest) {
  const body = await req.json();
  if (!body.email.includes('@')) { ... } // NO!
  // Use service instead
}

// ❌ Cross-layer imports
// shared/api/client.ts
import { authService } from '@/entities/auth'; // FORBIDDEN

// ❌ Bypassing public API
import { service } from '@/entities/auth/api/service'; // NO
import { service } from '@/entities/auth'; // YES
```

---

## Checklist

When creating new API integration:

- [ ] Create `entities/[entity]/api/service.ts`
- [ ] Create `entities/[entity]/api/endpoints.ts`
- [ ] Create `entities/[entity]/model/types.ts`
- [ ] Create `entities/[entity]/model/use[Entity].ts`
- [ ] Create `entities/[entity]/index.ts` (public API)
- [ ] Create `app/api/[...]/route.ts` (if needed)
- [ ] Use imports: `@/entities/[entity]`
- [ ] Ensure hooks return: `{ data, error, isLoading, refresh }`

---

**Last Updated**: January 2026 | **FSD**: 2.1 | **Next.js**: 16.x
