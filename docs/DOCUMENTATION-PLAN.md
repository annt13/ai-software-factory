# AI Software Factory — Documentation Plan

> **Project:** AI Software Factory  
> **Document:** Documentation Planning &amp; Preparation  
> **Version:** 1.0  
> **Status:** Draft  
> **Target:** Local-first MVP → Open-source

---

# 1. Purpose

Tài liệu này xác định toàn bộ documentation cần chuẩn bị trong quá trình xây dựng AI Software Factory.

Mục tiêu:

- Xác định rõ những tài liệu cần có trước khi bắt đầu coding.
- Chuẩn hóa cách thiết kế kiến trúc và các thành phần của hệ thống.
- Làm cơ sở để AI Coding Agents hiểu và triển khai hệ thống nhất quán.
- Hạn chế việc thay đổi kiến trúc lớn trong quá trình phát triển.
- Chuẩn bị nền tảng để phát triển dự án thành một Open-source Software trong tương lai.

## 1.1. Documentation Strategy

Chia documentation thành 3 giai đoạn:


| Giai đoạn                         | Mục tiêu                                                   |
| --------------------------------- | ---------------------------------------------------------- |
| Phase 1 — Pre-development         | Chuẩn bị tài liệu trước khi bắt đầu coding                 |
| Phase 2 — During Development      | Hoàn thiện và cập nhật tài liệu trong quá trình triển khai |
| Phase 3 — Open-source Preparation | Chuẩn bị tài liệu để public và hỗ trợ cộng đồng            |


**Nguyên tắc:** Không cần hoàn thiện tất cả tài liệu trước khi code. Chỉ cần hoàn thiện những tài liệu ảnh hưởng trực tiếp đến kiến trúc, phạm vi MVP và các quyết định kỹ thuật quan trọng.

---

# 2. Documentation Inventory

## 2.1. Product &amp; Requirements

Nhóm tài liệu xác định sản phẩm cần giải quyết vấn đề gì, phục vụ đối tượng nào và cung cấp những chức năng nào.


| STT | Document                            | Nội dung                                                   | Priority    |
| --- | ----------------------------------- | ---------------------------------------------------------- | ----------- |
| 01  | Product Overview                    | Tổng quan sản phẩm, mục tiêu, định hướng phát triển        | Required    |
| 02  | Product Requirements Document (PRD) | Đối tượng sử dụng, mục tiêu, tính năng và phạm vi sản phẩm | Required    |
| 03  | MVP Scope                           | Xác định chức năng có và không có trong phiên bản MVP      | Required    |
| 04  | User Stories &amp; Use Cases        | Các tình huống sử dụng và luồng thao tác thực tế           | Required    |
| 05  | Functional Requirements             | Chi tiết hành vi và yêu cầu của từng chức năng             | Required    |
| 06  | Non-functional Requirements         | Hiệu năng, bảo mật, độ ổn định và khả năng mở rộng         | Recommended |


### Example: Use Case

**UC-001: Execute Software Development Task**

```
Actor: User

Preconditions:
  - Project has been configured.
  - Provider CLI has been installed.
  - Provider authentication is valid.

Main Flow:
  1. User submits a development request.
  2. Orchestrator analyzes the request.
  3. Orchestrator generates an execution plan.
  4. User reviews and approves the plan.
  5. Task Engine creates tasks.
  6. Scheduler assigns tasks to agents.
  7. Agents execute assigned tasks.
  8. Runtime collects execution results.
  9. System validates the results.
  10. User receives the final result.

Alternative Flows:
  - Agent execution fails.
  - Provider becomes unavailable.
  - User rejects the execution plan.
  - Task is blocked by dependencies.
  - User requests execution cancellation.
  - Validation fails and requires rework.
```

## 2.2. System &amp; Technical Architecture

Nhóm tài liệu xác định cấu trúc hệ thống, các module, dữ liệu và cách những thành phần giao tiếp với nhau.


