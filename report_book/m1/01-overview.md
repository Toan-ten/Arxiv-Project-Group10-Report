# Tổng quan và phạm vi thu thập

## 1. Mục tiêu của Giai đoạn 1

Giai đoạn 1 xây dựng tập dữ liệu thô từ kho lưu trữ mở arXiv. Tập dữ liệu này đóng vai trò là nền tảng đầu vào trực tiếp cho bước phân tích cú pháp phân cấp ở Giai đoạn 2 và bài toán đối sánh trích dẫn học thuật ở Giai đoạn 3.

Ba mục tiêu kỹ thuật trọng tâm gồm:
1. Khám phá bài báo: Quét toàn bộ 20.000 vị trí số hiệu được giao, xác định các bài báo thực tế tồn tại và truy xuất danh sách mọi phiên bản công bố từ bản đầu tiên đến bản sửa đổi mới nhất.
2. Tải và trích xuất mã nguồn LaTeX: Thu nhận gói nguồn của từng phiên bản qua máy chủ phân phối, lọc bỏ toàn bộ tệp hình ảnh để tiết kiệm dung lượng đĩa, chỉ trích xuất các tệp `.tex` và `.bib`. Các câu lệnh mô tả hình ảnh trong mã nguồn LaTeX được bảo toàn nguyên vẹn để phục vụ dựng cây tài liệu.
3. Thu thập tài liệu tham khảo: Thu nhận danh mục trích dẫn kèm số hiệu arXiv thông qua Semantic Scholar Graph API, phục vụ xây dựng tập ứng viên cho bài toán đối sánh.

## 2. Phạm vi phân công và nguyên tắc thu thập

Nhóm 10 phụ trách bốn khoảng số hiệu arXiv liên tục trong năm 2025, tương ứng với nhiệm vụ của bốn thành viên:

- Trương Tuấn Anh, MSSV 24120260: từ 2501.02642 đến 2501.07641, quy mô 5.000 số hiệu thuộc tháng 01 năm 2025.
- Nguyễn Văn Toàn, MSSV 24120232: từ 2502.08235 đến 2502.13234, quy mô 5.000 số hiệu thuộc tháng 02 năm 2025.
- Lê Thành Đạt, MSSV 24120035: từ 2507.16659 đến 2507.21658, quy mô 5.000 số hiệu thuộc tháng 07 năm 2025.
- Trần Văn Tấn Khôi, MSSV 24120010: từ 2508.07872 đến 2508.12871, quy mô 5.000 số hiệu thuộc tháng 08 năm 2025.

Tổng quy mô cần khảo sát là 20.000 vị trí số hiệu. Dữ liệu thu thập căn cứ trực tiếp vào dải số hiệu định danh được phân bổ mà không phân biệt chuyên ngành hay thể loại bài báo. Nhằm tối ưu hóa tài nguyên mạng và năng lực tính toán, toàn bộ tiến trình thu thập được vận hành tập trung thông qua một đường ống xử lý thống nhất của nhóm.

Với mỗi bài báo hợp lệ, hệ thống thu thập tất cả các phiên bản có sẵn. Bài báo nộp một lần được quản lý với hậu tố `v1`.

## 3. Chiến lược tập mẫu thử nghiệm tất định

Trước khi vận hành đường ống trên toàn bộ quy mô 20.000 số hiệu, hệ thống thiết lập hai tập mẫu thử nghiệm tất định được trích xuất trực tiếp từ các dải số hiệu phân công:

- Tập mẫu kiểm thử khói (`pilot_10`): Lấy 2 số hiệu đầu tiên trong mỗi khoảng phân công của 4 thành viên (tổng cộng 8 bài đại diện). Tập mẫu này phục vụ kiểm tra thông suốt kết nối mạng, quyền ghi đĩa, cơ chế dừng khẩn cấp và các thao tác bóc tách cơ bản mà không làm tiêu tốn tài nguyên.
- Tập mẫu kiểm chuẩn hiệu năng (`pilot_500`): Lấy chính xác 125 số hiệu cho mỗi dải phân công của 4 thành viên (tổng cộng 500 bài). Tập mẫu này bao phủ đồng đều các giai đoạn khác nhau trong năm 2025 gồm tháng 1, tháng 2, tháng 7 và tháng 8. Đây là tập dữ liệu chuẩn mực để đo kiểm mức tiêu thụ bộ nhớ RAM, thời gian chạy và dung lượng lưu trữ có kiểm soát trước khi nhân rộng toàn diện.

