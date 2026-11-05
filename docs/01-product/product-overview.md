# Product Overview

## 1. Product Definition

AI Software Factory is a local-first, multi-agent software engineering platform.

It provides a unified environment where a user can submit a software requirement and delegate the execution process to a coordinated team of AI agents.

The platform combines:

- AI agent orchestration.
- Software development task management.
- Provider and model integrations.
- Agent execution management.
- Skills and tool management.
- Project workspace management.
- Execution monitoring and visualization.

The system is designed to operate as a personal development factory first, with a future path toward an extensible open-source product.

## 2. Core User Experience

### 2.1 Submit a requirement

The user selects a project and provides a high-level request.

Example:

"Build a simple task management web application with authentication, project management and a dashboard."

### 2.2 Analyze and plan

The Orchestrator:

1. Understands the requirement.
2. Identifies missing information.
3. Analyzes the project context.
4. Determines the necessary development phases.
5. Breaks the requirement into tasks and subtasks.
6. Identifies dependencies.
7. Selects suitable agent profiles.
8. Produces an execution plan.

The user can review and approve the plan before execution.

### 2.3 Delegate and execute

The Orchestrator delegates tasks to specialized agents.

Example:


| Agent           | Responsibility                                         |
| --------------- | ------------------------------------------------------ |
| Research Agent  | Research requirements, libraries and technical options |
| Architect Agent | Design architecture and technical specifications       |
| Frontend Agent  | Implement frontend features                            |
| Backend Agent   | Implement backend features                             |
| QA Agent        | Test features and identify defects                     |
| Reviewer Agent  | Review code and implementation quality                 |


These are agent profiles, not permanently running processes. The runtime creates agent instances when execution is needed.

### 2.4 Coordinate execution

The system manages:

- Task dependencies.
- Agent assignment.
- Execution order.
- Parallel execution where safe.
- Timeouts and retries.
- Human approval requests.
- Failed task recovery.
- Agent communication and task handoff.

### 2.5 Verify and deliver

After implementation, the system can:

1. Run configured tests.
2. Collect test results.
3. Ask QA or Reviewer agents to inspect changes.
4. Create follow-up tasks for defects.
5. Request human review.
6. Present the final results and generated artifacts.

The system should not claim that a task is complete merely because an agent says it is complete. Completion should be based on defined acceptance criteria and available verification results.

## 3. Main Product Modules

### 3.1 Command Center

The main overview of the factory.

Potential information:

- Active projects.
- Running tasks.
- Waiting tasks.
- Active agent instances.
- Failed executions.
- Recent activity.
- Usage and cost summaries, when available.

### 3.2 Project Management

A project represents a software development workspace.

A project may contain:

- Project name and description.
- Local repository path.
- Git repository information.
- Technology stack.
- Project instructions.
- Environment configuration references.
- Tasks and execution history.
- Generated artifacts.

The platform should support multiple projects without mixing their contexts or workspaces.

### 3.3 Task Management

The task system is the operational backbone of the factory.

A task can contain:

- Title and description.
- Acceptance criteria.
- Parent task.
- Subtasks.
- Dependencies.
- Assigned agent profile.
- Actual agent instance.
- Priority.
- Status.
- Input and output artifacts.
- Execution attempts.
- Logs and events.

Suggested task lifecycle:

`Backlog → Planned → Ready → Running → Review → Completed`

Additional states:

- Blocked
- Waiting for Input
- Failed
- Cancelled

Task status transitions must be controlled by the runtime and task engine, not by the visual interface alone.

### 3.4 Agent Management

Agent Management defines what kinds of agents the factory can use.

An Agent Profile describes:

- Name and role.
- System instructions.
- Responsibilities.
- Skills.
- Available tools.
- Allowed permissions.
- Preferred provider.
- Preferred model.
- Execution constraints.

An Agent Instance is a running execution of an Agent Profile.

For example:

- Frontend Agent is an Agent Profile.
- Frontend Agent Instance #001 is a live process working on a specific task.

