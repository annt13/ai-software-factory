# AI Software Factory

An open-source, provider-agnostic, multi-agent software engineering workspace.

## 1. Introduction

AI Software Factory is a local-first platform designed to coordinate multiple AI agents and automate software development workflows.

Instead of interacting with a single AI coding assistant, users can provide a high-level requirement to a central Orchestrator. The Orchestrator analyzes the request, decomposes it into tasks, delegates work to specialized agents, coordinates execution, reviews results, and tracks the overall progress.

The platform also provides a visual workspace where users can observe agents, tasks, workflows, execution status, and system activity.

The initial version is designed to run on an individual's Windows machine. The long-term goal is to release the project as an open-source platform that other developers can install, configure, extend, and use with their own AI providers.

## 2. Vision

Build a personal AI Software Factory that can:

- Understand high-level software requirements.
- Break requirements into manageable tasks and subtasks.
- Assign tasks to specialized AI agents.
- Coordinate multiple agents and execution processes.
- Generate, modify, and review source code.
- Run tests and analyze failures.
- Ask for human approval when necessary.
- Track progress, logs, artifacts, token usage, and costs.
- Provide a visual representation of the running agents.
- Support different AI providers without locking the platform to one vendor.

**Core idea:**

> One request in. A coordinated team of AI agents works through the software engineering process. The user supervises and controls the factory.

## 3. Product Principles

### 3.1 Local-first

The first version runs on the user's personal machine.

- No mandatory cloud infrastructure.
- Agents can execute through locally installed official CLI tools.
- Projects and workspaces remain under the user's control.
- Provider authentication is handled by the official provider tools.

### 3.2 Provider-agnostic

The platform must not depend on a single AI provider.

Potential integrations include:

- OpenAI Codex
- Anthropic Claude Code
- Google Antigravity and other supported Google AI tools
- Other providers through future adapters

Each provider is integrated through a normalized adapter interface.

### 3.3 Human-in-the-loop

The system should support autonomous execution while keeping users in control.

Users can:

- Review the execution plan.
- Approve or reject important actions.
- Pause or resume tasks.
- Cancel agent execution.
- Inspect generated artifacts.
- Review changes before they are applied or merged.

### 3.4 Observable by design

Every meaningful execution should produce trackable state and events.

The system should know:

- Which agent is running.
- What task it is working on.
- Which provider and model are being used.
- What the agent is waiting for.
- Whether execution succeeded or failed.
- What artifacts were produced.
- How much time and usage were consumed, when available.

### 3.5 Extensible and open-source ready

The system should have clear boundaries between its core runtime and external integrations.

It should eventually support:

- Custom agent profiles.
- Custom skills.
- Custom tools and MCP servers.
- Custom provider adapters.
- Community extensions.
- Configurable workflows.

## 4. Target Users

Initial target:

- Individual developers.
- Developers experimenting with multi-agent workflows.
- Developers who use multiple AI coding tools.
- Technical users who want to supervise AI-driven software development.

Future target:

- Development teams.
- Open-source contributors.
- Teams building internal AI engineering workflows.

## 5. High-level Capabilities


| Module             | Responsibility                                          |
| ------------------ | ------------------------------------------------------- |
| Command Center     | Overall factory status and activity                     |
| Project Management | Manage projects and their workspaces                    |
| Task Management    | Manage tasks, subtasks, dependencies and execution      |
| Agent Management   | Configure agent profiles and runtime instances          |
| Agent Office       | Visualize agents and their actual runtime states        |
| Orchestration      | Plan, delegate and coordinate agent work                |
| Skills Library     | Manage reusable agent instructions and capabilities     |
| Tools &amp; MCP    | Manage external tools and integrations                  |
| Observability      | Logs, events, execution history, usage and errors       |
| Settings           | Providers, models, permissions and system configuration |


## 6. Development Strategy

The project will be developed incrementally:

1. Build a working local agent runtime.
2. Implement task management and orchestration.
3. Integrate one provider first.
4. Add specialized agent profiles.
5. Add provider adapters for additional providers.
6. Build the Agent Office visualization.
7. Add observability and workflow management.
8. Prepare installation, documentation and open-source release.

The initial goal is not to build every feature. It is to prove that multiple agents can reliably collaborate on a real software development task.

## 7. Non-goals for the Initial MVP

The first version does not need:

- Multi-user collaboration.
- Enterprise organization management.
- Cloud-hosted agent execution.
- Distributed worker clusters.
- A marketplace for agents and skills.
- A fully visual workflow editor.
- Support for every AI provider.
- A complex permission management system.

These capabilities can be considered after the local-first foundation is stable.

## 8. Documentation

- [Product Overview](https://chatgpt.com/c/docs/01-product-overview.md)
- [System Architecture](https://chatgpt.com/c/docs/02-system-architecture.md)
- [Development Roadmap](https://chatgpt.com/c/docs/03-development-roadmap.md)

## 9. Guiding Principle

Build the execution engine first, then build the factory around it.

The dashboard, task board and Agent Office must all reflect the same authoritative runtime state. Visual representations must never become an independent source of execution truth.