| STT | Document                            | Nội dung                                            | Priority    |
| --- | ----------------------------------- | --------------------------------------------------- | ----------- |
| 07  | Technology Stack                    | Ngôn ngữ, framework, database và thư viện           | Required    |
| 08  | High-level Architecture             | Sơ đồ tổng thể và ranh giới hệ thống                | Required    |
| 09  | Component Architecture              | Trách nhiệm, interface và giao tiếp giữa các module | Required    |
| 10  | Domain Model                        | Entity, thuộc tính và mối quan hệ                   | Required    |
| 11  | Database Design                     | Bảng, cột, khóa, quan hệ và index                   | Required    |
| 12  | API Specification                   | API giữa frontend và backend                        | Recommended |
| 13  | Event Architecture                  | Event, payload, event lifecycle và event handling   | Required    |
| 14  | State Machine Specification         | Trạng thái và quy tắc chuyển trạng thái             | Required    |
| 15  | Architecture Decision Records (ADR) | Ghi lại các quyết định kiến trúc và lý do lựa chọn  | Recommended |


### Core Domain Entities

Các entity cần được xem xét trong quá trình thiết kế Domain Model:


| Entity           | Responsibility                             |
| ---------------- | ------------------------------------------ |
| Project          | Quản lý dự án phần mềm                     |
| Task             | Đơn vị công việc cần thực hiện             |
| TaskDependency   | Quan hệ phụ thuộc giữa các task            |
| AgentProfile     | Định nghĩa vai trò và cấu hình agent       |
| AgentInstance    | Phiên agent đang hoặc đã được khởi chạy    |
| Provider         | Nhà cung cấp hoặc công cụ AI               |
| Model            | Mô hình AI được sử dụng                    |
| Execution        | Một lần thực thi workflow hoặc task        |
| ExecutionAttempt | Một lần thử thực thi cụ thể                |
| Skill            | Khả năng hoặc hướng dẫn chuyên biệt        |
| Tool             | Công cụ agent được phép sử dụng            |
| Workspace        | Môi trường làm việc của project hoặc agent |
| Artifact         | File và kết quả được tạo ra                |
| Event            | Sự kiện phát sinh trong hệ thống           |
| ApprovalRequest  | Yêu cầu phê duyệt từ người dùng            |


**Important:** Cần phân biệt rõ:

- AgentProfile: Định nghĩa agent.
- AgentInstance: Một instance cụ thể của agent.
- Task: Công việc cần thực hiện.
- Execution: Quá trình thực thi công việc.

Không nên gộp các khái niệm này thành một entity duy nhất.

## 2.3. Agent System &amp; Runtime

Đây là nhóm tài liệu đặc thù và có vai trò quan trọng trong AI Software Factory.


| STT | Document                         | Nội dung                                                  | Priority    |
| --- | -------------------------------- | --------------------------------------------------------- | ----------- |
| 16  | Agent Specification              | Agent role, instruction, capability và permission         | Required    |
| 17  | Orchestrator Specification       | Phân tích yêu cầu, lập kế hoạch và điều phối agent        | Required    |
| 18  | Provider Adapter Specification   | Contract tích hợp Codex CLI, Claude Code và provider khác | Required    |
| 19  | Agent Execution Lifecycle        | Khởi tạo, chạy, chờ, tiếp tục, dừng và kết thúc agent     | Required    |
| 20  | Context Management               | Truyền context, kết quả và artifact giữa các agent        | Required    |
| 21  | Workspace &amp; Git Strategy     | Workspace isolation, branch, worktree và merge strategy   | Required    |
| 22  | Skills &amp; Tools Specification | Khai báo, nạp và phân quyền skills/tools                  | Recommended |
| 23  | Failure &amp; Recovery Strategy  | Retry, timeout, fallback, cancellation và recovery        | Required    |


### Provider Adapter Specification — Key Questions

Tài liệu cần trả lời được các vấn đề:

1. Làm sao phát hiện Codex CLI đã được cài đặt?
2. Làm sao kiểm tra trạng thái authentication?
3. Làm sao khởi chạy CLI process?
4. Làm sao gửi prompt và context?
5. Làm sao đọc output?
6. Làm sao xác định agent đang chạy, hoàn thành hay chờ input?
7. Làm sao gửi yêu cầu dừng process?
8. Làm sao xử lý process bị crash?
9. Làm sao quản lý timeout và retry?
10. Codex CLI và Claude Code có những khả năng khác nhau nào?
11. Làm sao xử lý các capability không được hỗ trợ đồng nhất giữa các provider?

**Design principle:** Không giả định tất cả provider có cùng khả năng hoặc cùng execution lifecycle. Adapter phải phản ánh đúng đặc điểm của từng provider.

