---
title: "Dev Discussions"
date: 2026-07-24
tags:
  - work
  - meeting
  - solidstart
  - go
  - propeak
publish: false
---

# Dev Discussions

Meeting notes and migration planning.

## June 29, 2026 — Bank Reconciliation POC

See [[bank-reconciliation-poc]].

## July 8, 2026 — Dev discussion (2:00 PM)

### Immediate ask

- Create a basic SolidStart app from the docs
- Create a basic boilerplate Go API
- Assume business rules and conventions are already defined

### ProPeak migration

Existing stack:

- Remix (frontend)
- Node.js (backend)
- MongoDB

Goal: migrate frontend → SolidStart, backend → Go.

#### Process

1. Have the LLM (or handwrite) a features/functionality list documenting each feature.
2. Document the full flow and decide what to build first.

#### SolidStart

- Create all required routes and UI first
- Mock the API

#### Go API

- Paste requirements and functionality to migrate
- Prepare endpoints and internal logic one by one

Related: [[solid-start-documentation]], [[go-fiber-gorm-boilerplate]].

## Related

- [[bank-reconciliation-poc]]
- [[solid-start-documentation]]
- [[go-fiber-gorm-boilerplate]]
- [[qubefini]]
