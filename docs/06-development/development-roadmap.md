```yaml
document_id: DOC-15
title: Development Roadmap
version: 1.0
status: draft
created_at: 2026-11-05
updated_at: 2026-11-05
owner: Project Team
related_documents:
  - DOC-02 MVP Scope
  - DOC-13 Product Overview
  - ADR-0001 First Provider — Claude Code
  - ADR-0003 Product Positioning — Template-based AI Software Factory
```

# Development Roadmap

## 1. Development Strategy

AI Software Factory will be developed incrementally.

The priority is to establish a reliable execution foundation before building advanced visualization, workflow editors or distributed infrastructure.

Development follows this principle:

**Make agents work first. Make the factory visible second. Make the factory extensible third.**

## 2. Phase Overview


| Phase   | Focus                               | Priority  |
| ------- | ----------------------------------- | --------- |
| Phase 0 | Project Foundation                  | Essential |
| Phase 1 | Agent Runtime                       | Critical  |
| Phase 2 | Task Management &amp; Orchestration | Critical  |
| Phase 3 | Agent Management &amp; Skills       | High      |
| Phase 4 | Multi-provider Support              | High      |
| Phase 5 | Agent Office &amp; Dashboard        | High      |
| Phase 6 | Observability                       | High      |
| Phase 7 | Workflow &amp; Integrations         | Medium    |
| Phase 8 | Open-source Preparation             | Future    |


---

## Phase 0 — Project Foundation

### Goal

Establish the repository, technical foundation and development conventions.

### Scope

- Create repository structure.
- Define application architecture.
- Define core domain contracts.
- Set up frontend and backend.
- Set up local database.
- Establish configuration management.
- Establish logging conventions.
- Create initial documentation.

### Expected Result

A working application shell with a clear project structure and a foundation for future modules.

### Not in Scope

- Multi-agent execution.
- Advanced orchestration.
- Agent Office visualization.

---

## Phase 1 — Agent Runtime

### Goal

Execute a real AI coding agent from the local application.

This is the most important technical milestone.

### Scope

- Integrate one provider first.
- Detect the installed CLI.
- Check authentication status using supported mechanisms.
- Start and terminate provider processes.
- Send input and receive output.
- Capture standard output and standard error.
- Track process lifecycle.
- Handle execution timeouts.
- Handle errors and unexpected termination.
- Persist execution records.

### Initial Provider

Start with Claude Code (see [ADR-0001](../02-architecture/adr/ADR-0001-first-provider-claude-code.md)). Codex CLI is planned as the second provider in Phase 4.

Do not integrate every provider at this stage.

### Expected Result

The user can submit a prompt from the application and see a real local agent execute it against a selected project.

### Acceptance Criteria

- The application can detect the configured CLI.
- The application can start an execution.
- The user can see live output.
- The user can terminate execution.
- Execution status is persisted.
- Failed executions are reported correctly.
- The application does not need to store provider subscription credentials.

---

## Phase 2 — Task Management &amp; Orchestration

### Goal

Turn individual agent execution into a coordinated task workflow.

### Scope

- Create tasks.
- Create subtasks.
- Define task dependencies.
- Define task priorities.
- Implement task lifecycle.
- Implement execution planning.
- Assign tasks to agent profiles.
- Implement a basic scheduler.
- Implement a central Orchestrator.
- Support sequential execution.
- Support safe parallel execution.
- Support retry and cancellation.
- Collect task results.

### Initial Workflow

```
User Request
    |
    v
Orchestrator Analysis
    |
    v
Execution Plan
    |
    v
Task Creation
    |
    v
Task Scheduling
    |
    v
Agent Assignment
    |
    v
Agent Execution
    |
    v
Result Collection
    |
    v
Verification
    |
    v
Final Delivery
```

### Expected Result

The user can submit one high-level requirement and the Orchestrator can create and execute multiple dependent tasks.

### Acceptance Criteria

- The Orchestrator creates structured tasks.
- Tasks can depend on other tasks.
- Ready tasks can be scheduled.
- Agents receive the correct task context.
- Execution results are stored.
- Failed tasks can be retried.
- The system prevents conflicting executions in the same workspace.

---

## Phase 3 — Agent Management &amp; Skills

### Goal

Make agents configurable and reusable.

### Scope

- Create Agent Profiles.
- Configure agent instructions.
- Assign skills.
- Configure tools.
- Configure provider preferences.
- Configure model preferences.
- Configure permissions.
- Create and terminate Agent Instances.
- Track instance lifecycle.
- Support multiple instances of the same profile.

### Initial Agent Profiles


| Agent Profile   | Responsibility                    |
| --------------- | --------------------------------- |
| Orchestrator    | Plan and coordinate tasks         |
| Research Agent  | Research and analyze requirements |
| Architect Agent | Design technical solutions        |
| Frontend Agent  | Implement frontend                |
| Backend Agent   | Implement backend                 |
| QA Agent        | Test and verify                   |
| Reviewer Agent  | Review implementation             |


These profiles should be configurable rather than permanently hard-coded into the runtime.

### Expected Result

The Orchestrator can select an appropriate agent profile based on task requirements.

---

## Phase 4 — Multi-provider Support

### Goal

Allow different agent profiles to use different AI providers.

### Scope

