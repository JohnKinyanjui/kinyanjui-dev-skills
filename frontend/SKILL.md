---
name: kinyanjui-nextjs-skill
description: Implement and review Next.js dashboard frontend changes with clear folder organization for app routes, API service modules, hooks, and components. Use when adding pages, wiring server-side API calls, creating service files, structuring components, and enforcing consistent frontend architecture for admin/dashboard systems.
---

# Frontend Nextjs Dashboard Style

## Overview

Use this skill for Next.js App Router dashboards where frontend code is organized around:
- route modules in `app/`
- API service modules in `lib/services/`
- reusable UI and feature components in `components/`
- client state and server state separation via hooks

## Canonical Frontend Layout

```text
app/
  api/                          # route handlers when needed
  dashboard/                    # authenticated dashboard area
    <domain>/<feature>/page.tsx
  pos/
  docs/
  sign-up/
  waitlist/
  layout.tsx
  page.tsx

components/
  ui/                           # reusable primitive UI components
  shared/                       # shared cross-feature components
  features/                     # feature-level components
  providers/                    # context/query/theme providers

hooks/                          # reusable hooks

lib/
  services/                     # server/API service layer
    <domain>/<feature>/
      data.ts
      get.ts
      post.ts
      put.ts
      delete.ts
      hook.ts
      index.ts
      utils.ts
  shared/http/                  # http client, auth/cookie utils
  utils.ts                      # shared frontend utilities (e.g. class merging)

public/
types/
```

## Reference Domain Map (Topduka-Style)

Use this domain split as default for dashboard-heavy apps.

App route groups:
- `app/dashboard/chats`
- `app/dashboard/customers`
- `app/dashboard/employees`
- `app/dashboard/inventory`
- `app/dashboard/listing`
- `app/dashboard/marketing`
- `app/dashboard/orders`
- `app/dashboard/pos`
- `app/dashboard/reports`
- `app/dashboard/reviews`
- `app/dashboard/settings`
- `app/dashboard/templates`

Service domains:
- `lib/services/accounts`
- `lib/services/analytics`
- `lib/services/chats`
- `lib/services/checkout`
- `lib/services/developer`
- `lib/services/listing`
- `lib/services/marketing`
- `lib/services/pos`
- `lib/services/settings`
- `lib/services/storage`
- `lib/services/subscriptions`

Component groups:
- `components/ui` for primitives
- `components/shared` for cross-domain blocks
- `components/features` for feature/domain composition
- `components/providers` for app-level providers

## Service Layer Rules (Server-Side API Focus)

- Keep server/API communication in `lib/services`, not directly in pages/components.
- Use feature folders under domain paths, for example:
  - `lib/services/listing/products/...`
  - `lib/services/accounts/customers/...`
  - `lib/services/marketing/banners/...`
- Use `data.ts` for feature types and payload contracts.
- Use operation files by intent:
  - `get.ts` read/list
  - `post.ts` create
  - `put.ts` update
  - `delete.ts` delete
- Use `hook.ts` for feature-specific React Query hooks when needed.
- Use `index.ts` as feature-level export surface.
- Use `utils.ts` for service-local helpers and keep them private unless cross-feature reuse is required.

When a feature is partial, include only needed files but keep canonical names.

## Page and Component Arrangement Rules

- Keep route pages under `app/dashboard/<domain>/<feature>/page.tsx`.
- Keep route-local composition in feature components, not in giant page files.
- Keep primitive reusable controls in `components/ui`.
- Keep business feature blocks in `components/features/<domain-or-feature>`.
- Keep cross-feature shared blocks in `components/shared`.
- Keep providers in `components/providers`.

## State and Data Flow Rules

- Keep API fetching/mutations in service functions and feature hooks.
- Keep server state in React Query hooks (or equivalent project standard).
- Keep UI state local to components/pages.
- Handle loading, empty, and error states explicitly.
- Do not scatter duplicated request logic across pages.

## Implementation Workflow

1. Define contracts first.
- Add/update `data.ts` for response models and request payloads.

2. Implement API operations.
- Add required `get/post/put/delete` files under service feature folder.

3. Add hooks for server state.
- Add or update `hook.ts` for query/mutation orchestration.

4. Build feature UI.
- Create/update components under `components/features/...`.

5. Wire route page.
- Compose feature components in `app/dashboard/.../page.tsx`.

6. Export cleanly.
- Update `index.ts` exports in service and component modules when needed.

## Code Quality Rules

- Keep components and hooks focused and short.
- Split large components into smaller feature parts.
- Use clear prop types and avoid ambiguous `any`.
- Keep utility logic out of JSX-heavy files.
- Reuse shared UI primitives from `components/ui` before adding new variants.

## Example Service Skeleton

```typescript
// lib/services/listing/products/get.ts
"use server";

import { clientV1 } from "@/lib/shared/http/client";
import { Product } from "./data";

export async function getProduct(id: string): Promise<Product> {
  const res = await clientV1.get<Product>(`products/${id}`);
  return res.data;
}
```

## Example Hook Skeleton

```typescript
// lib/services/listing/products/hook.ts
import { useQuery } from "@tanstack/react-query";
import { getProduct } from "./get";

export function useProduct(id?: string) {
  return useQuery({
    queryKey: ["product", id],
    queryFn: () => getProduct(id!),
    enabled: !!id,
  });
}
```

## Example Route Skeleton

```typescript
// app/dashboard/listing/products/page.tsx
"use client";

import { useProduct } from "@/lib/services/listing/products/hook";

export default function ProductsPage() {
  const { data, isLoading, error } = useProduct("some-id");

  if (isLoading) return <div>loading...</div>;
  if (error) return <div>failed to load</div>;

  return <div>{data?.name}</div>;
}
```

## Review Checklist

- API calls are inside `lib/services`, not scattered in UI files.
- Service file naming follows `data/get/post/put/delete/hook/index/utils`.
- Components are placed in the right folder (`ui`, `shared`, `features`, `providers`).
- Dashboard routes follow `app/dashboard/<domain>/<feature>/page.tsx`.
- Hooks are used for server state and pages handle loading/error states cleanly.
