# Tổng quan và phạm vi thu thập

## 1. Mục tiêu của Giai đoạn 1

Giai đoạn 1 xây dựng tập dữ liệu thô từ kho lưu trữ arXiv. Dữ liệu này là đầu vào trực tiếp cho bước phân tích cú pháp ở Giai đoạn 2 và bài toán đối sánh trích dẫn ở Giai đoạn 3.

Ba công việc chính gồm:
1. Khám phá bài báo: Quét 20.000 vị trí ID được giao, tìm các bài báo thực tế tồn tại và xác định toàn bộ các phiên bản công bố.
2. Tải mã nguồn LaTeX: Tải gói nguồn của từng phiên bản qua máy chủ phân phối, lọc bỏ tệp ảnh và chỉ giữ tệp `.tex` cùng `.bib`. Các lệnh `figure` và `\includegraphics` trong mã TeX được giữ nguyên để phục vụ dựng cây ở Giai đoạn 2.
3. Lấy dữ liệu trích dẫn: Lấy danh mục tài liệu tham khảo kèm arXiv ID qua API của Semantic Scholar để làm tập ứng viên cho Giai đoạn 3.

## 2. Phạm vi phân công

Nhóm 10 phụ trách bốn khoảng arXiv ID tương ứng với bốn thành viên:

- Trương Tuấn Anh (24120260): từ 2501.02642 đến 2501.07641 (5.000 số hiệu, tháng 01 năm 2025).
- Nguyễn Văn Toàn (24120232): từ 2502.08235 đến 2502.13234 (5.000 số hiệu, tháng 02 năm 2025).
- Lê Thành Đạt (24120035): từ 2507.16659 đến 2507.21658 (5.000 số hiệu, tháng 07 năm 2025).
- Trần Văn Tấn Khôi (24120010): từ 2508.07872 đến 2508.12871 (5.000 số hiệu, tháng 08 năm 2025).

Tổng số vị trí cần kiểm tra là 20.000 ID. Với mỗi bài báo hợp lệ, chương trình thu thập phải tải mọi phiên bản có sẵn. Bài báo chỉ nộp một lần được ghi nhận với hậu tố `v1`.

## 3. Tổ chức dữ liệu làm việc

Dữ liệu thu thập được lưu trên đĩa theo từng ấn phẩm. Mỗi bài báo nằm trong một thư mục mang tên số hiệu arXiv, dùng dấu gạch nối thay cho dấu chấm:

```text
working_corpus/
└── 2501-02642/
    ├── metadata.json
    ├── references_s2.json
    ├── v1/
    │   ├── main.tex
    │   └── references.bib
    ├── v2/
    │   ├── main.tex
    │   └── references.bib
    └── v3/
        └── empty_record.json
```

Nhóm lưu toàn bộ kết quả Semantic Scholar trả về ở máy cục bộ. Khi nộp dữ liệu cuối cùng ở Giai đoạn 3, chỉ các trích dẫn có arXiv ID mới đưa vào bảng `references`.

## 4. Lược đồ hai bảng dữ liệu xuất bản

Dữ liệu thu thập được giữ tại nhóm và chuyển thành bảng Parquet khi nộp Giai đoạn 3. Hai bảng của Giai đoạn 1 gồm:

Bảng `papers`: lưu thông tin chung và lịch sử sửa đổi của từng bài báo hợp lệ.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `arxiv_id` | String | Non-null, khóa chính | Số hiệu gốc của bài báo, ví dụ `2501.02642` |
| `title` | String | Non-null | Tiêu đề bài báo |
| `authors` | List of Strings | Non-null | Danh sách họ tên tác giả |
| `submission_date` | String | Non-null | Ngày nộp bản đầu tiên theo chuẩn ISO 8601 |
| `revised_dates` | List of Strings | Non-null | Danh sách ngày cập nhật các phiên bản tiếp theo |
| `venue` | String | Nullable | Nơi xuất bản hoặc hội nghị nếu có |

Bảng `references`: lưu các trích dẫn có arXiv ID, mỗi dòng là một liên kết trích dẫn.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `arxiv_id` | String | Non-null | Số hiệu bài báo thực hiện trích dẫn |
| `ref_arxiv_id` | String | Non-null | Số hiệu arXiv của bài báo được trích dẫn |
| `ref_title` | String | Non-null | Tiêu đề của bài báo được trích dẫn |
| `ref_authors` | List of Strings | Non-null | Danh sách tác giả của bài báo được trích dẫn |
| `ref_submission_date` | String | Nullable | Ngày công bố của bài báo được trích dẫn |
| `ref_s2_id` | String | Non-null | Định danh bài báo trên Semantic Scholar |