## 2.4. UI/UX &amp; Agent Visualization

Nhóm tài liệu phục vụ thiết kế giao diện quản lý và theo dõi hoạt động của các agent.


| STT | Document                   | Nội dung                                       | Priority                     |
| --- | -------------------------- | ---------------------------------------------- | ---------------------------- |
| 24  | Information Architecture   | Cấu trúc menu và phân cấp màn hình             | Recommended                  |
| 25  | User Flow                  | Luồng thao tác của người dùng                  | Recommended                  |
| 26  | Wireframes                 | Bố cục các màn hình chính                      | Recommended                  |
| 27  | UI Design System           | Design tokens, components và themes            | Later                        |
| 28  | Agent Office Specification | Không gian, phòng, nhân vật và animation       | Later                        |
| 29  | Runtime-to-Visual Mapping  | Mapping runtime state sang trạng thái hiển thị | Required before Agent Office |


### Proposed Main Screens

1. Command Center
2. Project List / Project Detail
3. Task Board
4. Task Detail
5. Agent Management
6. Agent Detail
7. Agent Office
8. Execution Detail / Logs
9. Provider Settings
10. Skills &amp; Tools

**Important:** Agent Office là lớp visualization của runtime, không phải nơi quyết định trạng thái thực thi. Runtime phải là nguồn dữ liệu có thẩm quyền.

## 2.5. Quality, Security &amp; Operations

Nhóm tài liệu xác định tiêu chuẩn chất lượng, bảo mật và vận hành hệ thống.


| STT | Document                    | Nội dung                                          | Priority    |
| --- | --------------------------- | ------------------------------------------------- | ----------- |
| 30  | Testing Strategy            | Unit, integration, E2E và runtime testing         | Required    |
| 31  | Security Specification      | Permission, sandbox, secrets và command execution | Required    |
| 32  | Logging &amp; Observability | Logs, events, traces và metrics                   | Recommended |
| 33  | Local Deployment Guide      | Hướng dẫn cài đặt và chạy trên Windows            | During MVP  |
| 34  | Configuration Guide         | Cấu hình provider, model và workspace             | During MVP  |
| 35  | Coding Conventions          | Quy tắc code, naming và project structure         | During MVP  |


## 2.6. Open-source Preparation

Nhóm tài liệu chuẩn bị cho việc public source code và hỗ trợ cộng đồng.


| STT | Document                           | Nội dung                                   | Priority      |
| --- | ---------------------------------- | ------------------------------------------ | ------------- |
| 36  | Contribution Guide                 | Quy trình đóng góp source code             | Before public |
| 37  | Plugin / Adapter Development Guide | Hướng dẫn phát triển adapter và plugin     | Before public |
| 38  | License &amp; Open-source Policy   | License, dependency và contribution policy | Before public |
| 39  | Release Strategy                   | Versioning, changelog và release process   | Before public |
| 40  | Community Guidelines               | Quy tắc ứng xử và tương tác cộng đồng      | Before public |


---

# 3. Pre-development Documentation — 12 Priority Documents

Đây là bộ tài liệu cần hoàn thiện trước khi bắt đầu triển khai MVP.

Không cần hoàn thiện toàn bộ 40 tài liệu ở trên. Chỉ cần tập trung vào 12 tài liệu dưới đây.

## 3.1. Priority Checklist


| STT | Document                               | Expected Outcome                                | Status      |
| --- | -------------------------------------- | ----------------------------------------------- | ----------- |
| 01  | Product Requirements Document          | Hiểu rõ sản phẩm và mục tiêu                    | Not Started |
| 02  | MVP Scope                              | Phạm vi MVP được xác định                       | Not Started |
| 03  | Use Cases &amp; User Stories           | Các luồng sử dụng được xác định                 | Not Started |
| 04  | Technology Stack                       | Công nghệ được lựa chọn                         | Not Started |
| 05  | Component Architecture                 | Module và trách nhiệm được xác định             | Not Started |
| 06  | Domain Model                           | Entity và quan hệ được thiết kế                 | Not Started |
| 07  | Database Design                        | Cấu trúc dữ liệu được xác định                  | Not Started |
| 08  | State Machine &amp; Event Architecture | State và event lifecycle được thiết kế          | Not Started |
| 09  | Agent &amp; Orchestrator Specification | Cách tổ chức và điều phối agent được xác định   | Not Started |
| 10  | Provider Adapter Specification         | Chuẩn tích hợp Codex CLI đầu tiên được xác định | Not Started |
| 11  | Workspace &amp; Security Strategy      | Cách cô lập và kiểm soát thực thi được xác định | Not Started |
| 12  | Testing Strategy                       | Chiến lược kiểm thử được xác định               | Not Started |