## 4. Tổ chức dữ liệu làm việc

Dữ liệu thu thập cục bộ tuân thủ nghiêm ngặt hợp đồng lưu trữ phiên bản 2. Mỗi bài báo được quản lý trong một thư mục riêng biệt đặt tên theo số hiệu arXiv chuẩn hóa, sử dụng dấu gạch nối thay cho dấu chấm:

```text
working_corpus/
└── 2501-02642/
    ├── manifest.json
    ├── metadata.json
    ├── references.json
    ├── v1/
    │   ├── main.tex
    │   └── references.bib
    └── v2/
        ├── main.tex
        └── references.bib
```

Vai trò của các tệp quản lý dữ liệu:
- `manifest.json`: Tệp trung tâm điều phối trạng thái, lưu trữ tiến trình khám phá, mã băm SHA256 kiểm tra tính toàn vẹn của từng tệp mã nguồn và nhật ký lỗi nếu có. Mỗi bản ghi trạng thái đều gắn liền với định danh MSSV phụ trách theo tệp cấu hình phân công.
- `metadata.json`: Lưu thông tin chung của bài báo gồm tiêu đề, danh sách tác giả, ngày nộp, ngày cập nhật và nguồn gốc trích xuất.
- `references.json`: Lưu danh mục tài liệu tham khảo thô nhận được từ Semantic Scholar.

Quy tắc tạo thư mục phiên bản: Thư mục con dạng `v1/`, `v2/` chỉ được tạo trên đĩa khi có ít nhất một tệp `.tex` hoặc `.bib` được trích xuất thành công. Với các bài báo không nộp bằng LaTeX, chẳng hạn như bản nộp bằng PDF thuần túy, hệ thống không tạo thư mục phiên bản rỗng mà ghi nhận trực tiếp trạng thái `source_status = "no_tex_source"` trong tệp `manifest.json`.

## 5. Lược đồ hai bảng dữ liệu xuất bản

Toàn bộ dữ liệu thu thập được lưu trữ tại kho cục bộ của nhóm trong hai giai đoạn đầu. Đến Giai đoạn 3, dữ liệu được chuyển đổi thành các bảng định dạng Parquet để phát hành. Hai bảng dữ liệu của Giai đoạn 1 gồm:

Bảng `papers`: lưu trữ thông tin chung và lịch sử cập nhật của từng bài báo hợp lệ.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `arxiv_id` | String | Non-null, khóa chính | Số hiệu gốc của bài báo, ví dụ `2501.02642` |
| `title` | String | Non-null | Tiêu đề của bài báo |
| `authors` | List of Strings | Non-null | Danh sách họ tên các tác giả |
| `submission_date` | String | Non-null | Ngày nộp bản đầu tiên theo định dạng ISO 8601 |
| `revised_dates` | List of Strings | Non-null | Danh sách ngày cập nhật các phiên bản tiếp theo |
| `venue` | String | Nullable | Nơi xuất bản hoặc tên hội nghị nếu có |

Bảng `references`: lưu trữ các liên kết trích dẫn có số hiệu arXiv xác định.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `arxiv_id` | String | Non-null | Số hiệu bài báo thực hiện trích dẫn |
| `ref_arxiv_id` | String | Non-null | Số hiệu arXiv của bài báo được trích dẫn |
| `ref_title` | String | Non-null | Tiêu đề của bài báo được trích dẫn |
| `ref_authors` | List of Strings | Non-null | Danh sách tác giả của bài báo được trích dẫn |
| `ref_submission_date` | String | Nullable | Ngày công bố của bài báo được trích dẫn |
| `ref_s2_id` | String | Non-null | Định danh bài báo trên hệ thống Semantic Scholar |
