# Quy trình phân tích cú pháp và khử trùng lặp

## 1. Dò tìm đường biên dịch đa tệp

Mã nguồn của một bài báo trên arXiv thường nằm ở nhiều tệp `.tex` khác nhau và nối với nhau qua lệnh `\input` hoặc `\include`.

Trong thư mục nguồn nộp lên arXiv thường có tệp nháp hoặc tệp mẫu không dùng. Để lấy đúng nội dung như trong tệp PDF, bộ phân tích cú pháp phải đi theo đường biên dịch:
- Xác định tệp gốc: Quét các tệp `.tex` trong thư mục phiên bản để tìm tệp chứa lệnh `\documentclass`. Tệp này là gốc của quá trình biên dịch.
- Lần theo lệnh liên kết: Bắt đầu từ tệp gốc, thuật toán lần theo mọi chỉ thị `\input` và `\include` đệ quy.
- Bỏ qua lệnh bị chú thích: Các dòng lệnh bắt đầu bằng ký tự `%` hoặc chỉ thị nằm sau ký tự `%` được xem là chú thích và bị bỏ qua.
- Loại bỏ tệp thừa: Các tệp `.tex` nằm trong thư mục nguồn nhưng không thuộc chuỗi liên kết `\input` hoặc `\include` từ tệp gốc sẽ bị loại bỏ.

## 2. Xây dựng cây phân cấp

Sau khi nối nội dung từ các tệp thuộc đường biên dịch, bộ phân tích cú pháp chuyển văn bản thành cây logic.

### 2.1. Cấu trúc cây

Mỗi phiên bản của bài báo được mô hình hóa thành một cây:
- Gốc tài liệu: Đại diện cho toàn bộ tài liệu của phiên bản đó.
- Các tầng trung gian: Tương ứng với các cấp mục `\chapter`, `\section`, `\subsection`, `\subsubsection` và `\paragraph`. Bộ phân tích cú pháp tự động nhận diện cấp bậc tương đối giữa các lệnh để gán mức phù hợp.
- Môi trường liệt kê: Khối `\begin{itemize}` hoặc `\begin{enumerate}` là nút phân nhánh, mỗi mục con `\item` là một phần tử con ở cấp sâu hơn.

### 2.2. Ba dạng nút lá

Nút lá là đơn vị nội dung nhỏ nhất trên cây, gồm:
- Câu văn xuôi: Các đoạn văn bản thân bài được tách thành các câu đơn lẻ dựa trên dấu chấm ngắt câu. Các trường hợp không ngắt câu gồm: bên trong công thức toán, các từ viết tắt như i.e., e.g., et al., Fig. 1, Eq. 2, địa chỉ URL, và dấu chấm trong số thập phân như 3.1415.
- Khối công thức toán: Nội dung của các môi trường toán khối như `\begin{equation}`, `\begin{align}`, `\begin{gather}` hoặc cặp dấu `$$...$$` được giữ thành một nút lá.
- Hình ảnh và bảng biểu: Khối `\begin{figure}` và `\begin{table}` đều là nút lá dạng hình ảnh, giữ nguyên cấu trúc chú thích và nhãn.

### 2.3. Quy tắc loại trừ và bao hàm

- Loại bỏ phần tài liệu tham khảo: Khối `\begin{thebibliography}` hoặc lệnh gọi tệp BibTeX `\bibliography` không đưa vào cây phân cấp vì danh mục trích dẫn được tách riêng sang bảng `bibtex`.
- Giữ lại lời cảm ơn và phụ lục: Phần lời cảm ơn và phụ lục, thường viết dưới dạng không đánh số `\section*`, bắt buộc phải giữ lại trên cây phân cấp.

## 3. Chuẩn hóa nội dung

Văn bản thô được chuẩn hóa qua ba bước:
- Dọn dẹp mã LaTeX: Xóa các lệnh định dạng không mang ngữ nghĩa, gồm lệnh căn lề `\centering`, tùy chọn vị trí `[htpb]`, lệnh kẻ bảng `\midrule`, `\toprule`, `\bottomrule`, và lệnh chỉnh cỡ chữ `\small`, `\large`.
- Chuẩn hóa công thức toán: Toán nội dòng được đưa đồng nhất về `$ ... $`. Toán khối được đưa về `\begin{equation} ... \end{equation}`.
- Chuyển `\bibitem` thành BibTeX: Với bài báo định nghĩa trích dẫn bằng lệnh `\bibitem` trong TeX, bộ phân tích cú pháp dùng biểu thức chính quy để bóc tách tác giả, tiêu đề, năm và tạp chí sang định dạng BibTeX.

## 4. Khử trùng lặp qua các phiên bản

Khử trùng lặp chạy trên hai cấp độ:

### 4.1. Khử trùng lặp mục trích dẫn

Khi sửa bài, tác giả có thể đổi khóa trích dẫn hoặc cập nhật thông tin:
- Hợp nhất các trường: Nếu hai mục trích dẫn ở hai phiên bản chỉ về cùng một tài liệu, hệ thống hợp nhất các trường thông tin. Ví dụ, phiên bản 1 dùng khóa `vaswani2017` với các trường title, author, year. Phiên bản 2 đổi khóa thành `vit_transformer` và thêm trường doi. Phiên bản 3 giữ khóa `vit_transformer` nhưng bỏ trường year. Bản ghi gộp sẽ chọn khóa `vit_transformer` và chứa đủ bốn trường: title, author, year, doi.
- Đổi tên lệnh `\cite`: Các lệnh trích dẫn `\cite{vaswani2017}` ở các đoạn văn bản thuộc phiên bản 1 được đổi thành `\cite{vit_transformer}` để trỏ đúng khóa đã chọn.

### 4.2. Khử trùng lặp nội dung toàn văn

Khử trùng lặp phần tử chạy sau bước dọn dẹp định dạng để tránh khoảng trắng làm lệch kết quả so sánh.

Khi một câu hoặc công thức xuất hiện ở nhiều phiên bản với nội dung văn bản sau khi dọn dẹp trùng khớp nhau, hệ thống cấp phát một `element_id` duy nhất trong bảng `elements`. Cấu trúc cây của từng phiên bản được lưu riêng trong bảng `hierarchy`.

Ví dụ dưới đây thể hiện một bài báo gồm hai phiên bản có câu `s1` giữ nguyên nội dung nhưng nằm ở hai tiêu đề khác nhau qua hai phiên bản:

Bảng `elements`: lưu tập hợp phần tử phân biệt.

| **element_id** | **kind** | **content** |
| :--- | :--- | :--- |
| `2501-02642#root1` | `document` | Phiên bản 1 gốc |
| `2501-02642#root2` | `document` | Phiên bản 2 gốc |
| `2501-02642#h1` | `heading` | Introduction |
| `2501-02642#h2` | `heading` | Background and Related Work |
| `2501-02642#s1` | `sentence` | We study the citation matching problem on arXiv. |

Bảng `hierarchy`: lưu các cạnh phân cấp theo phiên bản.

| **version** | **element_id** | **parent_id** |
| :--- | :--- | :--- |
| 1 | `2501-02642#h1` | `2501-02642#root1` |
| 1 | `2501-02642#s1` | `2501-02642#h1` |
| 2 | `2501-02642#h2` | `2501-02642#root2` |
| 2 | `2501-02642#s1` | `2501-02642#h2` |

Cách lưu này giúp câu `s1` chỉ chiếm một dòng trong `elements`, trong khi `hierarchy` vẫn ghi nhận ở phiên bản 1 nó trực thuộc mục `h1`, còn ở phiên bản 2 nó trực thuộc mục `h2`.
