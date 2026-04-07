---
name: swagger-endpoint-documentation
description: Use when adding or modifying REST endpoints in the Spring Boot project and OpenAPI/Swagger annotations need to be added or updated — covers controller annotations, DTO schemas, error responses, headers, security, and enum handling with springdoc-openapi
---

# Swagger Endpoint Documentation

## Overview

Every endpoint and DTO must be fully annotated with `io.swagger.v3.oas.annotations` so the generated OpenAPI spec is complete and accurate. Annotations only — never modify method bodies or business logic.
Follow the best practices and patterns in this guide to ensure consistency across the codebase. Always refer to existing annotated endpoints as examples.

## The Iron Rule

**Add annotations only. Do NOT modify method signatures, return types, or method bodies.** If the existing code returns `Mono<Void>`, annotate that — don't change it.

## Controller Annotations

### Class Level

```java
@RestController
@RequestMapping("/api/v1/polls")
@Tag(name = "Polls", description = "Community verification polls for profiles")
public class PollController {
```

### Endpoint Level — Full Example

```java
@Operation(
    summary = "Cast a vote on a poll",                    // short, imperative
    description = "Voter's city must match deceased's city. Each user votes once per poll.",
    security = @SecurityRequirement(name = "bearerAuth")  // only on authenticated endpoints
)
@ApiResponses({
    @ApiResponse(responseCode = "200", description = "Vote cast successfully"),
    @ApiResponse(responseCode = "401", description = "Missing or invalid JWT (UNAUTHORIZED)",
        content = @Content(schema = @Schema(implementation = ErrorResponse.class))),
    @ApiResponse(responseCode = "403", description = "City mismatch (POLL_NOT_ELIGIBLE) or banned (USER_BANNED)",
        content = @Content(schema = @Schema(implementation = ErrorResponse.class))),
    @ApiResponse(responseCode = "409", description = "Already voted (POLL_ALREADY_VOTED)",
        content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
})
@PostMapping("/{pollId}/vote")
public Mono<Void> vote(
    @Parameter(description = "UUID of the poll", required = true,
        example = "3fa85f64-5717-4562-b3fc-2c963f66afa6")
    @PathVariable UUID pollId,
    @Valid @RequestBody CastVoteRequest req,
    @Parameter(hidden = true) @AuthenticationPrincipal Mono<UserClaims> claims
) {
```

## Quick Reference — What to Annotate

| Element | Annotation | Key Attributes |
|---------|-----------|---------------|
| Controller class | `@Tag(name, description)` | One tag per controller |
| Each endpoint | `@Operation(summary, description)` | Add `security` for auth endpoints |
| Path/query params | `@Parameter(description, required, example)` | Use realistic UUID examples |
| `@AuthenticationPrincipal` | `@Parameter(hidden = true)` | Always hide from Swagger UI |
| Success response | `@ApiResponse(responseCode, description)` | Include `@Content` + `@Schema` if body exists |
| Void response | `@ApiResponse(responseCode = "200/204", description = "...")` | No `@Content` needed |
| Error response | `@ApiResponse` with `@Content(schema = @Schema(implementation = ErrorResponse.class))` | Include ErrorCode in description |
| Request headers | `@Parameter(in = ParameterIn.HEADER, name = "X-Request-ID")` | On controller class or method |
| DTO class/record | `@Schema(description)` | On the type itself |
| DTO fields | `@Schema(description, example)` | Every field |
| Enum fields | `@Schema(implementation = VoteChoice.class)` | Reference the enum class, don't use `allowableValues` strings |

## Error Response Pattern

All errors use `ErrorResponse`. Include the `ErrorCode` name in the `@ApiResponse` description so consumers can match on it:

```java
@ApiResponse(responseCode = "403", description = "City mismatch (POLL_NOT_ELIGIBLE)",
    content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
```

**Look up actual error codes** from `ErrorCode` enum before annotating. Never invent error codes.

| ErrorCode | HTTP Status |
|-----------|------------|
| `VALIDATION_ERROR` | 400 |
| `UNAUTHORIZED` | 401 |
| `FORBIDDEN` | 403 |
| `POLL_NOT_ELIGIBLE` | 403 |
| `USER_BANNED` | 403 |
| `PROFILE_NOT_FOUND` | 404 |
| `MEDIA_NOT_FOUND` | 404 |
| `THREAD_NOT_FOUND` | 404 |
| `POLL_ALREADY_VOTED` | 409 |
| `POLL_COOLDOWN_ACTIVE` | 409 |
| `MANAGER_ALREADY_CLAIMED` | 409 |

**This table may be outdated.** Always read `ErrorCode.java` before annotating to get the current list.

## DTO Schema Annotations

```java
@Schema(description = "Current state of a community verification poll")
public record PollResponse(
    @Schema(description = "Poll unique identifier", example = "3fa85f64-5717-4562-b3fc-2c963f66afa6")
    UUID id,

    @Schema(description = "Current poll status", implementation = PollStatus.class)
    String status,

    @Schema(description = "YES vote count", example = "42", minimum = "0")
    int yesVotes
) {}
```

**Rules:**
- Every field gets `@Schema(description, example)`
- Enum fields: use `implementation = EnumClass.class` instead of `allowableValues`
- Numeric fields: add `minimum`/`maximum` where applicable
- Date fields: add realistic `example` values

## Request/Response Headers

The project uses `X-Request-ID` (echoed in responses) and `X-Client-Platform`. These are added globally via `OperationCustomizer` in `OpenApiConfig` — **do not add them per-controller**.

**Important:** `@Parameters` is method-level only, NOT class-level. For global headers, use `OperationCustomizer`:

```java
@Bean
public OperationCustomizer globalHeaders() {
    return (operation, handlerMethod) -> {
        operation.addParametersItem(
            new HeaderParameter()
                .name("X-Request-ID")
                .description("Correlation ID echoed in response, auto-generated if absent")
                .required(false)
                .schema(new StringSchema().format("uuid")));
        return operation;
    };
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Modifying method body while adding annotations | Annotations only. Never change logic. |
| Inventing error codes not in `ErrorCode` enum | Read `ErrorCode.java` first |
| Using `allowableValues = {"YES", "NO"}` for enums | Use `implementation = VoteChoice.class` |
| Missing `@Parameter(hidden = true)` on `@AuthenticationPrincipal` | Always hide injected security principal |
| Forgetting `@Content` + `@Schema` on error responses | Every error response needs `ErrorResponse` schema |
| Adding `@Content` on void/empty responses | No content annotation needed for 200/204 with no body |
| Not including `security = @SecurityRequirement(name = "bearerAuth")` on auth endpoints | Check `SecurityConfig` for which endpoints need auth |

## Checklist

Before marking endpoint documentation complete:

- [ ] `@Tag` on controller class
- [ ] `@Operation(summary, description)` on every method
- [ ] `@SecurityRequirement` on authenticated endpoints
- [ ] `@Parameter` on all path/query params with examples
- [ ] `@Parameter(hidden = true)` on `@AuthenticationPrincipal`
- [ ] `@ApiResponse` for success AND all possible errors (check service layer for thrown `AppException`s)
- [ ] `@Schema` on every DTO class and field
- [ ] Enum fields use `implementation` not `allowableValues`
- [ ] Headers documented (`X-Request-ID`, `X-Client-Platform`)
- [ ] No method body modifications
