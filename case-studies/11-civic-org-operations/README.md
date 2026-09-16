# CivicOrgOps / 公民系統

> Status: **Recovered archive / needs review**. This document preserves architecture knowledge recovered from an inactive ChatGPT Project. It does not claim production readiness or complete conversation coverage.

## 1. Executive Summary

CivicOrgOps is a modular civic-organization operations platform originally explored as「公民組織營運平台」and later named「公民系統」. Recovered requirements include multi-role RBAC, an Azure deployment target, read replicas, asynchronous processing, auto-scaling, and a scale target described as up to millions of members. Product prototypes cover finance/political donations, materials ERP, mobilization, membership/permissions, legal/RAG, petitions, local shops, broadcasting, and campaign-operations views.

The most stable recovered architecture decision is an **API-first migration path**: Django REST Framework + OpenAPI remain the domain contract while early Django/static templates act as low-coupling prototypes; a later Vue frontend can reuse the same APIs. Historical RBAC and OpenAPI artifacts were reportedly produced, but their original files are not currently available, so the authorization model remains a migration gap.

## 2. Requirements

### Functional requirements

- Multi-module civic organization operations.
- Role-based access control across modules.
- Finance / political-donation operations.
- Materials / inventory ERP.
- Event and volunteer mobilization.
- Membership and permission administration.
- Legal knowledge / RAG workflows.
- Petition / case tracking.
- Local-shop list management.
- Campaign-oriented broadcasting / operations surfaces.

### Non-functional requirements recovered

- Azure deployment.
- Scale target: up to million-level membership.
- Read replicas for read scaling.
- Async processing for non-blocking/background work.
- Auto-scaling.
- Server-side authorization as the security boundary.

### Out of scope / uncertain

Earlier discussions also referenced a civic forum, verified-truth database, and a「中央廚房」counter-disinformation concept. The recovered source set does not establish whether these remain in scope.

## 3. Design Evolution

### Earlier product framing — back-office administration

A prototype centered on a backend-management dashboard with modules for finance, materials, mobilization, membership, legal, petitions, and local shops.

### Later or parallel framing — campaign operations

Another prototype used an「選戰作戰室」entry point with one-click broadcast, fundraising, materials dispatch, live KPIs, and participant-facing mobilization pages.

### Current interpretation

The evidence supports **persona / information-architecture evolution**, but does not prove whether the campaign UI superseded the admin UI or whether they were intended to coexist for different roles.

## 4. API-first Architecture Decision

### Context

The project already used Django, while a future Vue migration was being considered. The implementation team and final frontend direction were not fixed.

### Options

1. Couple domain behavior directly to Django templates.
2. Build a full Vue SPA immediately.
3. Keep a stable API contract and use lightweight templates as an interim prototype.

### Decision

Use an API-first architecture:

- Django REST Framework for APIs.
- `drf-spectacular` / OpenAPI for schema and contract generation.
- Low-coupling template prototypes using explicit JSON/mount boundaries.
- Vue/Vite can later replace the presentation layer without rewriting the domain API.

### Rationale

- Preserves prototype speed.
- Avoids frontend lock-in.
- Supports future web/mobile/integration clients.
- Makes RBAC and contract testing explicit.

### Trade-offs

- More API/schema discipline is required early.
- Prototype development has an extra abstraction layer.
- Schema drift becomes a failure mode that needs CI validation.

### Failure modes

- Templates bypass the API and read ORM internals directly.
- OpenAPI schema drifts from implementation.
- Frontend visibility rules are mistaken for authorization.
- Permission metadata diverges from backend enforcement.

### Revisit conditions

- If the product remains a single server-rendered Django UI indefinitely, SPA migration benefits decrease.
- If multiple clients emerge, the API-first boundary becomes more valuable.

## 5. RBAC Recovery

Historical work reportedly included:

- detailed RBAC design;
- a complete REST endpoint list;
- generated `RBAC.md`;
- generated `openapi.yaml`;
- admin / mayor / member / guest role definitions;
- another earlier role vocabulary: maintainer / mayor-candidate-KOL / ordinary member;
- a DRF direction using JWT, ViewSets, permission maps, schema-level permission metadata, and CI schema export.

**Gap:** the original `RBAC.md` and `openapi.yaml` are not part of the currently recoverable artifact set. Role-model lineage therefore remains unresolved.

## 6. Scale and Azure Requirements

Recovered system-design requirements include:

- million-level membership;
- RBAC;
- read replicas;
- async processing;
- auto-scaling;
- Microsoft Azure deployment.

These are **requirements**, not evidence that the corresponding production architecture was deployed or benchmarked.

### Validation still required

- DAU / peak QPS model.
- read/write ratio by module.
- database consistency boundaries.
- queue/event-backbone selection.
- autoscale signals and limits.
- RPO / RTO.
- multi-region strategy.
- cost model.
- threat model, identity, DDoS, secrets, and audit requirements.

## 7. Prototype Evidence

Recovered files indicate a static UX prototype with HTML/CSS/JavaScript and mock in-memory data. `app.js` contains mock donations, inventory, events, and member state plus rendering functions. This should be treated as UX/routing evidence, not backend implementation.

See [`prototype-inventory.md`](prototype-inventory.md) for provenance and version ambiguity.

## 8. Reusable Architecture Lessons

1. **Prototype UI should not define the domain boundary.** A disposable presentation layer can be useful, but the contract should survive a frontend rewrite.
2. **RBAC must be enforced server-side.** UI-level hiding is not an authorization boundary.
3. **Filename equality is not sufficient for deduplication.** Same-name artifacts may represent different personas, branches, or design iterations.

## 9. Open Questions

- Are admin dashboard, campaign war room, and participant center separate role-specific surfaces or successive prototypes?
- Which earlier modules were formally removed from scope?
- How do the two recovered role vocabularies map to each other?
- Which reads can tolerate replica lag?
- What async jobs/events require a queue or event bus?
- What are the actual traffic, storage, availability, and cost targets?
- What is the final Azure identity/network/security model?

## 10. Unfinished Work

1. Recover original `RBAC.md` and `openapi.yaml`.
2. Establish prototype lineage and superseded-by relationships.
3. Rebuild a confirmed module/role matrix.
4. Add explicit scale assumptions and SLOs.
5. Validate database, queue, autoscale, RPO/RTO, security, and cost choices.

## 11. Archive Status

This GitHub document preserves the architecture material that is currently recoverable, but **the source ChatGPT Project should not yet be deleted** because complete conversation coverage cannot be proven and original RBAC/OpenAPI artifacts are still missing.