## 3.2. Detailed Document Requirements

### DOC-01: Product Requirements Document

**Objective:** Xác định AI Software Factory là gì và hệ thống cần giải quyết vấn đề nào.

Nội dung:

- Product Vision.
- Problem Statement.
- Target Users.
- Product Goals.
- Core Features.
- Product Principles.
- High-level User Journey.
- Success Criteria.
- Out of Scope.

Expected outcome: Một tài liệu mô tả thống nhất về sản phẩm.

### DOC-02: MVP Scope

**Objective:** Giới hạn phạm vi triển khai phiên bản đầu tiên.

Nội dung:

- MVP Goals.
- In-scope Features.
- Out-of-scope Features.
- MVP Constraints.
- Technical Limitations.
- Acceptance Criteria.
- Future Roadmap.

Expected outcome: Danh sách chức năng cụ thể để tránh phát triển quá phạm vi.

### DOC-03: Use Cases &amp; User Stories

**Objective:** Xác định những tình huống thực tế mà người dùng cần thực hiện.

Các use case quan trọng:

- Create Project.
- Configure Provider.
- Create Development Task.
- Analyze User Request.
- Generate Execution Plan.
- Approve Execution Plan.
- Assign Task to Agent.
- Execute Task.
- Monitor Agent Execution.
- Review Execution Result.
- Retry Failed Task.
- Cancel Execution.
- Review Generated Artifacts.

Expected outcome: Các luồng sử dụng và acceptance criteria rõ ràng.

### DOC-04: Technology Stack

**Objective:** Chốt các công nghệ sử dụng trong MVP.

Nội dung:

- Frontend Framework.
- Backend Framework.
- Programming Language.
- Database.
- ORM / Data Access.
- Runtime / Process Management.
- CLI Integration.
- State Management.
- UI Component Library.
- Testing Framework.
- Development Environment.
- Build &amp; Packaging Strategy.

Expected outcome: Technology Stack được lựa chọn và có lý do cụ thể.

### DOC-05: Component Architecture

**Objective:** Phân chia hệ thống thành các module có trách nhiệm rõ ràng.

Các component dự kiến:


| Component         | Responsibility                    |
| ----------------- | --------------------------------- |
| Frontend          | Giao diện người dùng              |
| API Layer         | Tiếp nhận request và trả response |
| Project Manager   | Quản lý project                   |
| Task Engine       | Quản lý task và dependency        |
| Orchestrator      | Phân tích và điều phối workflow   |
| Agent Manager     | Quản lý agent                     |
| Provider Manager  | Quản lý provider và adapter       |
| Execution Runtime | Quản lý tiến trình thực thi       |
| Scheduler         | Lập lịch và phân công task        |
| Workspace Manager | Quản lý workspace và isolation    |
| Event Manager     | Phát sinh và phân phối event      |
| Artifact Manager  | Quản lý file và kết quả           |
| Approval Manager  | Quản lý approval                  |


Expected outcome: Component diagram, module responsibilities và communication contracts.

### DOC-06: Domain Model

**Objective:** Xác định các entity cốt lõi và quan hệ giữa chúng.

Nội dung:

- Core Entities.
- Entity Attributes.
- Entity Relationships.
- Aggregate Boundaries.
- Entity Lifecycle.
- Business Rules.
- Naming Conventions.

Expected outcome: Domain Model diagram và entity definitions.

### DOC-07: Database Design

**Objective:** Chuyển Domain Model thành cấu trúc lưu trữ.

Nội dung:

- Database Selection.
- Table Definitions.
- Primary Keys.
- Foreign Keys.
- Index Strategy.
- Constraints.
- JSON / Metadata Storage.
- Transaction Strategy.
- Data Retention.
- Migration Strategy.

Expected outcome: ERD và database schema.

### DOC-08: State Machine &amp; Event Architecture

