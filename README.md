# Hướng dẫn sử dụng: Media Downloader & Processor

Công cụ làm sạch, đối chiếu và trích xuất hàng loạt file Media (Hình ảnh, Video) từ link thư mục/tệp tin Google Drive được dán vào Google Sheets.

### Yêu cầu cài đặt
- Chạy dưới dạng **Standalone Script** trên nền tảng Google Apps Script bằng tài khoản nội bộ (`@ca-adv...`).
- Cần cấp quyền truy cập vào Google Sheets và Google Drive khi chạy lần đầu.

### Hướng dẫn sử dụng (Quy trình Standalone)
1. Truy cập vào file Google Sheets, dán toàn bộ dữ liệu làm việc của bạn vào Sheet `データ` (bắt đầu từ dòng 12).
2. Mở trình duyệt `script.google.com`.
3. Gắn ID của file Sheets vào biến `TARGET_SHEET_ID` trong code.
4. Chọn hàm bạn muốn thực thi trên thanh công cụ (VD: `downloadCRImages` hoặc `duplicateCRImages`) và nhấn **Run**.
5. Do script không hỗ trợ popup giao diện (để tuân thủ bảo mật), vui lòng mở mục **Execution Log (Nhật ký thực thi)** bên dưới trình soạn thảo để xem trạng thái hoàn thành hoặc các lỗi nếu có.

### Troubleshoot các lỗi phổ biến
- **Cảnh báo thiếu quyền Drive:** Hãy đảm bảo tài khoản chạy script đã được cấp quyền Viewer đối với các Link Drive được đưa vào bảng. Script sẽ bỏ qua và tự log cảnh báo với các link bị chặn quyền.
- **Liên hệ hỗ trợ:** Nếu bạn gặp lỗi không thể khắc phục, vui lòng liên hệ **DX Team** trên Slack.
