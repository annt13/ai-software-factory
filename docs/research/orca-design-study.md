```yaml
document_id: RES-001
title: Orca Design Study
version: 1.0
status: draft
created_at: 2026-11-05
updated_at: 2026-11-05
owner: Project Team
related_documents:
  - ADR-0001 First Provider — Claude Code
  - ADR-0002 Claude Code Integration via CLI Subprocess
  - DOC-05 Component Architecture
  - DOC-08 State Machine & Event Architecture
  - DOC-09 Agent & Orchestrator Specification
  - DOC-10 Provider Adapter Specification
  - DOC-11 Workspace & Security Strategy
```

# Orca Design Study

## 1. Mục đích

Tài liệu này ghi lại cách Orca thiết kế việc chạy và điều phối nhiều AI coding agent. Mục tiêu là học những gì phù hợp cho AI Software Factory và nhận diện những điểm nên tránh.

Đây là tài liệu nghiên cứu, **không phải quyết định kiến trúc**. Các đề xuất ở mục 9 chỉ có hiệu lực khi được ghi nhận bằng ADR hoặc đưa vào tài liệu thiết kế tương ứng.

### Nguồn và phương pháp

- Orca v1.4.244 cài trên máy phát triển. Chỉ dùng lệnh chỉ-đọc: `orca --help`, `--help` của từng lệnh, `orca agent-context --json`, `orca skills get orchestration|orca-cli|orca-per-workspace-env`, `orca status`, `worktree ps`, `run-list`, `gate-list`.
- Quan sát command line của tiến trình `claude.exe` do Orca khởi chạy.
- Repository công khai và tài liệu của Orca (xem mục 2).
- Tài liệu chính thức của Claude Code: CLI reference, headless, permissions, Agent SDK.

Không đọc file chứa thông tin đăng nhập, không giải nén mã nguồn ứng dụng đã cài.

Ký hiệu: **(suy luận)** đánh dấu nội dung không được nguồn nêu trực tiếp.

## 2. Orca là gì

| Mục | Thông tin |
| --- | --------- |
| Nhà phát triển | stablyai |
| Repository | https://github.com/stablyai/orca |
| License | MIT |
| Website / Docs | https://orca.dev, https://www.onorca.dev/docs |
| Mô tả | "the place for working with a fleet of coding agents (e.g. Claude Code, Codex). Run any agent with your own subscription." |
| Nền tảng | Desktop (Electron), mobile companion, remote runtime |
| Agent hỗ trợ | Khoảng 35 agent: Claude Code, Codex, Cursor, OpenCode, Antigravity, … |

Vì Orca là mã nguồn mở theo MIT, có thể đọc mã nguồn để học thêm khi cần. Tài liệu thiết kế nằm rải rác trong `docs/` của repository; không có một tài liệu kiến trúc tổng thể.

## 3. Tích hợp provider

### 3.1. Hai bề mặt: Chat và Terminal

| Bề mặt | Cách hoạt động | Dùng khi |
| ------ | -------------- | -------- |
| **Chat** (structured chat) | Chạy CLI ở chế độ headless, giao tiếp bằng JSON có cấu trúc qua stdin/stdout | Agent và host hỗ trợ |
| **Terminal** | Chạy giao diện TUI của CLI trong một pseudo-terminal, do một daemon riêng quản lý | Agent không hỗ trợ chat, hoặc người dùng chọn |

`orca agent launch` mặc định thử Chat; nếu không được thì lùi về Terminal và trả về "receipt" giải thích lý do (`outcome.kind`, `receipt.detail`).

Mọi thao tác điều phối dùng **cùng một bộ lệnh** cho cả hai bề mặt. Output được đọc qua `worker-read --source auto|transcript|terminal`.

### 3.2. Cách Orca chạy Claude Code (bề mặt Chat)

Orca gọi file `claude.exe` bằng đường dẫn cụ thể:

