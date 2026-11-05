```yaml
document_id: ADR-0001
title: First Provider — Claude Code
version: 1.0
status: accepted
created_at: 2026-11-05
updated_at: 2026-11-05
owner: Project Team
related_documents:
  - DOC-04 Technology Stack
  - DOC-10 Provider Adapter Specification
  - DOC-11 Workspace & Security Strategy
```

# ADR-0001: First Provider — Claude Code

## Status

Accepted — 2026-11-05

## Context

The roadmap requires integrating exactly one provider in Phase 1 (Agent Runtime) before any multi-provider work. Previous documents named two candidates without a decision:

- `DOCUMENTATION-PLAN.md` preferred Codex CLI.
- `development-roadmap.md` said "preferably Codex CLI or Claude Code".

Several Phase 1 documents depend on this choice: Technology Stack, Provider Adapter Specification, State Machine & Event Architecture, and Workspace & Security Strategy.

## Problem

Which provider should the MVP Agent Runtime integrate first?

## Decision

**Claude Code is the first provider.**

Codex CLI and other providers are deferred to Phase 4 (Multi-provider Support).

The integration mechanism was decided separately in [ADR-0002](ADR-0002-claude-code-cli-subprocess.md): **CLI subprocess**. The options considered were:

| Option | Description |
| ------ | ----------- |
| CLI subprocess | Run `claude -p` with `--output-format stream-json` and parse newline-delimited JSON events. Works from any backend language. |
| Agent SDK | Use the Claude Agent SDK (Python or TypeScript). It runs the Claude Code binary and exposes typed messages, permission callbacks and sessions. Only available if the backend is Python or TypeScript. |

## Supporting Factors

Claude Code's documented non-interactive mode maps directly onto the questions DOC-10 must answer:

| DOC-10 question | Documented Claude Code behavior |
| --------------- | ------------------------------- |
| Detect installation | `claude --version` |
| Detect authentication | `claude auth status` (JSON by default) → `loggedIn`, `authMethod`, `apiProvider`, `subscriptionType`. Does not require reading credential files. Verified locally on v2.1.310 |
| Start a process, send prompt | `claude -p "<prompt>"`, or prompt via stdin |
| Read output | `--output-format json` or `stream-json` (with `--verbose`) |
| Session metadata | First stream event `system/init` reports model, tools, MCP servers and a `capabilities` list for feature detection |
| Final result | Last stream event is a `result` message with result text, `session_id` and cost |
| Success / failure | Exit code 0 on success, non-zero on failure |
| Resume a session | `--resume <session_id>` or `--continue` |
| Structured output | `--json-schema` with `--output-format json` → `structured_output` field |
| Tool permissions | `--allowedTools`, `--permission-mode` (`acceptEdits`, `dontAsk`, `auto`, …), `--permission-prompt-tool`, `PermissionRequest` hooks |
| Unattended runs | `--permission-prompts none` → denials reported as `permission_denied` events |
| Cancellation | SIGINT ends the current turn; SIGTERM exits with code 143 and leaves the turn unfinished |
| Retries | `system/api_retry` events with error category (`rate_limit`, `authentication_failed`, …) |
| Cost | `total_cost_usd` in JSON output — documented as a client-side estimate |
| Reproducible runs | `--bare` skips auto-discovery of hooks, skills, plugins, MCP servers and `CLAUDE.md` |

Source: Claude Code documentation, "Run Claude Code programmatically" and "Agent SDK overview", read on 2026-11-05. `claude --version` and `claude auth status` were run locally against Claude Code v2.1.310. Several flags require a minimum Claude Code version; record the version used in the spike.

## Alternatives Considered

### Codex CLI first

Remains the planned second provider (Phase 4). Not chosen for Phase 1.

### Both providers in Phase 1

Rejected. The roadmap explicitly says to integrate one provider first so the runtime can be proven before the adapter contract is generalized.

## Consequences

### Authentication

Source: Claude Code documentation, "Legal and compliance" → "Authentication and credential use" and "Can customers offer Claude Code in their products?", read on 2026-11-05.

- **Subscription login via the CLI is permitted for the user's own account.** The terms do not prevent "an end user from signing in to the unmodified Claude Code binary with their own Claude subscription, including where a platform hosts Claude Code".
- **What the factory must never do:** offer its own claude.ai login, collect / store / intermediate claude.ai credentials or session tokens, or route requests through a user's Free / Pro / Max / Ultra plan on their behalf. Sign-in must complete through Anthropic's own flow.
- **Conditions when the factory runs Claude Code:**
  - Run the Claude Code binary unmodified, as published by Anthropic.
  - Do not remove, disable or restrict any built-in authentication method.
  - Each end user authenticates with their own API key, Claude subscription or cloud provider credentials (Amazon Bedrock, Google Cloud, Microsoft Foundry), billed directly to that user. The factory does not pay for, resell or intermediate usage.