- Define provider adapter contracts.
- Add a second provider adapter.
- Normalize execution requests.
- Normalize execution results.
- Handle provider-specific capabilities.
- Handle authentication checks.
- Handle process lifecycle differences.
- Add provider selection rules.
- Support fallback where appropriate.

### Example Configuration


| Role            | Provider    |
| --------------- | ----------- |
| Orchestrator    | Codex       |
| Research Agent  | Claude Code |
| Architect Agent | Claude Code |
| Frontend Agent  | Codex       |
| Backend Agent   | Codex       |
| Reviewer Agent  | Claude Code |


This configuration is only an example. Users should be able to change it.

### Expected Result

The factory can delegate tasks across multiple providers without changing the core Orchestrator.

---

## Phase 5 — Agent Office &amp; Command Center

### Goal

Build the visual representation of the actual factory.

### Agent Office

Potential features:

- Isometric or pixel-art office.
- Agent characters.
- Workstations.
- Different rooms.
- Agent movement.
- Agent activity animations.
- Status indicators.
- Agent detail panel.
- Current task information.

### Command Center

Potential widgets:

- Active projects.
- Running agents.
- Pending tasks.
- Completed tasks.
- Failed tasks.
- Recent events.
- Execution statistics.

### Important Rule

Every visual state must be derived from actual runtime state and events.

For example:

- Agent starts execution → Agent Office shows Working.
- Agent requests user input → Agent Office shows Waiting.
- Task is completed → Agent Office shows the completed state.

Do not implement fake activity states just to make the office look busy.

### Expected Result

The user can visually understand what the factory is doing without inspecting raw terminal output.

---

## Phase 6 — Observability

### Goal

Make executions transparent and debuggable.

### Scope

- Execution logs.
- Task event history.
- Agent lifecycle history.
- Error tracking.
- Retry history.
- Execution duration.
- Provider information.
- Model information.
- Token usage where available.
- Cost tracking where available.
- Artifact tracking.

### Expected Result

The user can investigate why an execution succeeded, failed or became blocked.

Usage and cost reporting must account for provider differences. If a provider does not expose reliable data, the UI should show unavailable or estimated values rather than inventing precise numbers.

---

## Phase 7 — Workflow &amp; Integrations

### Goal

Make the factory more flexible.

### Scope

- Reusable workflows.
- Workflow templates.
- Configurable approval steps.
- Custom tools.
- MCP integration management.
- Additional skills.
- Git integration.
- Optional browser automation.
- Workflow execution history.

### Expected Result

Users can define reusable development processes instead of relying on one fixed workflow.

---

## Phase 8 — Open-source Preparation

### Goal

Prepare the local-first product for public release.

### Scope

- Installation documentation.
- Windows setup instructions.
- Provider integration guides.
- Configuration examples.
- Environment variable documentation.
- Security documentation.
- Contribution guidelines.
- Issue and pull request templates.
- License selection.
- Versioning and release process.
- Plugin and adapter development guidelines.

### Future Considerations

- macOS and Linux support.
- Remote execution workers.
- Multi-user collaboration.
- Team workspaces.
- Cloud deployment.
- Community agent profiles and skills.
- Extension marketplace.

These should only be added when there is a clear use case and sufficient architectural maturity.

---

## 3. MVP Definition

The first MVP should focus on a small but complete workflow.

### MVP Scenario

User request:

"Build a simple landing page in my existing local project."

The system should:

1. Accept the request.
2. Analyze the project.
3. Generate a development plan.
4. Create several tasks.
5. Assign tasks to agent profiles.
6. Execute the tasks using a real local provider.
7. Collect execution results.
8. Run basic verification.
9. Present the final result.

### MVP Requirements


| Requirement              | Must have |
| ------------------------ | --------- |
| Local application        | Yes       |
| Project selection        | Yes       |
| One provider integration | Yes       |
| Agent Runtime            | Yes       |
| Orchestrator             | Yes       |
| Task Management          | Yes       |
| Agent Profiles           | Yes       |
| Execution logs           | Yes       |
| Task status              | Yes       |
| Agent Office             | No        |
| Multi-provider           | No        |
| Workflow editor          | No        |
| Remote execution         | No        |
| Multi-user               | No        |


The MVP should demonstrate real coordinated execution, not just a dashboard with simulated agents.

## 4. Development Priorities

When deciding what to build next, prioritize:

1. Execution reliability.
2. Correct task state management.
3. Safe workspace isolation.
4. Agent coordination.
5. Provider independence.
6. Observability.
7. User experience and visualization.
8. Extensibility.

## 5. Milestone Definition

A milestone should be considered complete only when its acceptance criteria can be demonstrated.

Avoid defining milestones solely by the number of screens or components implemented.

For example:

- "Agent Runtime completed" means a real provider process can be controlled and monitored.
- "Orchestration completed" means tasks can be planned, delegated and coordinated.
- "Agent Office completed" means displayed agent states reflect real runtime activity.

## 6. Long-term Direction

The long-term objective is to evolve AI Software Factory from a personal development tool into an open-source AI engineering platform.

The architecture should make it possible for developers to:

- Use their own provider accounts.
- Add new provider adapters.
- Define custom agents.
- Share reusable skills.
- Connect custom tools and MCP servers.
- Create reusable workflows.
- Run the factory locally.
- Extend execution to remote workers in the future.

The project should remain modular, provider-agnostic and community-extensible.