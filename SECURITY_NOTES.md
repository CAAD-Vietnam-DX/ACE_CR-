# Security Notes — Media Downloader & Processor

## 1. Phân loại Tool
- [ ] Tool chạy local (máy cá nhân)
- [x] Tool chạy shared (nhiều người dùng / server / cloud script)

## 2. Dữ liệu xử lý
- Loại data input: Tên file, Link Google Drive, định dạng file.
- Có xử lý PII không? [Không]
- Output lưu ở đâu: Log dữ liệu hệ thống / Base64.
- Phương thức lưu credentials: [Không lưu credentials. Xác thực bằng session nội bộ của Google].

## 3. Kết nối Mạng
- Có gọi external API không? [Không]
- Có gọi LLM API không? [Không]
- Danh sách kết nối (điền đầy đủ để DX Team xác nhận):

| URL | Phương thức truy cập | Mục đích sử dụng | Xác nhận ToS/Legal |
|---|---|---|---|
| N/A | N/A | Không có request ra ngoài mạng nội bộ Google | ✅ Đã xác nhận |

> ⚠️ Tool chỉ được DX Team approve khi **toàn bộ** hàng trong bảng có trạng thái ✅ Đã xác nhận.

## 4. Quyền truy cập cần thiết
- Database: [Không]
- File system: [Google Drive & Google Sheets của tài khoản Workspace nội bộ]
- Network: [Không mở port]

## 5. Rule đã áp dụng (theo file RULES_VN.md)
- [x] Không hardcode secret (Mục 1.1)
- [x] Không sử dụng GCP SDK hoặc file `credentials.json` (Mục 6.1)
- [ ] ⚠️ Dùng Standalone Script tách biệt với dữ liệu thay vì Container-bound (Mục 6.3.2) — **cần xác minh lại**, xem Mục 7 khuyến nghị #1
- [x] Không lưu trữ hoặc log `ScriptApp.getOAuthToken()` (Mục 6.3.3)

## 6. Điểm DX Team cần chú ý
- Code ban đầu sử dụng HTML Service để vẽ Popup lên file Sheet. Để tuân thủ hoàn toàn quy định về Container-bound Script (Mục 6.3.2), toàn bộ UI đã được lược bỏ. Script đã được tái cấu trúc thành Standalone Script ghi nhận kết quả thông qua Execution Log.
- ⚠️ **Cập nhật (audit ngày 2026-09-10):** Ghi chú trên chưa khớp với code hiện tại trong file `main` — code vẫn đang dùng `SpreadsheetApp.getActiveSpreadsheet()` và các hàm vẫn được mô tả là "gán vào nút" trên Sheet, cùng với `getUi()/showModalDialog/showModelessDialog`. Đây là dấu hiệu của container-bound script, không phải standalone. Xem chi tiết và hướng xử lý tại Mục 7.

## 7. Khuyến nghị / Hạng mục cần xử lý trước khi Approve
*(Bổ sung từ đợt audit ngày 2026-09-10 — đối chiếu RULES_VN.md)*

### Nghiêm trọng — bắt buộc xử lý trước khi Approve
- [ ] **Làm rõ kiến trúc Standalone vs Container-bound (Mục 6.3.2).** Code hiện dùng `SpreadsheetApp.getActiveSpreadsheet()` thay vì `SpreadsheetApp.openById(TARGET_SHEET_ID)`, và các hàm được gán trực tiếp vào nút vẽ trên Sheet — tính năng này chỉ hoạt động với container-bound script. Cần chọn một trong hai hướng:
  - Nếu tool thực sự chạy dạng bound → phải đánh giá lại theo đúng rủi ro Mục 6.3.2 (access list dùng chung với Spreadsheet, bất kỳ ai có quyền Viewer đều đọc được code).
  - Nếu muốn giữ đúng standalone như đã khai báo → sửa code dùng `openById()`, bỏ toàn bộ `getUi()/showModalDialog/showModelessDialog`, chuyển sang ghi nhận qua Execution Log, rồi cập nhật lại Mục 5 và Mục 6 cho khớp thực tế.
- [ ] **Nộp bổ sung file `Index.html`.** Code gọi `HtmlService.createTemplateFromFile('Index')` để xuất file ZIP nhưng file này chưa có trong repo/lịch sử git → DX Team chưa thể review phần xuất ZIP, đặc biệt là có nạp thư viện/script từ nguồn ngoài hay không (Mục 5.1, 7.1).

### Trung bình
- [ ] Đính kèm file `appsscript.json` (manifest) để xác minh `oauthScopes` khai báo ở mức tối thiểu cần thiết (Mục 1.3, 6.3.3) — việc đọc file/folder Drive theo ID bất kỳ nhiều khả năng cần scope Drive rộng hơn `drive.file`.
- [ ] Bổ sung `Logger.log()` khi bắt lỗi trong hàm `getFileBase64` (hiện trả về `null` mà không ghi log nguyên nhân lỗi) (Mục 4.2).
- [ ] Đổi tên file export từ `main` sang định dạng chuẩn `<ten_script>.gs.txt` (VD: `CR_MediaDownloader.gs.txt`) (Mục 6.3.4).

### Nhẹ
- [ ] Đồng bộ README.md: hướng dẫn hiện ghi "gắn ID Sheet vào biến `TARGET_SHEET_ID`" nhưng biến này chưa tồn tại trong code.
- [ ] Bổ sung thư mục `tests/` với ít nhất một checklist/test case thủ công (Mục 4.8).
- [ ] Bổ sung `.gitignore` để phòng trường hợp thêm config/manifest nhạy cảm sau này.