```
claude.exe
  --input-format stream-json --output-format stream-json --verbose
  --include-partial-messages --replay-user-messages
  --permission-prompt-tool stdio --permission-mode <mode>
  --setting-sources=user,project,local
  --resume=<session-id> | --session-id=<id>
  --model <id> --effort <level> --thinking-display <mode>
  --add-dir <path> --plugin-dir <path>
```

Đối chiếu với tài liệu chính thức của Claude Code:

| Flag | Trạng thái trong tài liệu | Ý nghĩa |
| ---- | ------------------------- | ------- |
| `--input-format stream-json` | Có tài liệu | Tiến trình sống lâu, nhận nhiều lượt user qua stdin, xử lý ngắt và yêu cầu phê duyệt. Định dạng từng dòng stdin chỉ được mô tả qua đối tượng của Agent SDK |
| `--replay-user-messages` | Có tài liệu | Phát lại message của user ra stdout để xác nhận đã nhận |
| `--include-partial-messages` | Có tài liệu | Stream từng phần nội dung (`stream_event`) |
| `--setting-sources` | Có tài liệu | Chọn nguồn cấu hình được nạp (`user`, `project`, `local`). Bỏ `project` thì **không nạp `.claude/settings.json` lẫn `.mcp.json` của project** |
| `--permission-prompt-tool stdio` | **Không có tài liệu** | Tài liệu chỉ nói về tên một MCP tool. Giá trị `stdio` là chi tiết nội bộ của Agent SDK: yêu cầu phê duyệt đi qua `control_request` / `control_response` trên stdin/stdout |
| `--thinking-display` | **Không có tài liệu** | Tương ứng với tuỳ chọn `ThinkingConfig.display` của Agent SDK |
| Ngắt một lượt chạy | Chỉ có qua Agent SDK | `interrupt()` của SDK gửi một `interrupt` control request qua stdin. Capability `interrupt_receipt_v1` được báo trong `system/init` |

**Nhận xét:** Orca thực chất đang nói chuyện với Claude Code bằng **giao thức điều khiển của Agent SDK** mà không dùng thư viện SDK. Cách này mạnh (phê duyệt, ngắt, nhiều lượt trong một tiến trình) nhưng phụ thuộc vào chi tiết chưa được công bố chính thức.

### 3.3. Khác biệt giữa các provider

- Model và effort được truyền dưới dạng mã của từng provider. Mỗi provider hỗ trợ một mức khác nhau (ví dụ OpenCode không có effort; một số agent tự đọc model từ cấu hình riêng).
- Receipt so sánh `launch.requested` với `launch.effective`, để người dùng biết cấu hình nào thực sự được áp dụng.
- Runtime công bố **capability có phiên bản** (ví dụ `orchestration.worker-launch-preferences.v1`). Trường mới chỉ được gửi khi bên kia hỗ trợ.
- Orca cài **hook quản lý** vào cấu hình của từng agent (ví dụ `settings.json` của Claude Code) để agent tự báo trạng thái, thay vì phải đọc màn hình terminal. Đọc màn hình và nhận diện "TUI idle" chỉ là phương án dự phòng.
- **Đăng nhập:** Orca không quản lý đăng nhập của Claude Code hay Codex; CLI tự lo. Riêng một số agent (OpenCode, Devin) có profile tài khoản riêng do Orca quản lý.

## 4. Domain model

| Thực thể Orca | Ý nghĩa | Tương ứng trong AI Software Factory |
| ------------- | ------- | ----------------------------------- |
| **Run** | Không gian tên bền vững và hộp thư của coordinator. Không tự lập lịch hay đặt worker | Execution (cấp workflow) |
| **Task** | Đơn vị công việc có `spec`, `deps`, `parent` | Task, TaskDependency |
| **Dispatch** | **Một lần thử thực thi Task có thẩm quyền.** Là nguồn sự thật về vòng đời | ExecutionAttempt |
| **Worker** | Tiến trình và terminal được giám sát, thuộc về một Dispatch | AgentInstance |
| **Message / Delivery** | Thông điệp có kiểu, giao theo lô FIFO có xác nhận | Event + kênh giao tiếp agent |
| **Gate** | Quyết định do coordinator sở hữu, chặn một Task | ApprovalRequest |
| **Terminal handle** | Định danh terminal trong một runtime; hết hiệu lực khi runtime khởi động lại | (Chi tiết nội bộ của Runtime) |
| **Worktree / Workspace** | Git worktree hoặc thư mục, có quan hệ cha–con | Workspace |
| **Project / Repo / Project-host setup** | Project là danh tính bền vững; setup làm project khả dụng trên một host | Project |
| **Host / Environment** | Máy local, SSH, hoặc Orca server đã ghép cặp | (Remote worker — tương lai) |
| **Automation** | Prompt chạy theo lịch | (Chưa có trong phạm vi MVP) |

