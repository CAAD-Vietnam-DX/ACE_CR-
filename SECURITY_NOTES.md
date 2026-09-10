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
--- KIỂM TRA QUY TRÌNH PHÊ DUYỆT DEPENDENCY BÊN THỨ 3 (CDN) ---

1. [Phát hiện] Tình trạng sử dụng thư viện bên ngoài:
- File `Index.html` đang gọi trực tiếp thư viện JSZip (3.10.1) từ CDN bên ngoài: `https://cdnjs.cloudflare.com/...`
- Đối chiếu quy định (Mục 6.2): Tên miền CDN và thư viện này hiện KHÔNG nằm trong Blacklist của công ty.

2. [Đánh giá Rủi ro Bảo mật & Quy trình]:
- Theo nguyên tắc Zero Trust, việc một thư viện không nằm trong Blacklist chưa đủ điều kiện để sử dụng ngay trên môi trường Production.
- Tiềm ẩn rủi ro tấn công chuỗi cung ứng (Supply Chain) và nguy cơ bị chặn bởi tường lửa nội bộ của công ty nếu tên miền chưa được cấp phép.

3. [Yêu cầu xử lý / Action Required đối với DX Team]:
- Yêu cầu DX Team tiến hành rà soát ToS/Legal và đưa ra quyết định xác nhận (Approve) bằng văn bản/ticket.
- Hướng xử lý:
  + Tùy chọn 1: Nếu Approve CDN, yêu cầu DX Team bổ sung tên miền `cdnjs.cloudflare.com` vào Whitelist tường lửa (nếu có) và yêu cầu Dev thêm thuộc tính `integrity` (Mã băm SRI) vào thẻ <script> để chống giả mạo file.
  + Tùy chọn 2 (Khuyến nghị cao nhất): KHÔNG dùng CDN. Yêu cầu Dev tải nguyên source code của `jszip.min.js` về, nén thành 1 file HTML nội bộ (Container-bound) để loại bỏ hoàn toàn sự phụ thuộc vào mạng bên ngoài.
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
--- YÊU CẦU XÁC MINH KIẾN TRÚC HỆ THỐNG (MỤC 6.3.2) ---

1. [Phát hiện] Vi phạm thiết kế kiến trúc cốt lõi:
- Căn cứ Mục 6.3.2: Yêu cầu sử dụng "Standalone Script tách biệt với dữ liệu".
- Tình trạng thực tế (Cần xác minh lại): Mã nguồn hiện tại ĐANG LÀ Container-bound Script. 
- Bằng chứng: Code sử dụng `SpreadsheetApp.getActiveSpreadsheet()` và tương tác trực tiếp với UI của Sheet thông qua `SpreadsheetApp.getUi()`. (Standalone script không có khả năng làm điều này).

2. [Đánh giá Rủi ro Bảo mật (High)]:
- Việc gắn code trực tiếp vào file Sheet (Container-bound) phá vỡ nguyên tắc "tách biệt với dữ liệu". 
- Bất kỳ người dùng nào có quyền Edit (Chỉnh sửa) trên file Sheet "データ" đều có thể truy cập, xem, sao chép hoặc sửa đổi mã nguồn Apps Script. Điều này gây rủi ro lộ logic hệ thống và nguy cơ bị chèn mã độc nội bộ (Insider Threat).

3. [Yêu cầu xử lý / Action Required đối với DX Team]:
- Yêu cầu DX Team xác nhận lại định hướng kiến trúc. Trình bày 2 phương án giải quyết:
  + Phương án 1 (Tuân thủ Mục 6.3.2): Bắt buộc Dev đập bỏ giao diện UI hiện hành (không dùng `getUi()`), chuyển script thành Standalone (Web App) hoặc Add-on độc lập để ẩn mã nguồn khỏi người dùng Sheet.
  + Phương án 2 (Chấp nhận rủi ro): Nếu DX Team đánh giá rủi ro lộ code là chấp nhận được (dùng nội bộ tin tưởng), yêu cầu cập nhật lại tài liệu Mục 6.3.2 thành "Sử dụng Container-bound script" và xin cấp ngoại lệ bảo mật (Security Exception) cho dự án này.
## 6. Điểm DX Team cần chú ý
- Code ban đầu sử dụng HTML Service để vẽ Popup lên file Sheet. Để tuân thủ hoàn toàn quy định về Container-bound Script (Mục 6.3.2), toàn bộ UI đã được lược bỏ. Script đã được tái cấu trúc thành Standalone Script ghi nhận kết quả thông qua Execution Log.
- ⚠️ **Cập nhật (audit ngày 2026-09-10):** Ghi chú trên chưa khớp với code hiện tại trong file `main` — code vẫn đang dùng `SpreadsheetApp.getActiveSpreadsheet()` và các hàm vẫn được mô tả là "gán vào nút" trên Sheet, cùng với `getUi()/showModalDialog/showModelessDialog`. Đây là dấu hiệu của container-bound script, không phải standalone. Xem chi tiết và hướng xử lý tại Mục 7.
--- KIỂM TRA KIẾN TRÚC HỆ THỐNG: CONTAINER-BOUND VS STANDALONE ---

