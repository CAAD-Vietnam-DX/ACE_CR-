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
- [x] Dùng Standalone Script tách biệt với dữ liệu thay vì Container-bound (Mục 6.3.2)
- [x] Không lưu trữ hoặc log `ScriptApp.getOAuthToken()` (Mục 6.3.3)

## 6. Điểm DX Team cần chú ý
- Code ban đầu sử dụng HTML Service để vẽ Popup lên file Sheet. Để tuân thủ hoàn toàn quy định về Container-bound Script (Mục 6.3.2), toàn bộ UI đã được lược bỏ. Script đã được tái cấu trúc thành Standalone Script ghi nhận kết quả thông qua Execution Log.
