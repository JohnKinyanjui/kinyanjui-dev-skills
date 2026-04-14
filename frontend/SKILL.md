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
- Use server functions for service operations and auth-sensitive requests.
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
- Every operation file (`get.ts`, `post.ts`, `put.ts`, `delete.ts`) must include `"use server"` at the top.
- For authenticated server calls, always resolve token with `const token = await getAccessToken();` and send it in the request headers.
- Handle cookie read/write in server functions using Next.js cookies APIs; keep cookie persistence/refresh logic in shared auth/http utilities, not in UI components.
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
import { getAccessToken } from "@/lib/shared/http/auth";
import { Product } from "./data";

export async function getProduct(id: string): Promise<Product> {
  const token = await getAccessToken();
  const res = await clientV1.get<Product>(`products/${id}`, {
    headers: token ? { Authorization: `Bearer ${token}` } : undefined,
  });
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

## Next.js Backend with Prisma

### Backend Architecture

When Next.js is used as the backend (full-stack monorepo), follow this structure:

```text
lib/server/
  db/                          # Prisma client and database utilities
    prisma.ts                  # Prisma client singleton (CRITICAL)
    seed.ts                    # Database seeding scripts
    migrations/                # Migration utilities (if needed)
  core/                        # Core business logic and utilities
    errors.ts                  # Custom error classes and error handling
    validation.ts              # Zod schemas and validation
    pagination.ts              # Pagination helpers
  auth/                        # Authentication utilities
    session.ts                 # Session management
    permissions.ts             # Role/permission checks
    guards.ts                  # Route guards and middleware helpers
  http/                        # HTTP client and request utilities
    client.ts                  # HTTP client configuration
    auth.ts                    # Auth token handling for requests
    helpers.ts                 # HTTP utility functions
  domain/                      # Domain-driven service modules (organize by business domain)
    accounts/
    analytics/
    billing/
    catalog/
      # Sub-folders when feature grows:
      variants/
        create.ts              # Nested feature: product variants
        list.ts
        utils.ts
        index.ts
    inventory/
    orders/
    # Each domain contains service files: create.ts, list.ts, update.ts, delete.ts, utils.ts, index.ts
  integrations/                # Third-party service integrations
    payment/
      stripe.ts                # Payment provider integration
    storage/
      s3.ts                    # File storage integration
    email/
      sendgrid.ts              # Email service integration
  services/                    # Deprecated - migrate to domain/ (legacy business logic)
    products/
      create.ts                # Business logic for creating products
      update.ts                # Business logic for updating products
      delete.ts                # Business logic with pre-delete checks
      list.ts                  # Listing with filters, sorting, pagination
      utils.ts                 # Private helpers for products domain only
      index.ts                 # Service exports
      # Sub-folders when feature grows:
      variants/
        create.ts              # Nested feature: product variants
        list.ts
        utils.ts
        index.ts
  types/                       # Server-side type definitions
    api.ts                     # API response types
    errors.ts                  # Error type definitions

app/api/                       # API route handlers (thin layer)
  [domain]/
    route.ts                   # Domain-level routes (GET, POST)
    [id]/
      route.ts                 # Individual resource routes (GET, PUT, DELETE)

prisma/
  schema.prisma                # Database schema
  migrations/                  # Auto-generated migrations
```

**Key Principle**: Keep business logic in `lib/server/services/`, not in API route handlers. API routes should only handle HTTP concerns (request/response parsing, status codes, headers).

### Prisma Rules

- **NEVER use raw SQL queries** - Always use Prisma Client for database operations
- All SQL-like queries must be written in lowercase (user rule)
- For CREATE and UPDATE queries, use `ON CONFLICT DO UPDATE SET` with the `id` field
- For DELETE queries, use `ON CONFLICT DO NOTHING`
- Always use `@` parameter syntax instead of `$1` positional parameters (when applicable)
- Define all database models in `prisma/schema.prisma`
- Run migrations using `npx prisma migrate dev` for development
- Use `npx prisma generate` after schema changes

### API Route Handlers with Prisma

```typescript
// app/api/products/route.ts
import { prisma } from "@/lib/server/db/prisma";
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  try {
    const products = await prisma.product.findMany({
      where: { active: true },
      orderBy: { createdAt: "desc" },
    });
    return NextResponse.json(products);
  } catch (error) {
    return NextResponse.json(
      { error: "Failed to fetch products" },
      { status: 500 },
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const product = await prisma.product.create({
      data: body,
    });
    return NextResponse.json(product, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: "Failed to create product" },
      { status: 500 },
    );
  }
}
```

### Error Prevention Patterns

**1. Always Use Service Layer**

```typescript
// BAD: Business logic in API route
export async function POST(request: NextRequest) {
  const body = await request.json();
  const product = await prisma.product.create({ data: body }); // Direct DB call
  return NextResponse.json(product);
}

// GOOD: Business logic in service layer
// lib/server/services/products/create.ts
export async function createProduct(data: CreateProductInput) {
  const validated = productSchema.parse(data);
  const slug = generateSlug(validated.name);
  return prisma.product.create({
    data: { ...validated, slug },
  });
}

// app/api/products/route.ts
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const product = await createProduct(body);
    return NextResponse.json(product, { status: 201 });
  } catch (error) {
    return handleApiError(error);
  }
}
```

**2. Centralized Error Handling**

```typescript
// lib/server/core/errors.ts
export class ApiError extends Error {
  constructor(
    public statusCode: number,
    message: string,
    public code?: string,
  ) {
    super(message);
  }
}

export function handleApiError(error: unknown): NextResponse {
  if (error instanceof ApiError) {
    return NextResponse.json(
      { error: error.message, code: error.code },
      { status: error.statusCode },
    );
  }
  if (error instanceof z.ZodError) {
    return NextResponse.json(
      { error: "Validation failed", details: error.errors },
      { status: 400 },
    );
  }
  console.error(error);
  return NextResponse.json({ error: "Internal server error" }, { status: 500 });
}
```

**3. Input Validation with Zod**

```typescript
// lib/server/core/validation.ts
import { z } from "zod";

export const productSchema = z.object({
  name: z.string().min(1).max(255),
  price: z.number().positive(),
  categoryId: z.string().uuid(),
  description: z.string().optional(),
});

// Usage in service
export async function createProduct(data: unknown) {
  const validated = productSchema.parse(data);
  // validated is now typed and safe
}
```

**4. Pagination Helpers**

```typescript
// lib/server/core/pagination.ts
export interface PaginationParams {
  page?: number;
  limit?: number;
  sortBy?: string;
  sortOrder?: "asc" | "desc";
}

export function getPagination({ page = 1, limit = 20 }: PaginationParams) {
  const skip = (page - 1) * limit;
  const take = Math.min(limit, 100); // Max 100 per page
  return { skip, take };
}

export function createPaginatedResponse<T>(
  data: T[],
  total: number,
  page: number,
  limit: number,
) {
  return {
    data,
    meta: {
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit),
    },
  };
}
```

**5. Type-Safe API Responses**

```typescript
// lib/server/types/api.ts
export interface ApiSuccessResponse<T> {
  data: T;
  meta?: {
    total?: number;
    page?: number;
    limit?: number;
  };
}

export interface ApiErrorResponse {
  error: string;
  code?: string;
  details?: unknown;
}

// Usage in service return types
export async function listProducts(
  params: PaginationParams,
): Promise<ApiSuccessResponse<Product[]>> {
  // Implementation
}
```

### Issue Fixes

**Common Issue: Prisma Client Not Singleton**

- Problem: Creating multiple Prisma Client instances causes connection pool exhaustion
- Fix: Export a singleton Prisma Client instance from `lib/server/db/prisma.ts`

```typescript
// lib/server/db/prisma.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

**Common Issue: Type Safety Loss**

- Problem: Using `any` or loose types in API responses
- Fix: Use Prisma-generated types and Zod validation

```typescript
import { productSchema } from "@/lib/server/core/validation";
const validatedData = productSchema.parse(body);
```

**Common Issue: Transaction Safety**

- Problem: Multiple related operations without atomicity
- Fix: Use Prisma transactions for related operations

```typescript
await prisma.$transaction([
  prisma.order.create({ data: orderData }),
  prisma.inventory.update({
    where: { id },
    data: { quantity: { decrement: 1 } },
  }),
]);
```

### Backend Service Integration

When combining Next.js backend with the service layer:

```typescript
// lib/services/products/get.ts (server-side)
"use server";

import { prisma } from "@/lib/server/db/prisma";
import { Product } from "./data";

export async function getProducts(): Promise<Product[]> {
  return prisma.product.findMany({
    where: { active: true },
    include: { category: true },
  });
}
```

## Review Checklist

### Frontend

- API calls are inside `lib/services`, not scattered in UI files.
- Service file naming follows `data/get/post/put/delete/hook/index/utils`.
- Components are placed in the right folder (`ui`, `shared`, `features`, `providers`).
- Dashboard routes follow `app/dashboard/<domain>/<feature>/page.tsx`.
- Hooks are used for server state and pages handle loading/error states cleanly.

### Backend

- **Prisma Client is used as a singleton** - check `lib/server/db/prisma.ts` exists with global pattern.
- **Never use raw SQL queries** - all DB operations use Prisma Client methods.
- **Business logic lives in `lib/server/services/`** - not in API route handlers.
- **All inputs validated with Zod** - no `any` types, use `schema.parse()`.
- **Centralized error handling** - use `handleApiError()` in all API routes.
- **Type-safe API responses** - use `ApiSuccessResponse<T>` and `ApiErrorResponse`.
- **Pagination implemented consistently** - use `getPagination()` helper.
- **Proper error logging** - errors logged on server, sanitized before sending to client.
- **No direct Prisma calls in API routes** - routes call services, services use Prisma.

### SQL Rules

- CREATE/UPDATE queries use `ON CONFLICT DO UPDATE SET` with `id` field.
- DELETE queries use `ON CONFLICT DO NOTHING`.
- All SQL-like queries in lowercase.
- Parameter syntax uses `@` instead of `$1` positional parameters.
