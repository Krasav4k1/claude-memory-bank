---
name: spring-reactive-logging
description: Use when adding or modifying logging in a Spring Boot WebFlux/R2DBC service — covers log level selection, reactive operator placement, sensitive data rules, AppException vs RuntimeException distinction, cross-module boundary logging, and replacing System.out/System.err
---

# Spring Reactive Logging

## Overview

Use SLF4J via Lombok `@Slf4j`. Never use `System.out`, `System.err`, or `java.util.logging`. Log levels must reflect operational meaning, not developer convenience.

## Framework Setup

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class MyServiceImpl implements MyService {
    // log field injected by Lombok
}
```

## Log Level Decision Table

| Situation | Level |
|---|---|
| Method entry / reactive subscription start | `DEBUG` |
| Significant business event (user registered, token issued, profile created) | `INFO` |
| Expected client error (`AppException` thrown — bad input, not found, duplicate) | `INFO` or omit |
| Cross-module call result (markAsPublic, applyStrike) | `INFO` |
| Domain state transition (POLL_ACTIVE→PUBLIC, user→BANNED) | `INFO` |
| Unexpected `RuntimeException` / system failure | `ERROR` with `Throwable` |
| Fire-and-forget subscribe error callback | `ERROR` with `Throwable` |
| External HTTP call failure (WebClient 4xx/5xx, timeout) | `ERROR` with `Throwable` |

Use `WARN` only when a recoverable anomaly needs ops attention (e.g. suspicious retry storm). Do not use `WARN` for normal validation failures.

## Reactive Operator Placement

Place log calls inside reactive operators — never outside the chain where they would execute eagerly:

```java
// Entry point
return userRepository.findByEmail(email)
    .doOnSubscribe(s -> log.debug("Registering user email={}", email))
    // expected business error — INFO or omit; AppException is handled by GlobalExceptionHandler
    .flatMap(existing -> Mono.<Void>error(new AppException(ErrorCode.VALIDATION_ERROR, "Email already registered")))
    .switchIfEmpty(Mono.defer(() -> {
        // ...
        return userRepository.save(user)
            .doOnSuccess(saved -> log.info("User registered email={}", email))
            .doOnError(e -> log.error("Failed to save user email={}", email, e))
            .then();
    }));
```

**Key operators:**

| Operator | When to use |
|---|---|
| `.doOnSubscribe(s -> log...)` | Log entry/start of chain |
| `.doOnNext(v -> log...)` | Log value produced (e.g. fetched entity) |
| `.doOnSuccess(v -> log...)` | Log successful Mono completion |
| `.doOnError(e -> log.error(..., e))` | Log errors — always pass the throwable as last arg |
| `.doOnError(ExcType.class, e -> ...)` | Filter to specific error type |

## Sensitive Data — Never Log

- Passwords, password hashes
- Raw tokens (JWT access tokens, refresh tokens, OAuth access tokens)
- OTP values
- Full phone numbers (log last 4 digits max, or omit)
- Card / payment data

```java
// OK
log.info("OTP generated for phone ending={}", phone.substring(phone.length() - 4));
// NEVER
log.info("OTP for {}: {}", phone, otp);  // leaks OTP value
log.debug("Bearer token: {}", accessToken);  // leaks credential
```

## AppException vs RuntimeException

`AppException` represents expected business errors handled by `GlobalExceptionHandler` — no `ERROR` log needed (the handler logs if needed). Log unexpected `RuntimeException`s at `ERROR` with the full throwable.

```java
// AppException — expected flow, no ERROR log
return Mono.error(new AppException(ErrorCode.POLL_NOT_ELIGIBLE));

// Unexpected — log ERROR with exception
.doOnError(e -> !(e instanceof AppException),
    e -> log.error("Unexpected error casting vote pollId={}", pollId, e))
```

## EventListener + Fire-and-Forget subscribe()

Event listeners use `subscribe()` which is fire-and-forget. The error callback is the only observability signal — must use `ERROR`:

```java
@EventListener
public void onProfileCreated(ProfileCreated event) {
    log.debug("Received ProfileCreated event profileId={}", event.profileId());
    createForProfile(event.profileId()).subscribe(
        poll -> log.info("Poll created for profileId={} pollId={}", event.profileId(), poll.getId()),
        err -> log.error("Failed to create poll for profileId={}", event.profileId(), err)
    );
}
```

## Cross-Module Calls

Log before and after calls that cross module boundaries (e.g. polls → profiles, polls → users):

```java
.flatMap(saved -> {
    log.info("Poll passed, marking profile public profileId={}", saved.getProfileId());
    return profileService.markAsPublic(saved.getProfileId());
})
.doOnSuccess(v -> log.info("Profile marked public profileId={}", profileId))
.doOnError(e -> log.error("Failed to mark profile public profileId={}", profileId, e))
```

## External HTTP Calls (WebClient)

```java
return webClient.get()
    .uri("/oauth2/v3/userinfo")
    .headers(h -> h.setBearerAuth(accessToken))  // never log accessToken
    .retrieve()
    .bodyToMono(Map.class)
    .doOnSubscribe(s -> log.debug("Fetching Google userinfo"))
    .doOnNext(body -> log.debug("Google userinfo received sub={}", body.get("sub")))
    .doOnError(e -> log.error("Google userinfo request failed", e))
    .map(body -> new OAuthUserInfo(...));
```

## Common Mistakes

| Mistake | Fix |
|---|---|
| `System.out.println(...)` | `log.info(...)` |
| `System.err.println(...)` | `log.error(..., err)` |
| Logging outside reactive chain (eager) | Wrap in `.doOnSubscribe`/`.doOnNext`/etc. |
| `log.error("msg: " + e.getMessage())` | `log.error("msg", e)` — pass throwable, not message |
| Logging OTP/token values | Log only non-sensitive identifiers |
| `ERROR` for `AppException` | `AppException` is expected flow — omit or use `INFO` |
| Missing throwable in `ERROR` | Always: `log.error("msg context={}", val, exception)` |