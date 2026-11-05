```yaml
document_id: ADR-0005
title: Application Form — Local Web App
version: 1.0
status: accepted
created_at: 2026-11-05
updated_at: 2026-11-05
owner: Project Team
related_documents:
  - DOC-01 Product Requirements Document
  - DOC-04 Technology Stack
  - DOC-05 Component Architecture
  - DOC-11 Workspace & Security Strategy
```

# ADR-0005: Application Form — Local Web App

## Status

Accepted — 2026-11-05 (decided by the project owner)

## Context

[DOC-01](../../01-product/product-requirements.md) §12 left open what kind of application the MVP is. DOC-04 (Technology Stack) cannot choose a frontend framework, backend framework or packaging strategy until this is known.

The existing architecture already points in one direction:

- System Architecture §8 shows the initial deployment as a **Web UI** talking to a **Local Application Backend**, which owns the database and the Agent Runtime.
- System Architecture §3.1: the UI talks to the backend through a defined API, receives real-time updates through an event channel, and must not launch provider processes or control task execution.
- System Architecture §10: a future version may move orchestration to a server and add remote workers and team collaboration.

## Problem

What form does the MVP application take?

## Decision

**The MVP is a local web app**: a backend process that runs on the user's machine, and a UI that the user opens in a browser.

- The backend owns all execution: the Application Core, the Agent Runtime, provider processes and the local database.
- The browser UI is a client only. It reads state through the API and the event channel and sends user commands (submit, approve, cancel). Closing the browser does not stop or change any execution.
- The MVP has no desktop shell and no CLI front end.

## Alternatives Considered

| Option | Description | Result |
| ------ | ----------- | ------ |
| Local web app | Local backend, UI in the browser | **Chosen** |
| Desktop app | Native window (for example Electron or Tauri) around the UI, with the backend embedded | Not for the MVP: adds packaging, signing and auto-update work that the MVP does not need. A desktop shell can be added later around the same web UI |
| CLI only | Terminal interface | Rejected: plan review, task board, approvals, live logs and later the Agent Office need a visual interface (Product Overview §3) |

## Consequences

- **Matches the existing architecture.** System Architecture §3.1 and §8 do not change.
- **DOC-04** chooses a web frontend framework, a backend framework, the API style and the real-time channel (for example WebSocket or Server-Sent Events). ADR-0002 already keeps the backend language open.
- **Security (input to DOC-11).** A local web server that can start agents on the user's machine is a high-value target for other web pages open in the same browser. DOC-11 must define at least:
  - The backend listens on the loopback interface only, never on external network interfaces.
  - Requests are checked against the expected `Host` and `Origin`, to prevent cross-site requests and DNS rebinding.
  - The UI must present a local secret (for example a token generated at startup) before it can call the API or open the event channel.
- **Lifecycle (input to DOC-08).** Executions belong to the backend process, not to a browser tab. DOC-08 must define what happens to running agent processes when the backend stops or crashes, and how state is restored when it starts again.
- **Real-time state.** Every view, including several open tabs, shows the same runtime state through the event channel. This follows README §9.
- **Future.** The same split (backend API plus browser UI) fits server-hosted orchestration later (System Architecture §10), and can be wrapped in a desktop shell without rewriting the UI.
