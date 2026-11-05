# Đồ án arXiv, Nhóm 10

*Báo cáo học phần Nhập môn Khoa học Dữ liệu, Khoa Công nghệ Thông tin, Trường Đại học Khoa học Tự nhiên, ĐHQG-HCM*

## 1. Bài toán và mục tiêu dự án

Kho lưu trữ mở arXiv chứa hơn hai triệu bài báo khoa học thuộc các ngành toán học, vật lý và khoa học máy tính. Đồ án này tiếp cận trực tiếp mã nguồn LaTeX của từng bài báo thay vì chỉ lấy tệp PDF.

Mục tiêu của đồ án gồm ba giai đoạn:

1. Giai đoạn 1, Thu thập dữ liệu: Thu thập mọi phiên bản công bố của các bài báo được giao, tải mã nguồn LaTeX, lọc bỏ tệp ảnh để giảm dung lượng, và lấy dữ liệu trích dẫn qua API của Semantic Scholar.
2. Giai đoạn 2, Phân tích cú pháp và chuẩn hóa: Lần theo đường biên dịch của tài liệu LaTeX đa tệp, chuyển văn bản thành cây phân cấp, chuẩn hóa biểu thức toán, chuyển `\bibitem` thành BibTeX, và khử trùng lặp qua các phiên bản.
3. Giai đoạn 3, Đối sánh trích dẫn: Xuất dữ liệu sang các bảng Parquet theo chuẩn Hugging Face Datasets, huấn luyện mô hình học máy để nối từng mục BibTeX với định danh arXiv tương ứng, đánh giá bằng độ đo MRR@5, và trình bày áp phích khổ A2.

Báo cáo này là sản phẩm chung cho hai giai đoạn đầu. Toàn bộ bảng và hình trong báo cáo sinh trực tiếp khi biên dịch từ sổ tính phân tích, không dùng ảnh chụp.

## 2. Sơ đồ quy trình xử lý tổng thể

Quy trình xử lý của đồ án gồm bốn bước nối tiếp nhau:

```{mermaid}
flowchart LR
    A["1. Thu thập Giai đoạn 1<br/>Khám phá ID và vN<br/>Tải tệp TeX và Bib<br/>Semantic Scholar"] --> B["2. Phân tích cú pháp Giai đoạn 2<br/>Tệp gốc document<br/>Đệ quy include<br/>Dựng cây phân cấp"]
    B --> C["3. Chuẩn hóa Giai đoạn 2<br/>Dọn dẹp định dạng<br/>Chuẩn hóa toán<br/>Khử trùng lặp"]
    C --> D["4. Đối sánh Giai đoạn 3<br/>Gán nhãn dữ liệu<br/>Trích chọn đặc trưng<br/>Đánh giá MRR@5"]
```

Dữ liệu đầu ra của Giai đoạn 1 là đầu vào cho bước phân tích cú pháp ở Giai đoạn 2. Dữ liệu sau khi chuẩn hóa và khử trùng lặp ở Giai đoạn 2 kết hợp với bảng tham chiếu ở Giai đoạn 1 để làm đầu vào cho mô hình học máy ở Giai đoạn 3.

## 3. Thành viên nhóm và phạm vi phân công

Nhóm 10 gồm bốn thành viên. Phạm vi nghiên cứu thực nghiệm của đề tài bao gồm bốn khoảng số hiệu arXiv liên tục, mỗi thành viên phụ trách theo dõi 5.000 số hiệu, tổng quy mô khảo sát đạt 20.000 vị trí ID ứng viên:

| **Thành viên** | **MSSV** | **Khoảng arXiv ID** | **Số vị trí ID** | **Nhiệm vụ chính** |
| :--- | :--- | :--- | :--- | :--- |
| Trương Tuấn Anh | 24120260 | từ 2501.02642 đến 2501.07641 | 5.000 | [...] |
| Nguyễn Văn Toàn | 24120232 | từ 2502.08235 đến 2502.13234 | 5.000 | [...] |
| Lê Thành Đạt | 24120035 | từ 2507.16659 đến 2507.21658 | 5.000 | [...] |
| Trần Văn Tấn Khôi | 24120010 | từ 2508.07872 đến 2508.12871 | 5.000 | [...] |