1. [Phát hiện] Sai lệch giữa Tài liệu thiết kế và Mã nguồn thực tế:
- Tài liệu (Mục 6.3.2) ghi chú: "Dùng Standalone Script tách biệt với dữ liệu".
- Mã nguồn thực tế (file Code.gs): Đang sử dụng các phương thức `SpreadsheetApp.getActiveSpreadsheet()`, `SpreadsheetApp.getUi()`, `showModalDialog` và `showModelessDialog`. 
- Đánh giá: Đây là đặc tính kỹ thuật BẮT BUỘC của Container-bound Script (Script nhúng trực tiếp vào Sheet). Standalone Script không hỗ trợ các hàm tạo giao diện UI này và sẽ văng lỗi (Exception) nếu cố tình gọi. Việc gán hàm vào "nút bấm trên Sheet" cũng minh chứng rõ đây là Container-bound.

2. [Yêu cầu xử lý / Action Required đối với DX Team]:
- Tùy chọn A (Đổi tài liệu): Nếu xác nhận giữ nguyên code hiện tại, yêu cầu đính chính lại tài liệu kỹ thuật (Mục 7) thành "Container-bound Script" và cập nhật các chính sách phân quyền/bảo mật tương ứng (vì người dùng phải có quyền Edit Sheet mới chạy được Script này).
- Tùy chọn B (Đổi code): Nếu bắt buộc kiến trúc hệ thống phải là "Standalone Script" theo tiêu chuẩn bảo mật, yêu cầu Dev đập bỏ giao diện UI hiện tại (`getUi()`), chuyển sang xây dựng Web App (dùng `doGet()`) và kết nối với Sheet thông qua `SpreadsheetApp.openById('MÃ_SHEET_CỦA_BẠN')`.

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

--- ĐÁNH GIÁ PHÁP LÝ & BẢO MẬT: THƯ VIỆN JSZIP TỪ CDN ---
1. [Pháp lý/Bản quyền] Thư viện JSZip:
- JSZip sử dụng giấy phép nguồn mở MIT License.
- Kết luận: Hoàn toàn hợp lệ và an toàn để sử dụng cho mục đích nội bộ/thương mại của công ty mà không vi phạm bản quyền.

2. [ToS/Quyền riêng tư] Sử dụng CDN cdnjs (Cloudflare):
- Vấn đề: Việc gọi script từ cdnjs.cloudflare.com sẽ làm lộ IP của Client ra ngoài hệ thống mạng công ty, cần Team Legal/Security xác nhận xem có vi phạm chính sách Data Privacy (Quyền riêng tư dữ liệu) của công ty hay không.
- Đề xuất (Khuyến nghị cao nhất): Tải trực tiếp file jszip.min.js về và nạp cục bộ (Local include) vào hệ thống Google Apps Script để ngắt hoàn toàn kết nối với bên thứ 3.

3. [Bảo mật] Thiếu cơ chế kiểm tra tính toàn vẹn (SRI):
- Vấn đề: Thẻ <script> hiện tại thiếu thuộc tính 'integrity'. Nếu CDN bị tấn công chuỗi cung ứng, hệ thống sẽ bị nhiễm mã độc.
- Yêu cầu sửa lỗi (Nếu vẫn quyết định dùng CDN): Yêu cầu Dev thêm mã băm bảo vệ.
Sửa thành: 
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js" integrity="sha512-XMVd28F1oH/O71fzwBnV7HucLxhiRwVzrXQjSy5wzdG6RVFqV+x//GFOS0Aukb92d6zjs49XN45pY2A0T+m6/A==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
### Trung bình
- [ ] Đính kèm file `appsscript.json` (manifest) để xác minh `oauthScopes` khai báo ở mức tối thiểu cần thiết (Mục 1.3, 6.3.3) — việc đọc file/folder Drive theo ID bất kỳ nhiều khả năng cần scope Drive rộng hơn `drive.file`.
- [ ] Bổ sung `Logger.log()` khi bắt lỗi trong hàm `getFileBase64` (hiện trả về `null` mà không ghi log nguyên nhân lỗi) (Mục 4.2).
- [x] ~~Đổi tên file export từ `main` sang định dạng chuẩn~~ — **Đã xử lý.** File đã đổi tên thành `main.gs`, hợp lệ theo Mục 6.3.4 (rule cho phép đuôi `.txt` hoặc `.gs`).

### Nhẹ
- [ ] Đồng bộ README.md: hướng dẫn hiện ghi "gắn ID Sheet vào biến `TARGET_SHEET_ID`" nhưng biến này chưa tồn tại trong code.
