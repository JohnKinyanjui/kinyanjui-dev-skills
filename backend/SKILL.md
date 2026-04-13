---
name: backend-golang-style
description: Implement and review Go backend changes using layered handler-service-data architecture, sqlc query workflows, and maintainable file organization. Use when adding API endpoints, refactoring backend domains, writing SQL query files, generating sqlc code, and enforcing consistent conventions across cmd, internal, services, pkg, infrastructure, and orchestrator folders.
---

# Backend Golang Style

## Overview

Use this skill for general Go backend work with a consistent architecture:
- handlers are thin
- services hold business logic
- SQL lives in query files
- sqlc generated code is treated as read-only output

## Canonical Backend Layout

Use this structure as default:

```text
cmd/                              # app entrypoints (api, migrator, workers)
internal/
  api/
    handler/
      internal/                   # authenticated/private handlers
      public/                     # public handlers
      router.go                   # central registration
    helpers/                      # response and request helpers
  db/
    schema/                       # base schema migrations
    migrations/                   # timestamped migrations
    query/                        # sqlc source queries (private/internal)
    pb_query/                     # sqlc source queries (public)
    generated/                    # sqlc generated output (read-only)
    seeds/                        # seed sql data
    migrator/                     # migration runner code
services/                         # domain business logic (can be nested)
pkg/                              # shared packages (helpers, middlewares, logger, etc.)
infrastructure/                   # infra integration logic
orchestrator/                     # orchestration/runtime deployment helpers
```

## Execution Workflow

1. Classify the change before coding.
- API route change: update handler files and router wiring.
- Business logic change: update services and supporting utils.
- Data/query change: update `query` or `pb_query` then run sqlc.

2. Implement from top to bottom with strict layering.
- Handler layer: parse request/context and call service.
- Service layer: validate input, enforce business rules, call db/query functions.
- Data layer: write SQL in query files; never in service strings.

3. Validate incrementally.
- Run targeted package tests first.
- Run broader `go test ./...` when practical.
- Ensure compile passes after sqlc generation.

## Non-Negotiable Rules

- Never edit files in `internal/db/generated` manually.
- Always write query changes in `internal/db/query` or `internal/db/pb_query`, then run `sqlc generate`.
- Keep SQL statements lowercase.
- Keep sqlc query annotation names in Upper Camel Case.
- Keep handlers thin; move business logic into services.
- Keep functions short; split long logic into focused helpers.
- Put shared service-local logic in `utils.go`.
- Keep every function in `utils.go` private (unexported, lowercase function names).

## SQLC Conventions

### Query annotation naming

Use verb-first Upper Camel Case names:
- `GetUser`
- `ListOrders`
- `CreateProduct`
- `UpdateStoreConfig`
- `DeleteCoupon`
- `ListAdminContentPosts`

Example:

```sql
-- name: ListAdminContentPosts :many
select id, title, created_at
from content_posts
where admin_id = @admin_id
order by created_at desc;
```

### SQL style

- Keep SQL keywords and statements lowercase.
- Keep query logic explicit and readable.
- Prefer stable ordering (`order by`) for list endpoints.
- Keep store/tenant filters explicit in where clauses for multi-tenant backends.

## Code Style Expectations

- Prefer small functions with one clear responsibility.
- Keep handlers mostly orchestration and response shaping.
- Parse and validate IDs early; return clear error messages.
- Keep exported API minimal; default to private helpers.
- Keep shared cross-domain helpers in `pkg`, not copied across services.

## Required File Arrangement

### Service feature folder pattern

```text
services/<domain>/<feature>/
  data.go
  get.go
  post.go
  put.go
  delete.go
  utils.go
```

### Handler feature folder pattern

```text
internal/api/handler/internal/<domain>/<feature>/ (or public/)
  router.go
  data.go
  get.go
  post.go
  put.go
  delete.go
  utils.go
```

When operations are partial, include only needed files, but keep canonical file names.

### Nested service folders for large domains

This is valid and recommended when a domain is large:

```text
services/accounts/auth/
services/accounts/teams/
services/accounts/users/
```

The same `data/get/post/put/delete/utils` file pattern applies inside each nested folder.

## Handler-Service Pattern Skeleton

### Handler example

```go
func (rt *Router) getThing(ctx echo.Context) error {
    userID := ctx.Get("user_id").(string)
    thingID := ctx.Param("id")

    result, err := thing_service.GetThing(userID, thingID)
    if err != nil {
        return respondError(ctx, err)
    }

    return respondSuccess(ctx, result)
}
```

### Service example

```go
func GetThing(userID, id string) (any, error) {
    parsedID, err := uuid.Parse(id)
    if err != nil {
        return nil, fmt.Errorf("invalid id: %w", err)
    }

    result, err := db.Query.GetThing(context.Background(), query.GetThingParams{
        ID: parsedID,
    })
    if err != nil {
        return nil, fmt.Errorf("failed to get thing: %w", err)
    }

    return result, nil
}
```

### Private utils example

```go
// utils.go
func normalizeStatus(raw string) string {
    return strings.TrimSpace(strings.ToLower(raw))
}
```

## Feature Delivery Checklist

- Define or update API contracts in `data.go`.
- Implement service operations in canonical files.
- Implement handler endpoints and register routes in `router.go`.
- Write/update SQLC query files.
- Ensure query names are Upper Camel Case.
- Ensure SQL statements are lowercase.
- Run `sqlc generate`.
- Verify generated APIs compile.
- Run tests.

## Fast Review Rules

- Reject direct edits in `internal/db/generated`.
- Reject lowercase or snake_case sqlc query names.
- Reject uppercase SQL keywords if project standard is lowercase SQL.
- Reject exported helper functions in `utils.go`.
- Reject large service files that should be split into nested feature folders.