Quy tắc yêu cầu mỗi Task spec phải nêu: **Target, Change, Constraints, Ownership, Observable acceptance**. Điều này khớp với yêu cầu acceptance criteria trong Product Overview §2.5.

## 5. Vòng đời và trạng thái

### 5.1. Task

Trạng thái: `pending → ready → dispatched → completed | failed`, cộng thêm `blocked`.

- Chỉ Task `ready` mới được khởi chạy worker; nếu không, lỗi `task_not_startable` kèm danh sách phụ thuộc chưa xong.
- `worker_done` hợp lệ tự động đóng Task và Dispatch.
- Huỷ được ghi là `failed` với kết quả `cancelled`. Các Task phụ thuộc vẫn bị chặn cho tới khi một lần thử lại (`--retry-of`) hoàn thành.

### 5.2. Dispatch (lần thử)

- Trạng thái: active, `failed`, `stopped`, `outcome_unknown`.
- **Circuit breaker:** sau 3 lần khởi động thất bại cho một Task, không cho thử lại nữa. Worker tự báo `failed` không bị tính vào giới hạn này.

### 5.3. Tài nguyên worker — tách khỏi trạng thái Task

`active, reclaimable, retained, release_pending, release_unknown, released`

"Một Task đã hoàn thành vẫn có thể sở hữu một terminal đang chạy." Output được lưu trữ trước khi giải phóng terminal.

### 5.4. Trạng thái sống — ba giá trị

`live | unverifiable | exited`

`unverifiable` luôn có lý do cụ thể: `missing_status`, `stale_status`, `restored_unconfirmed`, `host_unavailable`, `capability_unsupported`.

**Quy tắc an toàn cốt lõi:** *"Absence never authorizes stop, abandon, retry, or release."* Chỉ khi có bằng chứng `exited` hoặc kết quả đã được chấp nhận thì mới được dừng, bỏ, thử lại hay giải phóng.

## 6. Giao thức coordinator–worker

| Cơ chế | Cách hoạt động |
| ------ | -------------- |
| **Preamble** | Orca chèn vào prompt của worker đúng đường dẫn CLI, handle, Task ID, Dispatch ID. Worker phải sao chép nguyên văn |
| **Heartbeat** | `send --type heartbeat --phase investigating|implementing|reviewing|waiting`. Chứng minh còn sống, **không** chứng minh hoàn thành |
| **Hoàn thành** | `worker_done` gửi đúng một lần, bắt buộc có `--outcome succeeded|failed`, tóm tắt, tuỳ chọn danh sách file đã sửa |
| **Hỏi–đáp** | `ask` chặn tối đa 10–30 phút. Hết giờ thì câu hỏi vẫn được lưu; worker tiếp tục bằng `ask --resume <msgId>` thay vì hỏi lại |
| **Leo thang** | `--type escalation`, chỉ trước khi hoàn thành |
| **Điều hướng** | Coordinator gửi message tới `dispatch:<id>`; worker kiểm tra hộp thư ở các checkpoint và phải xác nhận từng lô. Lô chưa xác nhận sẽ được gửi lại |
| **Fencing** | Khi lần thử bị giao lại hoặc đã đóng, worker cũ nhận `consumer_fenced` và phải dừng. `worker-abandon` thu hồi quyền **mà không khẳng định tiến trình đã chết** |
| **Idempotency** | Mọi thao tác thay đổi trạng thái nhận `--retry-request <id>`; có thể tra trạng thái yêu cầu |
| **Lồng nhau** | Worker có thể tạo worker phụ, có giới hạn độ sâu |

