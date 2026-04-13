---
name: backend-golang-style
description: Implement and review Go backend changes using layered handler-service-data architecture, sqlc query workflows, and maintainable file organization. Use when adding endpoints, refactoring backend domains, writing query files, generating sqlc code, or enforcing consistent Go backend conventions across services and handlers.
---

# Backend Golang Style

## Overview

Use this skill for general Go backend work with a consistent architecture: handlers are thin, services hold business logic, and SQL is managed through query files and sqlc generation.

## Execution Workflow

1. Classify the change.
- API surface change: update handlers and router wiring.
- Business logic change: update services first, then handlers.
- Data change: update SQL query files and regenerate sqlc outputs.

2. Apply layering in this order.
- Handler layer: parse input, read auth/context values, call service, format response.
- Service layer: validate input, enforce business rules, call repository/sqlc methods.
- Data layer: keep SQL in query files; do not place SQL strings in services.

3. Follow sqlc workflow strictly.
- Write or update SQL query files first.
- Run `sqlc generate`.
- Use generated types/functions in services.

4. Verify quickly and incrementally.
- Run focused tests for changed packages first.
- Run broader `go test ./...` before finalizing when practical.

## Non-Negotiable Rules

- Never edit sqlc generated files directly.
- Always write queries in source query files, then run `sqlc generate`.
- Keep all SQL statements in lowercase.
- Keep handlers thin; business logic belongs in services.
- Keep code short and readable; split long logic into small helpers.
- Put shared service-only logic in `utils.go`.
- Keep all functions in `utils.go` private (unexported, lowercase function names).

## Required File Layout

Use these filenames consistently for both services and handlers.

Service package default:
- `data.go`
- `get.go`
- `post.go`
- `put.go`
- `delete.go`
- `utils.go`

Handler package default:
- `router.go`
- `data.go`
- `get.go`
- `post.go`
- `put.go`
- `delete.go`
- `utils.go`

For large domains, allow nested service folders to reduce file size and complexity.

Examples:
- `services/accounts/auth/...`
- `services/accounts/teams/...`
- `services/accounts/profile/...`

The same naming convention still applies inside each nested feature folder.

## Code Style Expectations

- Prefer small functions (target concise functions; split when complexity grows).
- Prefer explicit validation at boundaries (`data.go` params + validation methods).
- Parse/validate IDs early and fail fast with clear messages.
- Keep exported API surface minimal; default to private helpers unless reuse requires export.

## SQL Style Expectations

Use lowercase SQL consistently.

```sql
-- name: get_user :one
select id, email
from users
where id = @id;
```

```sql
-- name: list_users :many
select id, email
from users
order by created_at desc
offset @skip
limit @limit;
```

If your project requires specific sqlc query-name casing, keep that project convention, but keep SQL statements themselves lowercase.

## Implementation Skeletons

### Handler Pattern

```go
func (rt *Router) getThing(ctx echo.Context) error {
    userID := ctx.Get("user_id").(string)
    id := ctx.Param("id")

    result, err := thing_service.GetThing(userID, id)
    if err != nil {
        return respondError(ctx, err)
    }

    return respondSuccess(ctx, result)
}
```

### Router Pattern

```go
type Router struct {
    private *echo.Group
}

func Handler(private *echo.Group) *Router {
    return &Router{private: private}
}

func (rt *Router) Routes() {
    rt.private.GET("/things", rt.getThings)
    rt.private.GET("/things/:id", rt.getThing)
    rt.private.POST("/things", rt.createThing)
    rt.private.PUT("/things/:id", rt.updateThing)
    rt.private.DELETE("/things/:id", rt.deleteThing)
}
```

### Service Pattern

```go
func GetThing(userID, id string) (any, error) {
    thingID, err := uuid.Parse(id)
    if err != nil {
        return nil, fmt.Errorf("invalid thing id: %w", err)
    }

    result, err := store.Query.GetThing(context.Background(), query.GetThingParams{
        ID: thingID,
    })
    if err != nil {
        return nil, fmt.Errorf("failed to get thing: %w", err)
    }

    return result, nil
}
```

### Private Utils Pattern

```go
// utils.go
func normalizeStatus(input string) string {
    return strings.TrimSpace(strings.ToLower(input))
}

func parseOptionalUUID(raw string) (uuid.UUID, bool, error) {
    if strings.TrimSpace(raw) == "" {
        return uuid.Nil, false, nil
    }
    id, err := uuid.Parse(raw)
    if err != nil {
        return uuid.Nil, false, err
    }
    return id, true, nil
}
```

## Feature Delivery Checklist

- Define or update data contracts in `data.go`.
- Implement read operations in `get.go`.
- Implement create operations in `post.go`.
- Implement update operations in `put.go`.
- Implement delete operations in `delete.go`.
- Move repeated service logic into private helpers in `utils.go`.
- Update SQL query files.
- Run `sqlc generate`.
- Wire routes in `router.go`.
- Run tests.

## Try Prompts

### Prompt 1

Add a new feature module for team invitations in a Go backend using the file layout `data.go`, `get.go`, `post.go`, `put.go`, `delete.go`, `utils.go`, with private helpers in `utils.go` and sqlc-based data access.

### Prompt 2

Refactor a large `accounts` service into nested folders (`accounts/auth`, `accounts/teams`) while preserving handler routes and using the same canonical file naming convention.

### Prompt 3

Review a backend PR and enforce these rules: no direct edits to sqlc generated code, SQL in lowercase, short functions, and shared service logic moved into private `utils.go` helpers.
