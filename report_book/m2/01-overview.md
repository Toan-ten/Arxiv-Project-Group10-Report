# Tổng quan và cấu trúc dữ liệu

## 1. Mục tiêu của Giai đoạn 2

Giai đoạn 2 chuyển dữ liệu thô từ Giai đoạn 1 thành dữ liệu có cấu trúc cây và khử trùng lặp. Đầu vào là các tệp mã nguồn `.tex` và `.bib` của mọi phiên bản.

Ba công việc chính gồm:
1. Phân tích cú pháp phân cấp: Tìm tệp gốc có lệnh `\documentclass`, lần theo các chỉ thị `\input` và `\include`, bỏ tệp thừa, và chuyển văn bản thành cây phân cấp.
2. Chuẩn hóa văn bản: Dọn dẹp lệnh định dạng không mang ngữ nghĩa, đưa toán nội dòng về `$...$` và toán dạng khối về `\begin{equation}...\end{equation}`, rồi chuyển `\bibitem` thành BibTeX.
3. Khử trùng lặp qua các phiên bản: Hợp nhất các trường trích dẫn trong BibTeX, đổi tên lệnh `\cite` tương ứng, và dùng chung định danh phần tử cho các đoạn nội dung không đổi sau khi dọn định dạng.

Dữ liệu đầu ra được lưu tại nhóm để phục vụ bài toán đối sánh ở Giai đoạn 3.

## 2. Cơ chế kho lưu trữ chung giữa các phiên bản

Các phiên bản sửa đổi của một bài báo thường giữ nguyên phần lớn nội dung và chỉ cập nhật một số đoạn. Để tránh nhân bản dữ liệu, đề bài quy định lưu trữ chung giữa các phiên bản:
- Bảng phần tử `elements`: Lưu tập hợp phần tử phân biệt của bài báo qua mọi phiên bản. Nếu một câu hoặc công thức xuất hiện ở nhiều phiên bản với nội dung trùng nhau sau khi dọn định dạng, phần tử đó chỉ lưu đúng một dòng trong `elements` và mang cùng một `element_id`.
- Bảng cấu trúc `hierarchy`: Lưu các cạnh nối cha và con theo từng phiên bản. Một phần tử tạo từ phiên bản đầu có thể làm con của một tiêu đề mới ở phiên bản sau.
- Bản ghi rỗng: Các phiên bản dạng PDF hoặc Word không có mã nguồn TeX sẽ không tạo phần tử nào trong `elements` và được ghi nhận trong thống kê độ bao phủ.

## 3. Lược đồ ba bảng dữ liệu xuất bản

Dữ liệu sau khi xử lý được chuyển thành ba bảng Parquet:

Bảng `elements`: lưu các phần tử nội dung phân biệt qua mọi phiên bản.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `element_id` | String | Non-null, khóa chính | Định danh phần tử, ví dụ `2501-02642#s1` |
| `arxiv_id` | String | Non-null | Số hiệu bài báo |
| `kind` | String | Non-null | Loại phần tử theo từ điển chuẩn: `document`, `heading` (các cấp chương/mục), `paragraph`, `itemize`, `item`, và ba dạng nút lá gồm `sentence`, `block_formula`, `figure` (bao gồm cả bảng biểu `table`) |
| `content` | String | Non-null | Nội dung văn bản sau khi dọn định dạng và chuẩn hóa toán |

Bảng `hierarchy`: lưu các cạnh nối cha và con theo từng phiên bản.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `arxiv_id` | String | Non-null | Số hiệu bài báo |
| `version` | Integer | Non-null | Số thứ tự phiên bản |
| `element_id` | String | Non-null | Định danh phần tử con |
| `parent_id` | String | Non-null | Định danh phần tử cha hoặc gốc tài liệu |

Bảng `bibtex`: lưu các mục trích dẫn đã hợp nhất qua mọi phiên bản.

| **Tên cột** | **Kiểu dữ liệu** | **Ràng buộc** | **Ý nghĩa** |
| :--- | :--- | :--- | :--- |
| `arxiv_id` | String | Non-null | Số hiệu bài báo chứa trích dẫn |
| `citation_key` | String | Non-null | Khóa trích dẫn đã chọn |
| `entry_type` | String | Non-null | Loại trích dẫn như article, inproceedings, book |
| `title` | String | Non-null | Tiêu đề tài liệu được trích dẫn |
| `authors` | List of Strings | Non-null | Danh sách tác giả |
| `year` | Integer | Nullable | Năm xuất bản |
| `doi` | String | Nullable | Định danh đối tượng số nếu có |
| `journal` | String | Nullable | Tên tạp chí hoặc kỷ yếu hội thảo |
| `url` | String | Nullable | Đường dẫn trực tuyến |
