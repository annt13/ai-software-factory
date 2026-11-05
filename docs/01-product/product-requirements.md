```yaml
document_id: DOC-01
title: Product Requirements Document
version: 0.1
status: draft
created_at: 2026-11-05
updated_at: 2026-11-05
owner: Project Team
related_documents:
  - DOC-13 Product Overview
  - DOC-02 MVP Scope
  - DOC-03 Use Cases & User Stories
  - ADR-0001 First Provider — Claude Code
  - ADR-0003 Product Positioning — Template-based AI Software Factory
  - ADR-0005 Application Form — Local Web App
```

# Product Requirements Document

## 1. Purpose

This document states **what** AI Software Factory is, **why** it exists and **who** it is for. It is the reference for product scope when writing the MVP Scope (DOC-02), Use Cases (DOC-03) and the architecture documents.

To avoid repeating content (Documentation Plan §6.1), this document summarizes and links to existing sources instead of copying them:

| Topic | Source of truth |
| ----- | --------------- |
| Vision, principles, target users | [README](../../README.md) §2–§4 |
| Module descriptions | [Product Overview](product-overview.md) §3 |
| Product positioning | [ADR-0003](../02-architecture/adr/ADR-0003-product-positioning-template-based-factory.md) |
| Development order, MVP scenario | [Development Roadmap](../06-development/development-roadmap.md) |
| What exactly is in the MVP | DOC-02 MVP Scope (not yet written) |

Content that is new in this document (Problem Statement, Product Goals, feature IDs) is **draft** until this document is approved.

## 2. Product Vision

> One request in. A coordinated team of AI agents works through the software engineering process. The user supervises and controls the factory.

AI Software Factory is a local-first, provider-agnostic platform that turns a software requirement into coordinated work by multiple AI agents, with the user supervising and approving.

Per ADR-0003, the long-term product covers the whole software development lifecycle and lets users reuse the process itself as **factory templates**. The development order is unchanged: the execution engine comes first, and factory templates arrive in Roadmap Phase 7.

## 3. Problem Statement

*Draft — to be confirmed by the project owner.*

Developers already use AI coding agents, but working with them on anything larger than a single change runs into the following problems:

| # | Problem | Effect today |
| - | ------- | ------------ |
| P1 | **One agent, one session.** Each AI coding tool works as a single assistant in a single context. | The user splits the work, writes the prompts and moves context between sessions by hand. |
| P2 | **No shared state across tools.** Each provider has its own interface, authentication, output format and session model. | The user cannot see in one place what every agent is doing, has done or is waiting for. |
| P3 | **"Done" is not verified.** An agent reports completion based on its own judgment. | The user has to check every result: run the tests, review the changes and decide what to redo. |
| P4 | **Parallel work is unsafe.** Several agents working in the same repository can overwrite each other's changes. | In practice the user runs agents one at a time, or isolates them by hand. |
| P5 | **Failures are hard to manage.** Timeouts, rate limits, permission denials and crashes surface differently in each tool. | Retrying or resuming depends on the user noticing the failure and remembering where the work stopped. |
| P6 | **The process is not reusable.** Analysis, design, implementation, review and testing are repeated by hand for every project. | Good processes live in the user's head instead of in the tool. |

The factory addresses P1–P5 with its own agent runtime, task engine and orchestrator, and P6 with factory templates (Phase 7).

## 4. Target Users

Target users are defined in [README](../../README.md) §4. For requirements and user stories, this document uses one primary persona until the MVP is complete.

**Primary persona: the individual developer (initial target)**

- Works alone on their own projects, on a Windows machine.
- Already has an account with at least one AI coding provider. For the MVP this is Claude Code (ADR-0001), signed in with the user's own subscription or API key.
- Is comfortable with Git, the terminal and reviewing code.
- Wants to give a requirement and supervise the result, but does not want agents to act without control on their machine.

**Secondary personas (future targets):** development teams, open-source contributors who extend the factory with adapters, skills and templates, and teams building internal AI engineering workflows. They do not drive MVP requirements, but the architecture must not prevent them (README §3.5).

## 5. Product Goals

*Draft.* Goals describe outcomes. Measurable acceptance criteria for the MVP belong in DOC-02.

