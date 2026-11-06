# Tổng quan và phạm vi thu thập

## 1. Mục tiêu của Giai đoạn 1

Giai đoạn 1 xây dựng tập dữ liệu thô từ kho lưu trữ mở arXiv để làm đầu vào cho bước phân tích cú pháp phân cấp ở Giai đoạn 2 và mô hình đối sánh trích dẫn ở Giai đoạn 3.

Ba nhiệm vụ kỹ thuật gồm:
1. Khám phá bài báo: Quét toàn bộ 20.000 vị trí số hiệu được giao, xác định các bài báo thực tế tồn tại và truy xuất danh sách mọi phiên bản công bố từ bản đầu tiên (`v1`) đến bản sửa đổi mới nhất (`vN`).
2. Tải và trích xuất mã nguồn LaTeX: Tải gói nguồn của từng phiên bản qua máy chủ phân phối `export.arxiv.org`, loại bỏ toàn bộ tệp hình ảnh để tiết kiệm dung lượng đĩa, chỉ giữ lại các tệp `.tex` và `.bib`. Các câu lệnh định nghĩa hình ảnh trong mã nguồn LaTeX được giữ nguyên để phục vụ dựng cây tài liệu ở Giai đoạn 2.
3. Thu thập tài liệu tham khảo: Lấy danh mục trích dẫn kèm số hiệu arXiv và định danh Semantic Scholar qua Semantic Scholar Graph API để tạo tập ứng viên cho bài toán đối sánh.

## 2. Phạm vi phân công và nguyên tắc thu thập

Nhóm 10 phụ trách bốn khoảng số hiệu arXiv liên tục trong năm 2025, tương ứng với bốn thành viên:

- Trương Tuấn Anh, MSSV 24120260: từ 2501.02642 đến 2501.07641, gồm 5.000 số hiệu thuộc tháng 01 năm 2025.
- Nguyễn Văn Toàn, MSSV 24120232: từ 2502.08235 đến 2502.13234, gồm 5.000 số hiệu thuộc tháng 02 năm 2025.
- Lê Thành Đạt, MSSV 24120035: từ 2507.16659 đến 2507.21658, gồm 5.000 số hiệu thuộc tháng 07 năm 2025.
- Trần Văn Tấn Khôi, MSSV 24120010: từ 2508.07872 đến 2508.12871, gồm 5.000 số hiệu thuộc tháng 08 năm 2025.

Tổng quy mô khảo sát là 20.000 vị trí số hiệu ứng viên, không phân biệt chuyên ngành. Để kiểm soát tần suất gọi mạng và đồng bộ định dạng lưu trữ, nhóm vận hành chung một đường ống thu thập thống nhất. Với mỗi bài báo hợp lệ, hệ thống tải toàn bộ các phiên bản hiện có; bài báo chỉ nộp một lần được gán hậu tố `v1`.

## 3. Chiến lược tập mẫu thử nghiệm tất định

Trước khi chạy trên toàn bộ 20.000 số hiệu, module `src.pilots` trích xuất hai tập mẫu tất định từ các dải số hiệu được giao:

- Tập mẫu kiểm thử khói (`pilot_10`): Gồm đúng 10 số hiệu ứng viên, lấy 2 số hiệu biên đầu và cuối (`range_start`, `range_end`) trong mỗi khoảng của 4 thành viên (8 số hiệu) cộng thêm 2 số hiệu điểm giữa (`mid`) tại hai khoảng đầu tiên (`2508.10371` và `2507.19158`). Tập mẫu này dùng để kiểm tra kết nối mạng, quyền ghi đĩa, cơ chế dừng khi gặp lỗi HTTP và thao tác trích xuất cơ bản.
- Tập mẫu kiểm chuẩn hiệu năng (`pilot_500`): Lấy 125 số hiệu trải đều (gồm cả hai đầu mút) cho mỗi dải của 4 thành viên, tổng cộng 500 số hiệu ứng viên thuộc các tháng 1, 2, 7 và 8 năm 2025. Tập mẫu này dùng để đo kiểm có kiểm soát mức tiêu thụ RAM, thời gian chạy và dung lượng đĩa trước khi chạy toàn bộ tập dữ liệu.

