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
- Có gọi external API không? [Có — 1 kết nối, xem bảng dưới]
- Có gọi LLM API không? [Không]
- Danh sách kết nối (điền đầy đủ để DX Team xác nhận):

| URL | Phương thức truy cập | Mục đích sử dụng | Xác nhận ToS/Legal |
|---|---|---|---|
| https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js | Load script tĩnh phía client (trong `Index.html`, không qua `UrlFetchApp`) | Nén file thành ZIP ngay trên trình duyệt trước khi tải về máy user | ✅ Đã xác nhận |

> ⚠️ Cập nhật audit 2026-09-10: phát hiện `Index.html` nạp thư viện JSZip từ CDN — không nằm trong Blacklist Mục 6.2, nhưng bảng trên cần DX Team xác nhận trước khi approve.

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
*(Bổ sung từ đợt audit ngày 2026-09-10 — đối chiếu RULES_VN.md; cập nhật sau khi nhận thêm `Index.html` và `main.gs`)*

### Nghiêm trọng — bắt buộc xử lý trước khi Approve
- [ ] **[VẪN CHƯA XỬ LÝ] Làm rõ kiến trúc Standalone vs Container-bound (Mục 6.3.2).** Đã đối chiếu `main.gs` mới nộp với bản `main` cũ bằng `diff` — **nội dung giống hệt 100%**, chưa có thay đổi nào. Code vẫn dùng `SpreadsheetApp.getActiveSpreadsheet()` thay vì `SpreadsheetApp.openById(TARGET_SHEET_ID)`, và các hàm vẫn được mô tả "gán vào nút" vẽ trên Sheet — tính năng này chỉ hoạt động với container-bound script. Đây vẫn là **blocker chính** trước khi approve. Cần chọn một trong hai hướng:
  - Nếu tool thực sự chạy dạng bound → phải đánh giá lại theo đúng rủi ro Mục 6.3.2 (access list dùng chung với Spreadsheet, bất kỳ ai có quyền Viewer đều đọc được code).
  - Nếu muốn giữ đúng standalone như đã khai báo → sửa code dùng `openById()`, bỏ toàn bộ `getUi()/showModalDialog/showModelessDialog`, chuyển sang ghi nhận qua Execution Log, rồi cập nhật lại Mục 5 và Mục 6 cho khớp thực tế.
- [x] ~~Nộp bổ sung file `Index.html`~~ — **Đã nhận và review (2026-09-10).** File `Index.html` đã có trong repo. Kết quả review:
  - Không log/export `getOAuthToken()`, không hardcode secret. ✅
  - File ZIP chỉ tải xuống máy cục bộ của user qua `Blob URL` (`a.download`), không gửi dữ liệu ra server ngoài nào. ✅
  - ⚠️ Có nạp thư viện `JSZip 3.10.1` từ CDN `cdnjs.cloudflare.com` (dòng `<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js">`) — đã bổ sung vào bảng kết nối mạng ở Mục 3, cần DX Team xác nhận ToS/Legal.
  - ⚠️ Theo kiểm tra (Mục 5.1.3): JSZip chưa có bản release mới trong hơn 1 năm (bản 3.10.1 phát hành ~2022) → về nguyên tắc cần cảnh báo dấu hiệu "không còn tích cực bảo trì". Tuy nhiên đây là thư viện rất phổ biến (~40 triệu lượt tải/tuần), và lỗ hổng đã biết duy nhất (CVE-2022-48285 — Zip Slip qua `loadAsync`) **không áp dụng** cho cách dùng trong tool này (tool chỉ dùng `zip.file()` + `generateAsync()` để **tạo** ZIP, không dùng `loadAsync()` để **giải nén** file lạ). Khuyến nghị: DX Team ghi nhận và theo dõi định kỳ, chưa cần thay thế ngay.

### Trung bình
- [ ] Đính kèm file `appsscript.json` (manifest) để xác minh `oauthScopes` khai báo ở mức tối thiểu cần thiết (Mục 1.3, 6.3.3) — việc đọc file/folder Drive theo ID bất kỳ nhiều khả năng cần scope Drive rộng hơn `drive.file`.
- [ ] Bổ sung `Logger.log()` khi bắt lỗi trong hàm `getFileBase64` (hiện trả về `null` mà không ghi log nguyên nhân lỗi) (Mục 4.2).
- [x] ~~Đổi tên file export từ `main` sang định dạng chuẩn~~ — **Đã xử lý.** File đã đổi tên thành `main.gs`, hợp lệ theo Mục 6.3.4 (rule cho phép đuôi `.txt` hoặc `.gs`).

### Nhẹ
- [ ] Đồng bộ README.md: hướng dẫn hiện ghi "gắn ID Sheet vào biến `TARGET_SHEET_ID`" nhưng biến này chưa tồn tại trong code.