The same profile can have multiple instances when concurrency and workspace isolation allow it.

### 3.5 Agent Office

Agent Office is the visual representation of the factory.

Possible visual elements:

- Agent characters.
- Rooms or work areas.
- Workstations.
- Task boards.
- Meeting areas.
- Waiting areas.
- Review areas.

Example mapping:


| Runtime state     | Visual representation            |
| ----------------- | -------------------------------- |
| Idle              | Agent at its workstation         |
| Running           | Agent actively working           |
| Waiting for Input | Agent waiting for the user       |
| Blocked           | Agent waiting for a dependency   |
| Reviewing         | Agent reviewing work             |
| Failed            | Agent displaying an error state  |
| Completed         | Agent showing a completion state |


Important: Agent Office must subscribe to real runtime events and state. It must not maintain a separate, manually simulated agent status.

### 3.6 Workflow &amp; Orchestration

This module coordinates the software development process.

Responsibilities:

- Requirement analysis.
- Task decomposition.
- Agent selection.
- Scheduling.
- Dependency resolution.
- Parallel execution.
- Result aggregation.
- Retry and recovery.
- Human approval.

The initial version can use predefined workflows and orchestration logic. A visual workflow editor is a later feature.

### 3.7 Skills Library

Skills are reusable instructions, knowledge and procedures that agents can use.

Examples:

- Requirements analysis.
- System design.
- API design.
- Frontend development.
- Backend development.
- Code review.
- Testing.
- Documentation.

A skill is not an agent. An agent profile can use one or more skills.

### 3.8 Tools &amp; MCP

This module manages external capabilities available to agents.

Potential integrations:

- File system operations.
- Git operations.
- Terminal execution.
- Browser automation.
- Documentation search.
- Database inspection.
- MCP servers.

Tools should be granted according to each agent's role and permissions.

### 3.9 Observability

The system should provide visibility into execution.

Information includes:

- Agent lifecycle events.
- Task lifecycle events.
- Provider process output.
- Errors and stack traces.
- Execution duration.
- Retry history.
- Token usage and cost, when exposed by the provider.
- Generated artifacts.

The platform should distinguish measured data from estimated or unavailable data.

### 3.10 Settings &amp; Integrations

Potential settings:

- Provider configuration.
- Model selection.
- CLI executable paths.
- Authentication status.
- Default agent profiles.
- Concurrency limits.
- Workspace settings.
- Permission policies.
- Tool and MCP configuration.

Authentication should use official provider mechanisms. The platform should not extract, store or proxy private subscription session tokens.

## 4. Example End-to-End Scenario

User request:

"Create a landing page for a 3D digital world."

Execution:

1. User creates or selects a project.
2. User submits the requirement.
3. Orchestrator analyzes the project and request.
4. Research Agent investigates suitable libraries and implementation approaches.
5. Architect Agent prepares the technical plan.
6. Orchestrator creates tasks and dependencies.
7. Frontend Agent implements the landing page.
8. Reviewer Agent reviews the changes.
9. QA Agent runs tests and checks the result.
10. Failed checks generate follow-up tasks.
11. Orchestrator aggregates the results.
12. User reviews the final output.

The Agent Office displays the corresponding agent activity throughout execution.

## 5. Product Boundaries

The platform is an orchestration and execution management layer.

It is not intended to replace every provider's native coding agent.

Instead, it coordinates available provider tools and gives users a consistent way to manage projects, agents, tasks and workflows.

Provider capabilities may differ. The platform must expose those differences rather than pretending every provider supports identical features.

## 6. Success Criteria

The first useful version should demonstrate that:

- A user can create a project and submit a requirement.
- The Orchestrator can generate an actionable plan.
- Tasks can be assigned to specialized agent profiles.
- At least two agent instances can execute a coordinated workflow.
- Agents can work with a real local project.
- Execution results and artifacts can be inspected.
- Failures can be surfaced and retried safely.
- The UI reflects actual runtime state.
- The system can complete a small software development task end to end.