| ID | Goal | Addresses | Evidence of success |
| -- | ---- | --------- | ------------------- |
| G1 | Turn one high-level requirement into a plan of dependent tasks that agents execute | P1 | Product Overview §6: actionable plan, tasks assigned to agent profiles, end-to-end completion of a small task |
| G2 | Keep the user in control of what agents do on their machine | P1, P4 | The user can approve the plan, cancel execution and review changes (README §3.3) |
| G3 | Base completion on verification, not on the agent's own report | P3 | A task is completed only after its acceptance criteria and available checks pass (Product Overview §2.5) |
| G4 | Give one authoritative view of execution state across agents | P2, P5 | Task and agent status, logs and failures come from the runtime and are persisted (README §9) |
| G5 | Run agents safely in parallel on the same project | P4 | No conflicting executions in the same workspace (Roadmap Phase 2) |
| G6 | Recover from failures without losing work | P5 | Failed tasks are surfaced and can be retried safely (Product Overview §6) |
| G7 | Stay independent of any single provider | P2 | A second provider is added in Phase 4 without changing the Orchestrator (Roadmap Phase 4) |
| G8 | Make the development process reusable | P6 | Workflow templates in Phase 7 (ADR-0003) |

G1–G6 apply to the MVP. G7 and G8 shape the architecture now and are delivered later.

## 6. Product Principles

The principles are defined in [README](../../README.md) §3: Local-first, Provider-agnostic, Human-in-the-loop, Observable by design, Extensible and open-source ready.

The following product rules follow from those principles and from accepted decisions. Every feature must respect them:

| Rule | Source |
| ---- | ------ |
| The runtime is the single source of truth for execution. Visual views must never hold their own execution state. | README §9, System Architecture §12 |
| A task is not complete because an agent says so. Completion is based on acceptance criteria and verification results. | Product Overview §2.5 |
| Provider differences are exposed, not hidden. | Product Overview §5 |
| Users sign in through the provider's official mechanism. The factory never stores, extracts or proxies provider credentials. | Product Overview §3.10, ADR-0001 |
| Factory orchestration does not depend on a provider's native subagents. | System Architecture §6 |
| Measured, estimated and unavailable data (for example cost) are shown as different things. | Product Overview §3.9 |

## 7. Core Features

Features are numbered so that DOC-02 and DOC-03 can refer to them. Module descriptions are in [Product Overview](product-overview.md) §3.

The **MVP** column copies the MVP Requirements table in Roadmap §3. **TBD** means the existing documents do not decide it; DOC-02 must decide it.

| ID | Feature | Module | Roadmap phase | MVP |
| -- | ------- | ------ | ------------- | --- |
| F-01 | Create and select a project linked to a local repository | Project Management | 0, 2 | Yes |
| F-02 | Detect the provider CLI, its version and authentication status | Settings | 1 | Yes |
| F-03 | Start an agent execution, stream live output, terminate it, enforce timeouts | Agent Runtime | 1 | Yes |
| F-04 | Persist execution records and execution logs | Agent Runtime, Observability | 1 | Yes |
| F-05 | Submit a requirement; the Orchestrator analyzes the request and the project | Orchestration | 2 | Yes |
| F-06 | Generate an execution plan with tasks, dependencies and agent profiles | Orchestration | 2 | Yes |
| F-07 | Review and approve or reject the execution plan | Orchestration | 2 | TBD |
| F-08 | Manage tasks, subtasks, dependencies and the task lifecycle | Task Management | 2 | Yes |
| F-09 | Schedule ready tasks and assign them to agent instances, sequentially | Orchestration | 2 | Yes |
| F-10 | Run tasks in parallel when it is safe (workspace isolation) | Orchestration, Agent Runtime | 2 | TBD |
| F-11 | Handle approval requests raised by an agent during execution | Agent Runtime | 1–2 | TBD |
| F-12 | Verify results (run configured tests, review by an agent) and create follow-up tasks | Orchestration | 2 | Yes, depth TBD |
| F-13 | Retry and cancel tasks | Task Management | 2 | Yes, depth TBD |
| F-14 | Inspect results, changes and generated artifacts | Task Management | 2 | Yes |
| F-15 | Define agent profiles (role, instructions, tools, permissions, provider, model) | Agent Management | 3 | Yes, configurability TBD |
| F-16 | Skills Library | Skills Library | 3 | No |
| F-17 | Additional providers and provider selection rules | Settings, Agent Runtime | 4 | No |
| F-18 | Command Center | Command Center | 5 | No |
| F-19 | Agent Office visualization | Agent Office | 5 | No |
| F-20 | Usage and cost tracking, execution history views | Observability | 6 | No |
| F-21 | Reusable workflows and factory templates, configurable approval steps | Workflow & Orchestration | 7 | No |
| F-22 | Tools, MCP and Git integration management | Tools & MCP | 7 | No |

