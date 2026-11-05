```yaml
document_id: ADR-0003
title: Product Positioning — Template-based AI Software Factory
version: 1.1
status: accepted
created_at: 2026-11-05
updated_at: 2026-11-05
owner: Project Team
related_documents:
  - DOC-01 Product Requirements Document
  - DOC-02 MVP Scope
  - DOC-06 Domain Model
```

# ADR-0003: Product Positioning — Template-based AI Software Factory

## Status

Accepted — 2026-11-05

## Context

The [README](../../../README.md) describes a platform where "one request in" leads to "a coordinated team of AI agents [working] through the software engineering process". The original documents already contain the elements of a full lifecycle and of reusable processes, but they do not state how far the product is meant to reach:

- The Orchestrator "determines the necessary development phases" ([Product Overview](../../01-product/product-overview.md) §2.2).
- The Workflow Engine manages predefined workflows such as `Analysis → Architecture → Implementation → Review → Testing → Delivery` ([System Architecture](../system-architecture.md) §3.2).
- Configurable workflows are a goal (README §3.5), and reusable workflows and workflow templates are planned in Phase 7 ([Development Roadmap](../../06-development/development-roadmap.md)).

## Problem

What is the long-term scope of the product?

| Option | Description |
| ------ | ----------- |
| A. Multi-agent coding workspace | Coordinate agents on coding tasks in an existing project |
| B. AI SDLC Platform | Automate the whole software development lifecycle: idea, requirements, documentation, architecture, coding, testing and release |
| C. AI Software Factory | B, plus reusable factory templates: the process itself is reused to produce many different software products |

## Decision

**AI Software Factory is positioned as option C.** Option B is a subset of C and is delivered as part of it.

This decision sets the product direction. It does not change the original documents: the development order, the MVP and the responsibilities of each component remain as they are written there.

## Consistency with the Original Documents

| Element of C | Where it already exists |
| ------------ | ----------------------- |
| Whole lifecycle | Orchestrator determines the development phases (Product Overview §2.2). Workflow example from Analysis to Delivery (System Architecture §3.2) |
| Human control between phases | Plan review and approvals (README §3.3, System Architecture §5) |
| Completion based on verification | Product Overview §2.5 |
| Reusable processes (factory templates) | Configurable workflows (README §3.5). Reusable workflows and workflow templates (Roadmap Phase 7) |

## Consequences

- **Development order is unchanged.** The Roadmap phases and the MVP definition in Roadmap §3 remain the standard. "Build the execution engine first, then build the factory around it" (README §9) still applies.
- **Factory templates are delivered in Phase 7**, as the reusable workflows and workflow templates already planned there. Until then the Workflow Engine runs predefined workflows (System Architecture §3.2, §11).
- **The Orchestrator keeps its role** as described in Product Overview §2.2 and System Architecture §3.2: it determines the necessary phases and plans the work.
- **DOC-06 (Domain Model)** should not make it hard to add workflow templates in Phase 7, but it does not need to model them for the MVP.
- **Lifecycle stages beyond coding** (requirements, documentation, release) are part of the product goal. When and how they are added is decided in the Roadmap, not in this ADR. Actions that reach outside the user's machine, such as publishing a release, follow the approval rules in README §3.3 and System Architecture §9.