**Objective:** Chuẩn hóa cách quản lý trạng thái và sự kiện trong hệ thống.

Nội dung:

**State Machine:**

- Project State.
- Task State.
- Agent Instance State.
- Execution State.
- Execution Attempt State.
- Approval State.

**Event Architecture:**

- Event Naming Convention.
- Event Schema.
- Event Payload.
- Event Producer.
- Event Consumer.
- Event Ordering.
- Event Persistence.
- Error Handling.

Expected outcome: State transition diagrams và event catalog.

### DOC-09: Agent &amp; Orchestrator Specification

**Objective:** Xác định cách định nghĩa agent và cơ chế điều phối nhiều agent.

Nội dung:

**Agent:**

- Agent Profile.
- Agent Role.
- System Instructions.
- Capabilities.
- Permissions.
- Tools.
- Skills.
- Input / Output Contract.
- Execution Lifecycle.

**Orchestrator:**

- Request Analysis.
- Requirement Decomposition.
- Plan Generation.
- Task Creation.
- Dependency Resolution.
- Agent Assignment.
- Execution Coordination.
- Result Validation.
- Failure Handling.
- Human Approval.

Expected outcome: Agent architecture và orchestration workflow.

### DOC-10: Provider Adapter Specification

**Objective:** Xây dựng cơ chế tích hợp nhiều AI coding providers.

MVP ưu tiên Codex CLI làm provider đầu tiên.

Nội dung:

- Provider Interface.
- Provider Capability Model.
- CLI Detection.
- Authentication Detection.
- Process Initialization.
- Prompt Delivery.
- Output Collection.
- Session Management.
- Status Detection.
- Input / Approval Handling.
- Cancellation.
- Timeout.
- Crash Recovery.
- Error Mapping.

Expected outcome: Adapter contract và Codex CLI integration specification.

### DOC-11: Workspace &amp; Security Strategy

**Objective:** Đảm bảo agent có môi trường thực thi riêng và được kiểm soát.

Nội dung:

**Workspace:**

- Project Workspace.
- Agent Workspace.
- Workspace Isolation.
- Git Branch Strategy.
- Git Worktree Strategy.
- File Access.
- Artifact Storage.
- Merge Strategy.
- Cleanup Strategy.

**Security:**

- Permission Model.
- Command Execution Policy.
- Shell Access.
- Filesystem Access.
- Secret Management.
- Environment Variables.
- Process Isolation.
- Resource Limits.
- Approval Requirements.
- Audit Logging.

Expected outcome: Workspace architecture và security rules.

### DOC-12: Testing Strategy

**Objective:** Xác định cách kiểm thử hệ thống trước và trong quá trình phát triển.

Nội dung:

- Unit Testing.
- Integration Testing.
- API Testing.
- E2E Testing.
- Provider Adapter Testing.
- Process Lifecycle Testing.
- Orchestrator Testing.
- State Transition Testing.
- Failure &amp; Recovery Testing.
- Security Testing.
- Test Fixtures.
- Acceptance Criteria.

Expected outcome: Test strategy, test coverage requirements và testing workflow.

---

# 4. Documentation Development Lifecycle

Documentation sẽ được hoàn thiện theo từng giai đoạn.

## Phase 1 — Before Coding

**Goal:** Hoàn thiện nền tảng để có thể bắt đầu triển khai MVP.

Documents:

- Product Requirements Document.
- MVP Scope.
- Use Cases &amp; User Stories.
- Technology Stack.
- High-level Architecture.
- Component Architecture.
- Domain Model.
- Database Design.
- State Machine &amp; Event Architecture.
- Agent &amp; Orchestrator Specification.
- Provider Adapter Specification.
- Workspace &amp; Security Strategy.
- Testing Strategy.

Exit Criteria:

- Product scope được thống nhất.
- Kiến trúc tổng thể được xác định.
- Core domain được thiết kế.
- Provider integration strategy được xác định.
- Execution lifecycle được xác định.
- Security boundaries được xác định.
- Có thể chia công việc thành các implementation tasks.

## Phase 2 — During Development

**Goal:** Hoàn thiện tài liệu chi tiết dựa trên implementation thực tế.

Documents:

