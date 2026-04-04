---
name: eshop-dotnet-architecture
description: >-
  Explains and reasons about eShopOnContainers .NET microservice layout, bounded
  contexts, DDD vs CRUD services, integration events, gateways, and Docker/K8s
  deployment shape. Use when the user asks about architecture, service boundaries,
  where code lives, how services communicate, or designing changes in this repo.
---

# eShopOnContainers — .NET architecture lens

## Ground rules

- Treat this repository as the **source of truth**. Prefer **paths and projects under `src/`** over generic microservice advice.
- When describing flows, **name the service(s) and projects** involved and mention **sync (HTTP/gRPC)** vs **async (integration events)** when relevant.
- **Database-per-service**: each microservice owns its data store; do not assume a shared transactional database across services.

## Solution entry points

| Area | Location |
|------|----------|
| Main solution | `src/eShopOnContainers-ServicesAndWebApps.sln` |
| Compose (local stack) | `src/docker-compose*.yml` |
| Kubernetes / Azure | `deploy/k8s/`, `deploy/azure/` |
| High-level overview (diagrams, links) | `README.md` (architecture section + wiki links) |

## Bounded contexts (backend services)

Typical layout: `src/Services/<Context>/<Project>/`.

| Context | API / host projects | Notes |
|---------|---------------------|--------|
| **Catalog** | `Catalog.API` | Product catalog; often simpler data-oriented API. |
| **Basket** | `Basket.API` | Shopping basket; Redis-backed in this sample. |
| **Ordering** | `Ordering.API`, `Ordering.Domain`, `Ordering.Infrastructure`, `Ordering.SignalrHub`, `Ordering.BackgroundTasks` | **DDD-style** vertical slice (domain + infrastructure + API); commands/queries and domain model live here. |
| **Identity** | `Identity.API` | Authentication / identity concerns for clients. |
| **Payment** | `Payment.API` | Payment processing (demo). |
| **Webhooks** | `Webhooks.API` | Webhook handling. |

Confirm behavior by reading the relevant `*API` startup, controllers/integration event handlers, and `docker-compose` service definitions—not from memory alone.

## Building blocks (shared infrastructure)

Under `src/BuildingBlocks/`:

- **Event bus** — `EventBus/`, `EventBusRabbitMQ/`, `EventBusServiceBus/`, `IntegrationEventLogEF/`: **integration events** and outbox-style persistence for reliable publishing (verify exact usage per service in code).
- **Web host** — `WebHostCustomization/WebHost.Customization/`: shared host extensions used by services.
- **Devspaces** — `Devspaces.Support/`: optional dev environment support.

Shared libraries are **technical cross-cutting** pieces, not business “shared domains.” Keep new business rules inside the owning service.

## Clients and API composition

| Role | Location |
|------|----------|
| Web MVC | `src/Web/WebMVC/` |
| Web SPA (Angular) | `src/Web/WebSPA/` |
| Health / status UI | `src/Web/WebStatus/` |
| Webhook demo client | `src/Web/WebhookClient/` |
| **BFF / HTTP aggregators** | `src/ApiGateways/Web.Bff.Shopping/aggregator/`, `src/ApiGateways/Mobile.Bff.Shopping/aggregator/` |

Aggregators compose calls to backend APIs for a specific client experience; they should stay **thin** (orchestration, DTO shaping)—not a second place for domain rules.

## How to answer architecture questions

1. **Identify the bounded context** (which service owns the data and rules).
2. **Trace the call path**: client → gateway/BFF (if any) → service API → domain/application → infrastructure.
3. **Cross-service consistency**: prefer **integration events** and eventual consistency; call out where **sagas** or compensations appear in this codebase if you find them.
4. **Propose changes** that respect **service boundaries** (new table only in owning service, contracts for integration events, version/API impact on gateways).

## Suggested response shape (for non-trivial questions)

Use short sections:

- **Context** — which user journey or feature.
- **Ownership** — which service(s) own data and behavior.
- **Communication** — REST/gRPC vs integration events; which queues/topics if applicable.
- **Trade-offs** — consistency, latency, failure modes.
- **Repo pointers** — specific folders/projects to open next.

## External references

Official wiki and eBooks are linked from `README.md` (e.g. Explore the code, deployment guides). Use those for narrative tutorials; keep answers anchored to **this tree** when the user is working in the repo.