Lý do tồn tại của fencing: không phải lúc nào cũng chứng minh được một tiến trình đã chết (nhất là trên máy từ xa). Vì vậy **quyền thẩm quyền được thu hồi tách biệt với việc dừng tiến trình**.

## 7. Workspace, remote và quan sát

### 7.1. Workspace

- Vị trí worker: `current | <exact> | new-child | new-top-level`. "Worker mới nghĩa là terminal agent mới, không phải git worktree mới"; worktree mới chỉ tạo khi có xung đột.
- Setup: `run | skip | inherit`; repo có thể chọn `start-immediately` hoặc `wait-for-setup` (prompt chỉ được gửi khi setup thành công).
- Các quyết định độc lập: quan hệ cha–con, nhánh git gốc, cô lập file, quan hệ điều phối, nhóm hiển thị, máy thực thi.
- Dọn dẹp: chỉ xoá nhánh khi chứng minh được nhánh đã merge hoặc được tạo cùng worktree.

### 7.2. Remote

- `orca serve` chạy runtime không giao diện; ghép cặp bằng `orca://pair?...`.
- Có thể đặt riêng một worker lên máy khác (`worker-start --on <env>`), trong khi Run và Task vẫn có thẩm quyền ở server hiện tại.
- Mất kết nối được báo là `unverifiable`, không bao giờ là `exited`, và không bao giờ tự lùi về chạy local.

### 7.3. Quan sát

- `worker-list`: tổng quan đội worker, có `attention.requiresAction` và **`nextAction` là một câu lệnh cụ thể** để chạy tiếp.
- `worktree ps`: trạng thái từng agent, prompt và message cuối, kèm dòng báo host nào chưa liên lạc được.
- `terminal wait --for exit|tui-idle`, `terminal send --wait-submit` với receipt theo từng bước (`input_accepted`, `turn_started`).
- Lỗi trả về mã ổn định kèm `nextSteps`.

## 8. So sánh kiến trúc

| Khía cạnh | Orca | AI Software Factory (thiết kế hiện tại) |
| --------- | ---- | --------------------------------------- |
| Ai điều phối | **Một agent coordinator** gọi lệnh `orca` để giao việc. Orca không có scheduler tích hợp; vòng lặp điều phối chính là agent | **Orchestrator trung tâm** trong runtime của factory |
| Tính đúng đắn của giao thức | Phụ thuộc agent tuân theo quy tắc viết bằng văn xuôi ("đúng một lần", "xác nhận trước khi…") **(suy luận: dễ bị vi phạm)** | Có thể kiểm soát bằng code và schema |
| Chuyển trạng thái | Do runtime Orca thực thi, nhưng được kích hoạt bởi lệnh mà agent gọi | Do Task Engine / Runtime Manager thực thi |
| Điều hướng worker | Kéo (pull): worker chỉ thấy message khi tự kiểm tra hộp thư | Chưa thiết kế |
| Phạm vi sản phẩm | Môi trường làm việc tổng quát cho đội agent | Quy trình phát triển phần mềm có kế hoạch, kiểm chứng và trực quan hoá |

## 9. Bài học cho AI Software Factory

### 9.1. Nên học

