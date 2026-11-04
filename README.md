# Báo cáo đồ án arXiv, Nhóm 10

*Học phần Nhập môn Khoa học Dữ liệu, Khoa Công nghệ Thông tin, Trường Đại học Khoa học Tự nhiên, ĐHQG-HCM*

Kho lưu trữ này chứa mã nguồn trang báo cáo khoa học của Nhóm 10, được biên dịch bằng công cụ Jupyter Book 2 và xuất bản tự động qua GitHub Pages.

## 1. Tổng quan đề tài

Đồ án tập trung nghiên cứu, thu thập và bóc tách cấu trúc dữ liệu từ kho lưu trữ mở arXiv, phục vụ bài toán đối sánh trích dẫn khoa học. Dự án gồm ba giai đoạn:

1. Giai đoạn 1, Thu thập dữ liệu: Thu thập mọi phiên bản công bố của các bài báo thuộc phạm vi được giao, tải mã nguồn LaTeX, lọc bỏ tệp ảnh để giảm dung lượng, và lấy siêu dữ liệu trích dẫn qua API của Semantic Scholar.
2. Giai đoạn 2, Phân tích cú pháp và chuẩn hóa: Lần theo đường biên dịch của tài liệu LaTeX đa tệp, chuyển văn bản thành cây phân cấp logic, chuẩn hóa công thức toán, chuyển `\bibitem` thành BibTeX, và khử trùng lặp nội dung qua các phiên bản.
3. Giai đoạn 3, Đối sánh trích dẫn: Đóng gói tập dữ liệu Parquet theo chuẩn Hugging Face Datasets, huấn luyện mô hình học máy để nối từng mục BibTeX với định danh arXiv tương ứng, đánh giá theo độ đo MRR@5, và trình bày poster khoa học.

Trang báo cáo này là sản phẩm chung cho hai giai đoạn đầu của đồ án. Toàn bộ bảng số liệu và biểu đồ trong báo cáo được sinh trực tiếp khi biên dịch từ sổ tay phân tích, không dùng ảnh chụp tĩnh.

## 2. Thành viên nhóm và phạm vi phân công

Nhóm 10 gồm bốn thành viên. Giảng viên phân công cho nhóm bốn khoảng arXiv ID liên tục, mỗi thành viên phụ trách 5.000 số hiệu, tương ứng 20.000 vị trí ID ứng viên:

| **Thành viên** | **MSSV** | **Khoảng arXiv ID** | **Số vị trí ID** | **Nhiệm vụ chính** |
| :--- | :--- | :--- | :--- | :--- |
| Trương Tuấn Anh | 24120260 | từ 2501.02642 đến 2501.07641 | 5.000 | [...] |
| Nguyễn Văn Toàn | 24120232 | từ 2502.08235 đến 2502.13234 | 5.000 | [...] |
| Lê Thành Đạt | 24120035 | từ 2507.16659 đến 2507.21658 | 5.000 | [...] |
| Trần Văn Tấn Khôi | 24120010 | từ 2508.07872 đến 2508.12871 | 5.000 | [...] |

Con số 20.000 là số vị trí cần kiểm tra trên arXiv, không phải số bài báo chắc chắn tồn tại. Cột nhiệm vụ chính sẽ được cập nhật cụ thể sau khi nhóm hoàn tất phân chia chi tiết các hạng mục công việc.

## 3. Cấu trúc kho lưu trữ

Kho lưu trữ được tổ chức tách bạch giữa mã nguồn báo cáo xuất bản và tài liệu hướng dẫn nội bộ:

- `report_book/`: Chứa toàn bộ mã nguồn cấu thành trang báo cáo Jupyter Book 2 theo yêu cầu của đề tài:
  - `myst.yml`: Cấu hình mục lục và giao diện báo cáo.
  - `index.md`: Trang giới thiệu tổng quan đề tài và phân công nhóm.
  - `m1/`: Tổng quan, phương pháp, sổ tay thống kê và đo hiệu năng của Giai đoạn 1.
  - `m2/`: Tổng quan, quy trình, sổ tay thống kê và đo hiệu năng của Giai đoạn 2.
  - `data/`: Các tệp số liệu thực nghiệm phục vụ sinh bảng và biểu đồ lúc biên dịch.
  - `references.bib`: Danh mục tài liệu tham khảo theo định dạng BibTeX.
  - `appendix.md`: Phụ lục kỹ thuật và hướng dẫn tái lập thực nghiệm.
  - `requirements.txt`: Danh sách thư viện Python phục vụ dựng trang báo cáo.
- `docs/`: Thư mục lưu trữ tài liệu hướng dẫn nội bộ:
  - `docs/huong_dan_bien_dich_bao_cao.md`: Hướng dẫn cài đặt môi trường, dựng trang HTML cục bộ và kiểm tra triển khai GitHub Pages.

## 4. Liên kết dự án

- Trang báo cáo trực tuyến trên GitHub Pages: https://toan-ten.github.io/Arxiv-Project-Group10-Report/
- Kho mã nguồn xử lý của nhóm: https://github.com/Toan-ten/Arxiv-Project-Group10
- Hướng dẫn biên dịch chi tiết: xem tệp [docs/huong_dan_bien_dich_bao_cao.md](docs/huong_dan_bien_dich_bao_cao.md)

## 5. Giấy phép

Dự án được phân phối dưới giấy phép MIT. Xem tệp LICENSE để biết thêm chi tiết.
