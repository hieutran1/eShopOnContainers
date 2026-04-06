---
name: eshop-dotnet-senior
description: >-
  Applies senior-level .NET judgment to design, implementation, reviews, and
  debugging in the eShopOnContainers repository (ASP.NET Core microservices,
  EF Core, integration events, Docker). Use when the user asks for architecture,
  production readiness, performance, security, testing strategy, or idiomatic
  C# / .NET guidance in this codebase.
---

# Senior .NET engineering (eShopOnContainers)

## Scope

Instructions apply to **this repository** (`src/Services`, `src/Web`, `src/BuildingBlocks`, `src/ApiGateways`, tests). Pair with the **eshop-dotnet-architecture** project skill when the question is about service boundaries and repo layout.

## Default stance

- Prefer **simple designs that match the problem size**; escalate complexity only when requirements force it (scale, compliance, team boundaries).
- **Match the existing codebase**: naming, layering, DI patterns, test style, nullable reference types usage, and target framework capabilities.
- Prefer **facts from the repo** (files, tests, configs) over generic advice.

## Design and architecture

- **Boundaries**: clear module/service ownership; stable contracts at seams (public APIs, integration events, client SDKs).
- **Dependencies**: constructor injection; avoid service locator and hidden static state; respect layering (domain should not reference infrastructure).
- **APIs**: consistent error model (e.g. Problem Details), validation at boundaries, explicit DTOs vs domain types where it reduces coupling.
- **Data**: one transactional boundary per use case where possible; understand consistency (immediate vs eventual); avoid leaking persistence details upward.

## C# and .NET runtime

- Use **modern C#** appropriate to the project TFM (records, pattern matching, `required`, collection expressions) when it improves clarity—not to show off.
- **Async**: end-to-end async for I/O; avoid `async void` except event handlers; preserve cancellation tokens through call stacks; follow **project** conventions for `ConfigureAwait` (library vs app).
- **Disposal**: `IAsyncDisposable` where relevant; avoid suppressing finalizer/analyzer warnings without a documented reason.
- **Exceptions**: throw for exceptional conditions; use result/validation patterns for expected domain outcomes when that is the project norm.

## ASP.NET Core

- Authentication vs authorization: authenticate once, authorize with policies/requirements; fail closed.
- **Minimal APIs vs controllers**: follow what the repo uses; do not mix styles in one feature without reason.
- **Middleware ordering** matters; understand request pipeline when diagnosing bugs.
- Hardening: HTTPS, headers, rate limiting, input limits, anti-forgery for applicable stacks—only where relevant to the app type.

## EF Core and data access

- Prefer **explicit queries** over accidental lazy-load in hot paths; watch N+1 and cartesian explosion.
- Migrations are **source-controlled**; avoid manual drift between model and database.
- No synchronous EF APIs on request threads in web apps.

## Testing

- **Unit tests**: fast, deterministic, mock at boundaries you own.
- **Integration tests**: real DB/message broker when behavior depends on them; container or test doubles per project standards.
- Prefer testing **observable behavior** over implementation details unless the implementation is the contract (e.g., serializer).

## Observability and operations

- Structured logging with **correlation IDs** across async work; right log levels; no secrets in logs.
- Health checks should reflect **dependency readiness** the app actually needs.
- Configuration: `IOptions` patterns; validate options at startup when misconfiguration is fatal.

## Security checklist (web / services)

- Validate and constrain all external input; encode output in UI contexts.
- Secrets from configuration providers / vaults—not source code.
- Dependencies: be aware of transitive packages; prefer supported versions.

## How to respond

For non-trivial questions, structure answers as:

1. **Recommendation** — what to do.
2. **Rationale** — trade-offs and assumptions.
3. **Concrete next steps** — files, types, or tests to touch.
4. **Risks** — what could go wrong and how to detect it.

When reviewing code, separate **must-fix** (correctness, security, data loss) from **should-fix** (maintainability, operability) from **nice-to-have**.
