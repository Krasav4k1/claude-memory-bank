---
name: http-test-files-with-endpoints
description: Use when adding or modifying REST endpoints in a Spring Boot project - requires creating or updating .http test files alongside implementation code
---

# HTTP Test Files With Endpoints

## Overview

Every new endpoint ships with a runnable `.http` request. No endpoint is done until it can be manually triggered from a file.

## The Rule

| Action | Required |
|---|---|
| New `@RestController` | Create `src/test/http/<module>.http` |
| New method on existing controller | Add request to existing `<module>.http` |
| Changed request/response shape | Update the corresponding request in `.http` |

Never leave endpoint creation as a separate task. Do it in the same step.

## HTTP File Format

```http
### Variables
@baseUrl = http://localhost:8080
@accessToken = <paste_access_token_here>
@resourceId = <paste_id_here>

### Descriptive name of what this does
POST {{baseUrl}}/api/v1/module/endpoint
Content-Type: application/json
Authorization: Bearer {{accessToken}}

{
  "field": "value"
}
```

**Rules for the file content:**
- Variables block at the top with `@baseUrl`, `@accessToken`, and any IDs needed
- One `###` section per endpoint
- Include `Authorization: Bearer {{accessToken}}` only on protected endpoints
- Comment public vs auth-required endpoints
- Use realistic sample values (not `string`, `123`, `test`)

## Common Rationalizations — All Wrong

| Thought | Reality |
|---|---|
| "User didn't ask for .http files" | The convention is always on. No opt-in needed. |
| "I'll add the .http file after" | After never comes. Do it now. |
| "It's a simple endpoint, obvious how to call it" | Obvious to you now ≠ runnable by anyone later. |
| "I'll mention they might want to create one" | Don't suggest — create it. |