## 8. High-level User Journey

The detailed journey is in [Product Overview](product-overview.md) §2 and the example in §4. In summary, with the points where the user is in control:

| Step | User | Factory |
| ---- | ---- | ------- |
| 1. Set up | Selects a local project; signs in to the provider with its own CLI | Detects the provider and its authentication status (F-01, F-02) |
| 2. Request | Submits a requirement | Analyzes the request and the project (F-05) |
| 3. Plan | **Reviews and approves the plan** | Produces tasks, dependencies and agent assignments (F-06, F-07) |
| 4. Execute | Watches progress; **answers approval requests**; can cancel | Schedules tasks, runs agent instances, records state and logs (F-03, F-04, F-08–F-11) |
| 5. Verify | — | Runs checks, creates follow-up tasks for failures (F-12, F-13) |
| 6. Deliver | **Reviews the result and the changes** | Presents results and artifacts (F-14) |

## 9. Success Criteria

**MVP:** the criteria in [Product Overview](product-overview.md) §6. DOC-02 turns them into acceptance criteria.

**Long term** (Roadmap §6), the product succeeds when another developer can:

- Install and run the factory locally with their own provider accounts.
- Add a provider adapter, an agent profile or a skill without changing the core.
- Reuse a factory template to take a new project through the development lifecycle.

## 10. Out of Scope

**Never in scope** (product boundaries):

- Replacing the providers' own coding agents. The factory coordinates them (Product Overview §5).
- Offering its own provider login, or storing, extracting or proxying provider credentials (ADR-0001).
- Paying for, reselling or acting as an intermediary for provider usage (ADR-0001).
- Simulated agent activity that does not come from the runtime (README §9).

**Not in the initial MVP:** see [README](../../README.md) §7. The final MVP boundary is set in DOC-02.

## 11. Constraints and Assumptions

| Type | Statement | Source |
| ---- | --------- | ------ |
| Constraint | The first version runs on one Windows machine, for one user. | README §1, §3.1 |
| Constraint | The first and only MVP provider is Claude Code, run as a CLI subprocess. | ADR-0001, ADR-0002 |
| Constraint | The MVP is a local web app: a local backend owns all execution, and the user works in a browser. | ADR-0005 |
| Constraint | Apart from the factory itself, the user only needs to install the provider CLIs. | ADR-0004 |
| Constraint | The Claude Code binary is run unmodified, and every user authenticates with their own account. | ADR-0001 |
| Constraint | Personal data returned by the provider (for example the account email) is not persisted or logged. | ADR-0001 |
| Assumption | The user has installed the provider CLI and signed in before using the factory. | UC-001 preconditions (Documentation Plan §2.1) |
| Assumption | Provider usage limits apply. Parallel agents use them up faster, so concurrency must be configurable. | ADR-0001 |
| Assumption | The target project is a local Git repository. | Product Overview §3.2 — to be confirmed in DOC-11 |

## 12. Open Questions

Questions for this document:

1. Is the Problem Statement (§3) accurate and complete?
2. Is the primary persona (§4) correct, and should DOC-03 use only this persona?
3. Should Product Goals (§5) have numeric targets, or are the success criteria enough?

Questions passed to DOC-02 (MVP Scope), marked TBD in §7:

4. Does the MVP need parallel execution (F-10)? Product Overview §6 asks for "at least two agent instances" in a coordinated workflow. Roadmap §3 does not say whether they run in parallel. Parallel execution requires workspace isolation (DOC-11).
5. Which approval points are in the MVP: plan approval (F-07), approval requests during execution (F-11), approval before changes are applied?
6. What does "basic verification" (F-12) include: build, tests, lint, agent review?
7. How deep are retry and cancellation in the MVP (F-13)?
8. Are agent profiles configurable in the MVP, or predefined (F-15)?
9. Which lifecycle stages does the MVP cover? Roadmap §3 covers analysis to verification. ADR-0003 leaves the timing of other stages to the Roadmap.

Resolved:

10. ~~What kind of application is the MVP: local web app, desktop app or CLI?~~ Resolved 2026-11-05: local web app, see [ADR-0005](../02-architecture/adr/ADR-0005-application-form-local-web-app.md).