- API Specification.
- Detailed Component Design.
- Logging &amp; Observability.
- Configuration Guide.
- Local Deployment Guide.
- Coding Conventions.
- Runtime-to-Visual Mapping.
- UI/UX Wireframes.
- Architecture Decision Records.
- Failure &amp; Recovery Details.

Exit Criteria:

- Các module có interface rõ ràng.
- API và event contract được cập nhật.
- Có hướng dẫn chạy local.
- Có hướng dẫn cấu hình provider.
- Testing và observability được tích hợp.
- Documentation phản ánh đúng implementation.

## Phase 3 — Open-source Preparation

**Goal:** Chuẩn bị để người khác có thể cài đặt, sử dụng và đóng góp cho dự án.

Documents:

- [README.md](http://README.md).
- Installation Guide.
- Quick Start Guide.
- Configuration Guide.
- Provider Integration Guide.
- Plugin / Adapter Development Guide.
- Contribution Guide.
- Code of Conduct.
- License.
- Security Policy.
- Changelog.
- Release Strategy.
- Troubleshooting Guide.

Exit Criteria:

- Người dùng mới có thể cài đặt và chạy dự án.
- Có hướng dẫn cấu hình provider.
- Có hướng dẫn phát triển adapter.
- Có license rõ ràng.
- Có contribution workflow.
- Không chứa secrets hoặc thông tin cá nhân.
- Có quy trình phát hành version.

---

# 5. Recommended Documentation Folder Structure

```
ai-software-factory/
│
├── README.md
│
├── docs/
│   │
│   ├── 01-product/
│   │   ├── product-overview.md
│   │   ├── product-requirements.md
│   │   ├── mvp-scope.md
│   │   ├── user-stories.md
│   │   └── functional-requirements.md
│   │
│   ├── 02-architecture/
│   │   ├── system-architecture.md
│   │   ├── component-architecture.md
│   │   ├── technology-stack.md
│   │   ├── domain-model.md
│   │   ├── database-design.md
│   │   ├── api-specification.md
│   │   ├── event-architecture.md
│   │   ├── state-machine.md
│   │   └── adr/
│   │       ├── README.md
│   │       └── ADR-0001-template.md
│   │
│   ├── 03-agent-system/
│   │   ├── agent-specification.md
│   │   ├── orchestrator-specification.md
│   │   ├── provider-adapter.md
│   │   ├── execution-lifecycle.md
│   │   ├── context-management.md
│   │   ├── workspace-strategy.md
│   │   ├── skills-and-tools.md
│   │   └── failure-recovery.md
│   │
│   ├── 04-ui-ux/
│   │   ├── information-architecture.md
│   │   ├── user-flows.md
│   │   ├── wireframes.md
│   │   ├── design-system.md
│   │   ├── agent-office.md
│   │   └── runtime-visual-mapping.md
│   │
│   ├── 05-quality-security/
│   │   ├── testing-strategy.md
│   │   ├── security-specification.md
│   │   └── observability.md
│   │
│   ├── 06-development/
│   │   ├── development-roadmap.md
│   │   ├── coding-conventions.md
│   │   ├── local-setup.md
│   │   ├── configuration.md
│   │   └── troubleshooting.md
│   │
│   └── 07-open-source/
│       ├── contribution-guide.md
│       ├── adapter-development.md
│       ├── code-of-conduct.md
│       ├── security-policy.md
│       ├── license-policy.md
│       └── release-strategy.md
│
└── ...
```

---

# 6. Documentation Standards

Để documentation có thể được sử dụng trực tiếp bởi AI Coding Agents, cần thống nhất một số nguyên tắc.

## 6.1. General Rules

1. Mỗi tài liệu phải có mục đích rõ ràng.
2. Mỗi tài liệu phải xác định phạm vi và nội dung liên quan.
3. Không lặp lại cùng một nội dung ở nhiều tài liệu.
4. Các quyết định kiến trúc quan trọng phải có lý do.
5. Khi implementation thay đổi, tài liệu liên quan phải được cập nhật.
6. Không ghi nhận một quyết định chưa được phê duyệt như một quyết định chính thức.
7. Phân biệt rõ Proposed, Accepted và Deprecated.

## 6.2. Document Metadata Template

Mỗi tài liệu nên có metadata:

```
document_id: DOC-001
title: Product Requirements Document
version: 1.0
status: draft
created_at: YYYY-MM-DD
updated_at: YYYY-MM-DD
owner: Project Team
related_documents:
  - DOC-002
  - DOC-003
```

## 6.3. Architecture Decision Records

Các quyết định quan trọng cần được ghi lại bằng ADR.

Ví dụ:

- ADR-0001: Database Selection.
- ADR-0002: Provider Adapter Architecture.
- ADR-0003: Workspace Isolation Strategy.
- ADR-0004: Agent Execution Model.
- ADR-0005: Event Architecture.
- ADR-0006: Local Runtime Strategy.

Mỗi ADR nên có:

- Context.
- Problem.
- Decision.
- Alternatives Considered.
- Consequences.
- Status.

---

# 7. Documentation Progress Tracking

## 7.1. Status Definitions


| Status      | Meaning                         |
| ----------- | ------------------------------- |
| Not Started | Chưa bắt đầu                    |
| Drafting    | Đang soạn thảo                  |
| In Review   | Đang review                     |
| Approved    | Đã được phê duyệt               |
| Implemented | Đã được triển khai và đối chiếu |
| Deprecated  | Không còn được sử dụng          |


## 7.2. Pre-development Completion Criteria

Bộ 12 tài liệu ưu tiên chỉ được xem là hoàn thành khi:

- Nội dung chính đã được xác định.
- Không còn các quyết định quan trọng ở trạng thái chưa rõ.
- Các tài liệu liên quan không mâu thuẫn.
- Các diagram cần thiết đã được bổ sung.
- Có thể chuyển yêu cầu thành implementation tasks.
- Có acceptance criteria cho các chức năng MVP.
- Các rủi ro kỹ thuật chính đã được nhận diện.

---

# 8. Recommended Preparation Order

Không nhất thiết phải viết tài liệu theo số thứ tự. Nên triển khai theo dependency giữa các tài liệu.


| Step | Document                               | Reason                         |
| ---- | -------------------------------------- | ------------------------------ |
| 01   | Product Requirements                   | Xác định sản phẩm              |
| 02   | MVP Scope                              | Giới hạn phạm vi               |
| 03   | Use Cases &amp; User Stories           | Xác định luồng sử dụng         |
| 04   | Technology Stack                       | Chọn công nghệ                 |
| 05   | High-level Architecture                | Thiết kế tổng thể              |
| 06   | Component Architecture                 | Chia module                    |
| 07   | Domain Model                           | Thiết kế domain                |
| 08   | State Machine &amp; Event Architecture | Xác định lifecycle             |
| 09   | Agent &amp; Orchestrator Specification | Thiết kế agent system          |
| 10   | Provider Adapter Specification         | Thiết kế provider integration  |
| 11   | Database Design                        | Thiết kế persistence           |
| 12   | Workspace &amp; Security Strategy      | Thiết kế execution environment |
| 13   | Testing Strategy                       | Xác định cách kiểm thử         |


Lưu ý: Đây là thứ tự đề xuất để chuẩn bị. Database Design có thể được thực hiện song song với Domain Model và State Machine sau khi các khái niệm cốt lõi đã rõ.

---

# 9. Final Recommendation

Đối với AI Software Factory, documentation không chỉ phục vụ developer mà còn là nguồn context và specification cho các AI Agents tham gia phát triển dự án.

Vì vậy, cần phân biệt:

**1. Product Documentation**

Giúp xác định sản phẩm cần xây dựng.

**2. Architecture Documentation**

Giúp xác định hệ thống được tổ chức và hoạt động như thế nào.

**3. Implementation Documentation**

Giúp AI Coding Agents biết phải triển khai những gì và tuân theo contract nào.

**4. Operational Documentation**

Giúp người dùng cài đặt, cấu hình và vận hành hệ thống.

**5. Community Documentation**

Giúp cộng đồng hiểu, sử dụng và mở rộng dự án.

## Core Principle

> Documentation First, Implementation Incrementally.

Không cần thiết kế hoàn chỉnh toàn bộ hệ thống trước khi coding. Tuy nhiên, các quyết định ảnh hưởng đến domain, runtime, provider integration, workspace isolation và security cần được xác định đủ rõ trước khi triển khai.

Mục tiêu cuối cùng là xây dựng một AI Software Factory có kiến trúc rõ ràng, có thể mở rộng provider, agent và workflow mà không phải thiết kế lại toàn bộ hệ thống.