- **Agent SDK differs:** developers building products with the Agent SDK are directed to use API key authentication. This favors the CLI subprocess option when subscription login must be supported.
- **`--bare` mode** never reads OAuth credentials or the system keychain; it requires `ANTHROPIC_API_KEY` (or an `apiKeyHelper`) or cloud provider credentials. Subscription users therefore cannot use `--bare` (see the Security section).
- **Usage limits:** advertised Pro / Max / Ultra limits assume ordinary, individual usage. Parallel agent instances consume limits quickly; the Scheduler must enforce a configurable concurrency limit.
- The factory must not store, extract or proxy provider credentials (Product Overview §3.10).
- **Personal data in `claude auth status`:** the output also contains the account `email`, `orgId` and `orgName`. The adapter reads only the fields it needs (`loggedIn`, `authMethod`, `apiProvider`, `subscriptionType`) and must not persist or log the account email or organization identifiers.
- **Plan determines the governing terms:** Free, Pro, Max and Ultra are governed by the Consumer Terms; Team, Enterprise and Claude API usage by the Commercial Terms. The plan reported by `subscriptionType` should be shown on the Provider Settings screen so the user knows which terms and usage limits apply.
- Before the public release, confirm the permitted authentication setup with Anthropic (the documentation directs such questions to Anthropic sales).

### Branding

The product may state in plain text that it runs Claude Code. It must not use the Claude Code or Anthropic names or logos in its own product, feature or company name, or in a way that suggests Anthropic built, endorses or partners with it. AI Software Factory keeps its own branding.

### Security (input to DOC-11)

- Without `--bare`, a `-p` run loads the target project's `.claude/settings.json` hooks and `.mcp.json` servers **without a trust prompt**, even in a folder never trusted before. Agents working on untrusted repositories must not load that configuration. `--bare` prevents it but cannot be used with subscription login. `--setting-sources user` also prevents it and works with subscription login: the permissions documentation states that with it "Claude Code reads neither the project's settings files nor its `.mcp.json`".
- In recent versions, `-p` runs start in Manual permission mode: calls that need approval are denied unless approved by a rule, a `PermissionRequest` hook or `--permission-prompt-tool`. This is the integration point for the factory's `ApprovalRequest` flow.

### Runtime and state machine (input to DOC-08)

- `stream-json` events are the raw input for `agent.instance.*` events. The adapter translates them; Claude-specific event names must not leak into the core.
- "Waiting for input" in the factory maps to a pending permission request, not to an interactive terminal prompt.

### Native subagents

Claude Code can spawn its own subagents; their messages carry `parent_tool_use_id`. Per System Architecture §6, factory orchestration must not depend on Claude Code's native subagents, because they are not portable to other providers.

### Provider independence

Because only one provider exists until Phase 4, there is a risk that the adapter contract silently becomes "the Claude Code contract". DOC-10 must define the contract in provider-neutral terms and list Claude-specific capabilities separately.

## Open Questions for the Phase 1 Spike

1. **Cancellation on Windows.** The documented SIGINT / SIGTERM behavior assumes POSIX signals. Verify how a Windows process supervisor can end a turn cleanly versus killing the process tree. Candidate: an `interrupt` control request in long-lived mode (see ADR-0002, Cancellation).
2. ~~**CLI subprocess vs Agent SDK.**~~ Resolved 2026-11-05: CLI subprocess, see [ADR-0002](ADR-0002-claude-code-cli-subprocess.md).
3. **Approval flow.** Prototype `--permission-prompt-tool` or a `PermissionRequest` hook to route approvals into the factory UI.
4. ~~**Project configuration without `--bare`.**~~ Resolved 2026-11-05: `--setting-sources user` (see Security). The spike should confirm that project hooks and `.mcp.json` servers are not loaded.
5. ~~**Authentication detection.**~~ Resolved 2026-11-05: use `claude auth status` (see Supporting Factors). The spike should still confirm its output when logged out and when using an API key.
6. **Minimum supported Claude Code version.** v2.1.310 is the version verified so far. The spike must confirm that every flag listed above works on it.