| # | Ý tưởng | Ảnh hưởng tới |
| - | ------- | ------------- |
| 1 | **Tách Task / ExecutionAttempt / AgentInstance.** Lần thử (không phải Task, không phải tiến trình) là nơi giữ thẩm quyền. Đúng với yêu cầu "không gộp các khái niệm" trong Documentation Plan §2.2 | DOC-06, DOC-08 |
| 2 | **Fencing bằng generation:** mỗi lần thử có số thế hệ; kết quả từ thế hệ cũ bị từ chối | DOC-08, Failure & Recovery Strategy |
| 3 | **Trạng thái sống ba giá trị có lý do**, và quy tắc "thiếu bằng chứng không cho phép hành động phá huỷ" | DOC-08, DOC-11 |
| 4 | **Tách vòng đời tài nguyên tiến trình khỏi trạng thái Task**; lưu output trước khi giải phóng | DOC-08, DOC-10 |
| 5 | **Idempotency key cho mọi thao tác thay đổi trạng thái** (API và lệnh nội bộ) | API Specification, DOC-05 |
| 6 | **Kết quả hoàn thành có cấu trúc bắt buộc:** outcome, tóm tắt, file đã sửa. Heartbeat không bao giờ được hiểu là hoàn thành | DOC-09 |
| 7 | **Receipt "requested vs effective"** và **`nextAction` cụ thể** trong lỗi và trạng thái — rất hợp với Command Center | DOC-10, UI |
| 8 | **Circuit breaker** giới hạn số lần thử lại | Failure & Recovery Strategy |
| 9 | **`--setting-sources user`** để chạy Claude Code trên repo không tin cậy mà vẫn dùng đăng nhập bằng gói (thay cho `--bare`) | ADR-0001, DOC-11 |
| 10 | **Tiến trình sống lâu với `--input-format stream-json`:** nhiều lượt trong một tiến trình, ngắt bằng control request thay vì tín hiệu — giải quyết vấn đề không có SIGINT trên Windows | ADR-0002, DOC-10 |
| 11 | **Capability có phiên bản** và lùi về phương án dự phòng có giải thích | DOC-10 |
| 12 | **Task spec có cấu trúc** (Target, Change, Constraints, Ownership, Observable acceptance) | DOC-09 |

### 9.2. Nên tránh hoặc làm khác

| Điểm yếu của Orca | Hướng của AI Software Factory |
| ----------------- | ----------------------------- |
| Giao thức phụ thuộc agent tuân thủ quy tắc văn xuôi | Giữ lập lịch và chuyển trạng thái **xác định trong code**. LLM chỉ dùng cho phân tích, lập kế hoạch, thực thi và review |
| Agent báo cáo bằng cách gọi CLI với nhiều tham số phải sao chép nguyên văn | **(Đề xuất)** Factory cung cấp một MCP server cho agent, với các tool có schema như `report_progress`, `complete_task`, `ask_question`. Cùng server đó có thể làm `--permission-prompt-tool` theo cách **có tài liệu chính thức** |
| Điều hướng kiểu pull, có thể trễ | Với tiến trình sống lâu, Runtime có thể gửi thẳng message vào stdin của agent |
| `unverifiable` có thể chờ vô thời hạn | Có chính sách timeout rõ ràng, hết hạn thì tạo ApprovalRequest cho người dùng |
| Dựa vào flag chưa công bố (`--permission-prompt-tool stdio`, `--thinking-display`) | Ưu tiên cơ chế có tài liệu; nếu buộc dùng cơ chế nội bộ thì cô lập trong adapter, kiểm tra capability và có test hồi quy theo phiên bản |
| Độ phức tạp do tương thích ngược phiên bản (`*.v1`, legacy contract) | Giữ MVP đơn giản, nhưng chuẩn bị sẵn trường version trong event schema |

## 10. Câu hỏi mở cho PRD và MVP Scope

1. **Khác biệt sản phẩm:** Orca đã có worktree theo agent, đa provider, task có phụ thuộc, gate phê duyệt và giám sát worker. AI Software Factory khác ở đâu? Ứng viên: Orchestrator trung tâm có kiểm chứng kết quả theo acceptance criteria, workflow phát triển phần mềm định sẵn, Agent Office, quan sát chi phí và trạng thái theo runtime.
2. **Xây mới hay xây trên Orca:** có nên dùng Orca (MIT) làm lớp thực thi, hoặc tích hợp như một "provider" dạng runtime, thay vì tự viết Runtime Manager?
3. **Kênh báo cáo của agent:** MCP server của factory hay CLI như Orca?
4. **Mô hình tiến trình:** chạy một lần mỗi lần thử (`-p`) hay tiến trình sống lâu (`--input-format stream-json`)? Cần làm rõ trong spike Phase 1 (xem ADR-0002).
