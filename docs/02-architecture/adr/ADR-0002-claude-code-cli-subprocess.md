```yaml
document_id: ADR-0002
title: Claude Code Integration via CLI Subprocess
version: 1.0
status: accepted
created_at: 2026-11-05
updated_at: 2026-11-05
owner: Project Team
related_documents:
  - ADR-0001 First Provider — Claude Code
  - DOC-04 Technology Stack
  - DOC-10 Provider Adapter Specification
  - DOC-11 Workspace & Security Strategy
  - DOC-12 Testing Strategy
```

# ADR-0002: Claude Code Integration via CLI Subprocess

## Status

Accepted — 2026-11-05

## Context

[ADR-0001](ADR-0001-first-provider-claude-code.md) selected Claude Code as the first provider and left two integration mechanisms open:

- **CLI subprocess:** run `claude -p` and parse its JSON output.
- **Agent SDK:** use the Claude Agent SDK library (Python or TypeScript).

## Problem

How should the Claude Code Adapter launch and control Claude Code executions?

## Decision

**The Claude Code Adapter runs the user's installed, unmodified `claude` executable as a child process in non-interactive mode.**

- The adapter starts Claude Code in non-interactive print mode with `--output-format stream-json` and `--verbose`. Whether each execution attempt uses a one-shot process or a long-lived process is decided in the Phase 1 spike (see Process model).
- The prompt is written to the process's stdin, not passed as a command-line argument.
- stdout is parsed as newline-delimited JSON events. stderr is captured as diagnostic logs.
- Sessions are continued with `--resume <session_id>`, using the `session_id` reported by the previous run.
- Installation is detected with `claude --version`. Authentication is detected with `claude auth status` (see ADR-0001).
- Authentication is handled entirely by Claude Code. The adapter never reads, stores or forwards credentials.

## Rationale

1. **Supports subscription login.** Users may sign in to the unmodified Claude Code binary with their own Claude subscription. Products built on the Agent SDK are directed to use API key authentication instead (ADR-0001, Authentication).
2. **Language-independent.** Any backend language can launch a process and parse JSON. The Technology Stack decision (DOC-04) is not constrained by SDK availability.
3. **Matches the architecture.** System Architecture §11 already sets "Official local CLI tools first". The Runtime Manager owns the process lifecycle directly.
4. **Same pattern for later providers.** Other coding CLIs are expected to be integrated as child processes as well. This must be verified per provider in Phase 4.

## Alternatives Considered

### Claude Agent SDK

Provides typed messages, a `canUseTool` permission callback and an `interrupt()` method. Not chosen because it is directed to API key authentication for products and would tie the backend to Python or TypeScript. It may be reconsidered as an additional adapter for API-key-only deployments.

### Driving the interactive terminal UI through a pseudo-terminal

Rejected. It would require screen-scraping terminal output, which provides no structured events and breaks whenever the UI changes.

## Consequences

### Executable resolution

- A machine can have more than one Claude Code installation. The development machine has both a native installation (`claude.exe`, v2.1.310) and an npm installation (`claude.cmd`, v2.1.270). Which one `claude` resolves to depends on `PATH` order.
- The adapter resolves an explicit executable path during detection, stores it in Provider Settings and uses that path for every execution. The user can override it.
- The Claude Code version is recorded on every Execution so behavior differences can be traced.
- Prefer a native executable. A `.cmd` shim requires launching through `cmd.exe`, which adds argument-quoting and command-injection risks; if a shim must be used, no user-controlled text may appear in the command line.

### Prompt delivery

The prompt goes through stdin because Windows limits command-line length and has fragile quoting rules. Piped stdin is capped at 10 MB by Claude Code; larger context must be written to files in the workspace and referenced in the prompt.

### Output parsing

- The factory owns a parser for the `stream-json` format. The format can change between Claude Code versions.
- The parser ignores unknown event types and fields instead of failing.
- Optional behavior is feature-detected through the `capabilities` list in the `system/init` event rather than by comparing version strings.
- The adapter maps Claude Code events to the factory's normalized execution events. Claude-specific event names do not leave the adapter.

### Process model

Two process models are possible. The Phase 1 spike chooses one and records it in DOC-10.

| Model | Description | Trade-offs |
| ----- | ----------- | ---------- |
| One-shot | One `claude -p` process per execution attempt; follow-up turns use `--resume` | Simplest. Cancellation needs an OS signal or killing the process |
| Long-lived | One process per agent instance with `--input-format stream-json`; user turns are sent as JSON lines on stdin | Documented as supporting multiple queued messages, interruptions and permission requests in one process. The raw stdin message format is only documented through Agent SDK types |

Prior art: Orca runs Claude Code in the long-lived model (see [Orca Design Study](../../research/orca-design-study.md) §3.2).

### Approvals

There is no in-process permission callback as in the Agent SDK. Options:

| Option | Documentation status |
| ------ | -------------------- |
| `--permission-prompt-tool mcp__<server>__<tool>`: the factory exposes an MCP server whose tool answers permission requests | Documented |
| `PermissionRequest` hook | Documented |
| `--permission-prompt-tool stdio`: permission requests arrive as `control_request` messages on stdout and are answered on stdin (used by Orca) | **Not documented** for the CLI; it is the Agent SDK's internal transport |

Prefer a documented option. If the `stdio` transport is used, isolate it inside the adapter, gate it on the Claude Code version and cover it with regression tests.

### Cancellation

Each execution is an OS process controlled by the Runtime Manager.

- Documented graceful cancellation in one-shot mode relies on SIGINT, which Windows does not provide in the same way. SIGTERM exits with code 143 and leaves the turn unfinished.
- In the long-lived model, a turn can be interrupted with an `interrupt` control request on stdin. This is documented through the Agent SDK's `interrupt()` ("only available in streaming input mode"); support is advertised by the `interrupt_receipt_v1` capability in `system/init`. The raw message format is not documented.

The Phase 1 spike must define how to end a turn cleanly on Windows and when to fall back to terminating the process tree.

### Concurrency

One agent instance runs one `claude` process at a time. The Scheduler's concurrency limit therefore also bounds the number of concurrent Claude Code processes and the rate at which subscription usage is consumed.

### Testing (input to DOC-12)

The adapter can be tested without calling Claude by:

- Replaying recorded `stream-json` output as fixtures.
- Substituting a fake executable that emits scripted events, exit codes and delays.

Only a small set of end-to-end tests needs a real, signed-in Claude Code installation.
