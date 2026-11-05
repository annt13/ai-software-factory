# CLAUDE.md

Hướng dẫn cho Claude khi làm việc trong repository này.

## Dự án

AI Software Factory: nền tảng local-first, chạy trên máy cá nhân, điều phối nhiều AI coding agent. Xem [README](README.md).

**Giai đoạn hiện tại: chuẩn bị tài liệu trước khi code** (Documentation Plan §3, §4 Phase 1). Repository chưa có mã nguồn. Không viết code ứng dụng cho tới khi bộ 12 tài liệu ưu tiên đạt tiêu chí hoàn thành ở Documentation Plan §7.2, trừ khi người dùng yêu cầu rõ (ví dụ spike kỹ thuật).

## Nguồn sự thật theo chủ đề

| Chủ đề | Tài liệu |
| ------ | -------- |
| Danh mục tài liệu, trạng thái, cấu trúc thư mục, chuẩn viết | `docs/DOCUMENTATION-PLAN.md` |
| Vision, nguyên tắc, người dùng mục tiêu | `README.md` §2–§4 |
| Mô tả module | `docs/01-product/product-overview.md` |
| Vấn đề, mục tiêu, danh sách tính năng (F-xx) | `docs/01-product/product-requirements.md` (DOC-01) |
| Kiến trúc tổng thể, bảng quyết định | `docs/02-architecture/system-architecture.md` (§11) |
| Quyết định kiến trúc | `docs/02-architecture/adr/` |
| Thứ tự phát triển, định nghĩa MVP | `docs/06-development/development-roadmap.md` |
| Nghiên cứu (chỉ tham khảo, không phải quyết định) | `docs/research/` |

## Quy tắc 1 — Mọi tài liệu phải đồng bộ với nhau

Đây là quy tắc quan trọng nhất. Một thay đổi chưa được coi là xong khi các tài liệu liên quan chưa được cập nhật.

1. **Trước khi sửa**, tìm mọi nơi tham chiếu tới nội dung sắp đổi: mã tài liệu (`DOC-xx`, `ADR-xxxx`, `F-xx`), tên file, và tham chiếu mục (ví dụ `Product Overview §2.5`). Dùng grep trên toàn bộ `docs/` và `README.md`.
2. **Sau khi sửa**, cập nhật tất cả những nơi đó trong cùng một lượt làm việc.
3. **Danh sách nơi phải cập nhật thêm** theo loại thay đổi:

   | Thay đổi | Cập nhật thêm |
   | -------- | ------------- |
   | Tạo tài liệu mới | Cây thư mục ở Documentation Plan §5; danh sách tài liệu ở README §8; trạng thái ở Documentation Plan §3.1 nếu là DOC-01…DOC-12 |
   | Đổi trạng thái DOC-01…DOC-12 | Documentation Plan §3.1 và `status` trong metadata của tài liệu |
   | Tạo hoặc đổi trạng thái ADR | `adr/README.md`; Documentation Plan §5 và §6.3; System Architecture §11 nếu là quyết định kiến trúc; mục "Open Questions" của tài liệu đã nêu câu hỏi đó (gạch câu hỏi và ghi "Resolved <ngày>: …") |
   | Đánh số lại hoặc đổi tên mục | Mọi tham chiếu `§` trỏ tới mục đó |
   | Đổi tên hoặc di chuyển file | Mọi link tương đối trỏ tới file đó |

4. **Khi phát hiện mâu thuẫn** giữa các tài liệu: không tự chọn một bên và sửa im lặng. Báo cho người dùng, nêu rõ hai vị trí mâu thuẫn, đề xuất cách giải quyết, và ghi vào mục "Open Questions" của tài liệu liên quan nếu chưa giải quyết ngay.
5. **Cuối mỗi lượt sửa**, liệt kê cho người dùng những file đã đồng bộ.

## Viết tài liệu

- **Không lặp nội dung** (Documentation Plan §6.1). Nội dung có nguồn sự thật ở tài liệu khác thì tóm tắt ngắn và dẫn link, không chép lại.
- **Mỗi tầng tài liệu đúng vai trò.** Tài liệu sản phẩm (`01-product`) nói *cái gì* và *vì sao*, không nêu công nghệ, thư viện hay công cụ bên thứ ba cụ thể. Chi tiết triển khai thuộc về tài liệu kiến trúc và ADR.
- **Metadata:** tài liệu mới bắt đầu bằng khối YAML theo Documentation Plan §6.2, mã tài liệu theo quy ước ở cùng mục đó. Khi sửa nội dung, cập nhật `updated_at`, và tăng `version` nếu nội dung thay đổi về ý nghĩa.
- **Dẫn nguồn** theo dạng `Tên tài liệu §N`, kèm link tương đối khi nhắc lần đầu.
- **Thông tin về provider** (flag CLI, hành vi, điều khoản) phải kiểm tra trên tài liệu chính thức và ghi nguồn cùng ngày đọc, như ADR-0001 đã làm. Không viết theo trí nhớ.
- **Văn phong:** câu ngắn, rõ; ưu tiên bảng và danh sách; không dùng ngôn ngữ quảng cáo.

## Quyết định và trạng thái

- **Không ghi một quyết định chưa được duyệt như quyết định chính thức** (Documentation Plan §6.1). Nội dung do Claude đề xuất phải đánh dấu *Draft* hoặc đặt ở "Open Questions".
- Chỉ người dùng được duyệt quyết định. Chỉ chuyển ADR sang `Accepted`, hoặc tài liệu sang `Approved`, khi người dùng đã quyết định rõ.
- Quyết định kiến trúc quan trọng ghi thành ADR từ `adr/template.md`, đánh số tiếp theo. Một ADR đã Accepted mà bị thay thế thì tạo ADR mới và đổi ADR cũ sang `Superseded by ADR-xxxx`, không xóa.
- Những gì còn chưa quyết thì đánh dấu **TBD** và chỉ rõ tài liệu nào sẽ quyết.

## Ngôn ngữ

- Giữ nguyên ngôn ngữ hiện có của từng tài liệu: `DOCUMENTATION-PLAN.md` và `docs/research/` viết tiếng Việt; README, `01-product`, `02-architecture`, ADR và roadmap viết tiếng Anh.
- Tài liệu mới trong `01-product` đến `07-open-source` và ADR viết bằng tiếng Anh.
- Trả lời người dùng bằng tiếng Việt.

## Dữ liệu nhạy cảm

Không ghi vào tài liệu: email, ID tài khoản hoặc tổ chức, token, credential, hay bất kỳ thông tin cá nhân nào, kể cả khi lấy từ output của lệnh (ví dụ `claude auth status`).

## Git

- Chỉ commit khi người dùng yêu cầu. Không push nếu chưa được yêu cầu.
- Commit message theo dạng `docs: <mô tả>`, có danh sách thay đổi chính trong phần thân, như các commit hiện có.
- Không làm việc trực tiếp trên `main`; tài liệu nền tảng đang làm trên nhánh `docs/foundation`.
