---
trigger: always_on
---

Folder Structure & FSD Layers

## Overview

This project follows **Feature-Sliced Design (FSD)** combined with Next.js App Router. The `app/` directory is used **only for routing**. All business logic, UI, and domain code lives under `src/` organized by FSD layers.

The folder structure shown here is a **reference example**, not a template to copy. Adapt it to your project's actual needs — not every project needs all layers or all folders shown.

---

## Layer Responsibilities

| Layer            | Responsabilidad                                                       |
| ---------------- | --------------------------------------------------------------------- |
| `app/` (Next.js) | Solo routing: layouts, páginas, error/loading boundaries              |
| `src/app/`       | Providers globales, shell de la aplicación (solo si aplica)           |
| `src/pages/`     | Composición de la página completa                                     |
| `src/widgets/`   | Bloques UI grandes compuestos de features y entities (solo si aplica) |
| `src/features/`  | Escenarios de interacción del usuario (acciones de negocio)           |
| `src/entities/`  | Modelos de dominio, queries y UI primitiva del dominio                |
| `src/shared/`    | Código reutilizable sin lógica de negocio (UI kit, utils, config)     |

> `widgets/` is optional. In smaller projects, `pages/` can compose directly from `features/` and `entities/`. Only add layers the project actually needs.

---

## Core Principles

- **`app/**/page.tsx` must be thin.\*\* Import a page component, pass params, return UI. No business logic.
- **Never import from a slice's internals.** Only import from its `index.ts` public API.
- **`shared/` must not contain domain logic.** If it knows about your domain, it belongs in `entities/`.
- **Every slice must have an `index.ts`** that exports only what external consumers need.
- **Routes are composition nodes**, not implementations. They should read like orchestration.
- **Only create folders the project actually needs.** Do not generate speculative structure.

---

## Reference Directory Structure

This is an illustrative reference for a mid-sized project. Your project may only need a subset of this.

```
app/
  (public)/
    layout.tsx
    page.tsx
  (auth)/
    login/
      page.tsx
  layout.tsx
  error.tsx
  loading.tsx

src/
  pages/
    home/
      ui/
        HomePage.tsx
      index.ts

  features/
    auth/
      login/
        ui/
          LoginForm.tsx
        model/
          login.schema.ts
          useLogin.ts
        api/
          login.action.ts
        index.ts

  entities/
    user/
      model/
        types.ts
      ui/
        UserAvatar.tsx
      index.ts

  shared/
    ui/
      button/
        Button.tsx
    lib/
      cache/
        tags.ts
    config/
      env.ts
```

The minimum viable structure is just:

```
src/
  pages/
  features/
  entities/
  shared/
```

---

## Example: Thin Route File

```tsx
// app/(public)/page.tsx
import { HomePage } from '@/pages/home';

export default function Page() {
  return <HomePage />;
}
```

```tsx
// app/products/[id]/page.tsx
import { ProductDetailsPage } from '@/pages/product-details';

export default function Page({ params }: { params: { id: string } }) {
  return <ProductDetailsPage productId={params.id} />;
}
```