Con số 20.000 là số vị trí cần kiểm tra trên arXiv, không phân biệt chuyên ngành, và không phải số bài báo chắc chắn tồn tại. Trên thực tế, một số vị trí chưa được cấp phát hoặc bài báo đã bị rút. Mỗi bài báo tồn tại lại có thể có nhiều phiên bản. Do đó, số vị trí ID ứng viên, số bài báo hợp lệ và tổng số phiên bản là ba số đo khác nhau.

## 4. Mục lục báo cáo

Báo cáo gồm hai phần nội dung và một phụ lục:

Phần 1: Giai đoạn 1, Thu thập dữ liệu
- 01. Tổng quan và phạm vi thu thập: Quy định về 20.000 vị trí ID, cấu trúc thư mục làm việc, và lược đồ hai bảng `papers`, `references`.
- 02. Khám phá bài báo và danh sách phiên bản: So sánh các phương thức tiếp cận, bóc tách Atom XML qua thư viện chuẩn và cách xác định mọi phiên bản.
- 03. Thu thập và trích xuất mã nguồn LaTeX: Tải gói nguồn từ `export.arxiv.org`, nhận diện đa định dạng, lọc tệp ảnh và các lớp kiểm soát an toàn tệp.
- 04. Thu thập tài liệu tham khảo: Giao tiếp qua Semantic Scholar Graph API, lọc số hiệu arXiv và xử lý định danh cũ.
- 05. Khả năng chịu lỗi và kiên cố lưu trữ: Kiểm soát tốc độ bằng đồng hồ đơn điệu, giải thuật backoff hàm mũ kèm jitter, dừng khẩn cấp và mô hình điểm kiểm soát ba tầng.
- 06. Thống kê dữ liệu: Bảng và biểu đồ về phân bố phiên bản, độ phủ TeX, độ giảm dung lượng khi lọc ảnh, và tỷ lệ trích dẫn có arXiv ID.
- 07. Đánh giá hiệu năng: Thời gian chạy, bóc tách thời gian mạng so với tính toán CPU, phương pháp ghép nối nhiều phiên Colab, mức RAM và dung lượng đĩa.

Phần 2: Giai đoạn 2, Phân tích cú pháp và chuẩn hóa
- 01. Tổng quan và cấu trúc dữ liệu: Mục tiêu cấu trúc hóa, cơ chế kho lưu trữ chung, và lược đồ ba bảng `elements`, `hierarchy`, `bibtex`.
- 02. Quy trình phân tích cú pháp và khử trùng lặp: Thuật toán dò đường biên dịch đa tệp, dựng cây với ba dạng nút lá, chuẩn hóa toán, và khử trùng lặp hai cấp độ.
- 03. Thống kê kết quả phân tích cú pháp: Tỷ lệ bóc tách thành công, độ giảm phần tử sau khử trùng lặp, và phân loại lỗi cú pháp.
- 04. Đánh giá hiệu năng bộ phân tích cú pháp: Thời gian phân tích mỗi bài báo và mức dùng tài nguyên máy tính.

Phần cuối:
- Phụ lục: Hướng dẫn chạy lại thí nghiệm trên Colab CPU, cấu hình môi trường, và danh mục tài liệu tham khảo.

## 5. Liên kết tài nguyên dự án

- Kho mã nguồn: https://github.com/Toan-ten/Arxiv-Project-Group10
- Kho báo cáo: https://github.com/Toan-ten/Arxiv-Project-Group10-Report
- Đề bài gốc: https://hcmus-cs-ta.github.io/lab-book/intro-ds/statement/