## 4. Tổ chức dữ liệu làm việc

Dữ liệu thu thập cục bộ tuân thủ hợp đồng lưu trữ phiên bản 2 (`contract_version = 2`). Mỗi bài báo nằm trong một thư mục riêng đặt tên theo số hiệu arXiv chuẩn hóa qua hàm `arxiv_id_to_folder_key()`, thay dấu chấm bằng dấu gạch nối:

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

Chức năng của từng tệp quản lý dữ liệu:
- `manifest.json`: Lưu trạng thái tổng hợp (`status`, `discovery_status`, `version_status`, `metadata_status`, `references_status`), tiến trình từng phiên bản, bảng mã băm SHA256 (`file_checksums`) của từng tệp mã nguồn và nhật ký lỗi. Mỗi bản ghi gắn với MSSV phụ trách theo tệp cấu hình.
- `metadata.json`: Lưu tiêu đề, danh sách tác giả, ngày nộp bản đầu (`submission_date`), danh sách ngày cập nhật (`revised_dates`), nơi công bố (`venue`), danh sách phiên bản và nguồn gốc dữ liệu (`provenance`).
- `references.json`: Lưu toàn bộ danh mục tài liệu tham khảo trung gian lấy từ Semantic Scholar (kể cả các mục không có số hiệu arXiv), thông tin nguồn gốc `provenance` và trạng thái độ phủ phân trang `coverage`.

Quy tắc tạo thư mục phiên bản: Thư mục con `v1/`, `v2/` chỉ được tạo khi có ít nhất một tệp `.tex` hoặc `.bib` được giữ lại. Nếu gói nén chỉ có tệp `.bib` mà không có `.tex`, hệ thống vẫn lưu tệp `.bib` nhưng ghi nhận `source_status = "no_tex_source"`. Với các bài báo chỉ có bản PDF hoặc PostScript thuần túy, hệ thống không tạo thư mục phiên bản rỗng mà ghi trực tiếp bản ghi phiên bản với `source_status = "no_tex_source"` vào `manifest.json`.

## 5. Lược đồ hai bảng dữ liệu xuất bản

Dữ liệu thu thập được lưu tại kho cục bộ của nhóm trong hai giai đoạn đầu và xuất thành các bảng Parquet ở Giai đoạn 3. Hai bảng dữ liệu của Giai đoạn 1 gồm:

Bảng `papers`: lưu thông tin chung và lịch sử cập nhật của từng bài báo hợp lệ.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `arxiv_id` | String | Non-null, khóa chính | Số hiệu gốc của bài báo, ví dụ `2501.02642` |
| `title` | String | Non-null | Tiêu đề của bài báo |
| `authors` | List of Strings | Non-null | Danh sách họ tên các tác giả |
| `submission_date` | String | Non-null | Ngày nộp bản đầu tiên theo định dạng ISO 8601 |
| `revised_dates` | List of Strings | Non-null | Danh sách ngày cập nhật các phiên bản tiếp theo |
| `venue` | String | Nullable | Nơi xuất bản hoặc tên hội nghị nếu có |

Bảng `references`: lưu các liên kết trích dẫn có số hiệu arXiv xác định.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `arxiv_id` | String | Non-null | Số hiệu bài báo thực hiện trích dẫn |
| `ref_arxiv_id` | String | Non-null | Số hiệu arXiv của bài báo được trích dẫn |
| `ref_title` | String | Non-null | Tiêu đề của bài báo được trích dẫn |
| `ref_authors` | List of Strings | Non-null | Danh sách tác giả của bài báo được trích dẫn |
| `ref_submission_date` | String | Nullable | Ngày công bố/nộp của bài báo được trích dẫn |
| `ref_s2_id` | String | Non-null | Định danh bài báo (`paperId`) trên hệ thống Semantic Scholar |
