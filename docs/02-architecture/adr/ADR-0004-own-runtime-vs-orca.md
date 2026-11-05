```yaml
document_id: ADR-0004
title: Own Agent Runtime vs Building on Orca
version: 1.1
status: accepted
created_at: 2026-11-05
updated_at: 2026-11-05
owner: Project Team
related_documents:
  - ADR-0002 Claude Code Integration via CLI Subprocess
  - DOC-04 Technology Stack
  - DOC-05 Component Architecture
  - RES-001 Orca Design Study
```

# ADR-0004: Own Agent Runtime vs Building on Orca

## Status

Accepted — 2026-11-05

## Context

The [Orca Design Study](../../research/orca-design-study.md) §9 asked whether the factory should use Orca (MIT) as its execution layer instead of writing its own Runtime Manager.

Orca already provides, according to the study:

- About 35 supported agents, including Claude Code and Codex, using each user's own subscription.
- Structured chat with Claude Code, with a terminal fallback.
- Git worktree management, remote hosts and worker monitoring.
- Its own Run / Task / Dispatch / Worker model with fencing, liveness states and gates.

[ADR-0002](ADR-0002-claude-code-cli-subprocess.md) assumes that the factory's Runtime Manager starts Claude Code itself. System Architecture §3 assigns task state to the Task Manager and runtime state to the Runtime Manager.

## Problem

Should the factory own the agent runtime, or delegate agent execution to Orca?

## Decision

**The factory owns its agent runtime and has no dependency on Orca.**

- The Runtime Manager starts provider CLIs directly, as defined in ADR-0002.
- The factory does not use Orca's code, CLI, runtime, protocol or data, and does not require Orca to be installed. This applies to the MVP and to later phases.
- **Orca is a reference only for how to connect to and use agent providers:** how a provider CLI is launched, which flags and input/output formats are used, how sessions, approvals and interruptions are handled, and how provider differences are exposed (Study §3).
- Other parts of Orca's design (domain model, lifecycle, coordinator protocol; Study §4–§7) are not a reference. If a factory design document adopts a similar mechanism, it must justify it on its own merits.

## Options Considered

| Option | Description | Result |
| ------ | ----------- | ------ |
| 1. Own runtime | The Runtime Manager starts provider CLIs directly. Orca is used only as a reference for provider integration | **Chosen** |
| 2. Orca as execution layer | The factory calls the `orca` CLI to create worktrees, launch agents and read results | Rejected |
| 3. Fork Orca | Start from Orca's source code and add factory features to it | Rejected |
| 4. Own runtime, Orca as optional backend later | Option 1 now, with an Orca backend added later behind the Runtime Manager contract | Rejected: it would still create a dependency on Orca |

## Rationale

| Criterion | 1. Own | 2. Orca layer | 3. Fork |
| --------- | ------ | ------------- | ------- |
| Single source of truth for execution state (System Architecture §12) | Yes | No: Orca's Task/Dispatch state must be mirrored and kept in sync | Yes |
| Task and runtime state owned by the factory (System Architecture §3, §11) | Yes | Partly: execution steps go through Orca's protocol | Yes |
| Effort for Phase 1 (one provider) | Medium | Low | High: large Electron codebase with a different product shape |
| Effort for Phase 4 (more providers) | One adapter per provider | Low: many agents already supported | Low |
| Extra installation for users | None beyond provider CLIs | Orca runtime (desktop app or `orca serve`) | None |
| Exposure to another project's changes | Provider CLIs only | Provider CLIs and the Orca CLI contract | Upstream merges |
| Open-source independence | Full | Depends on Orca | Full, with MIT attribution |

- "The runtime is the source of truth for execution" (System Architecture §12). Option 2 creates two state machines, Orca's and the factory's, that must agree.
- Phase 1 needs only one provider, and ADR-0002 already defines how to run it.
- Option 1 costs more provider adapter work in Phase 4. This cost is accepted in exchange for independence.

## Consequences

- ADR-0002 remains valid. The Phase 1 spike proceeds as planned.
- Each additional provider in Phase 4 needs its own adapter. Study §3 is a starting point for how each provider is launched and controlled, but behavior must be verified against the provider's own documentation.
- Where Orca relies on undocumented provider behavior (Study §3.2), the factory prefers documented mechanisms, as already stated in ADR-0002.
- DOC-05 still defines the Runtime Manager contract in terms that do not assume a local child process, so that remote workers can implement it later (System Architecture §10). This is not intended for an Orca backend.
