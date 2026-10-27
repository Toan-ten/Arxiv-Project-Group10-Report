# Đồ án arXiv — Group 10

Đây là báo cáo đồ án arXiv của **Group 10** trong môn Introduction to Data Science
tại HCMUS. Nhóm gồm bốn thành viên:

- **Nguyễn Văn Toàn** — MSSV 24120232.
- **Trần Văn Tấn Khôi** — MSSV 24120010.
- **Lê Thành Đạt** — MSSV 24120035.
- **Trương Tuấn Anh** — MSSV 24120260.

Báo cáo tập trung vào **Milestone 1 — Data Crawling** và dành một phần cho
Milestone 2 — Parsing & Standardization.

:::{important}
Bản hiện tại là khung báo cáo. Phần kết quả thu thập dữ liệu và benchmark
đang chờ số liệu thực nghiệm.
:::

## Mục tiêu đồ án

Đồ án gồm ba giai đoạn có quan hệ phụ thuộc:

1. **M1 — Data Crawling:** thu thập các publication trong phạm vi được giao,
   mọi version, nguồn TeX/BibTeX và metadata cần thiết.
2. **M2 — Parsing & Standardization:** phân tích và chuẩn hóa nội dung LaTeX,
   BibTeX; xây dựng cấu trúc nội dung và xử lý trùng lặp giữa các version.
3. **M3 — Reference Matching:** xây dựng và đánh giá phương pháp ghép BibTeX
   với references của cùng publication.

Corpus của M1 là đầu vào cho quá trình xử lý ở M2 và ghép tham chiếu ở M3.

## Vai trò của báo cáo

Báo cáo trình bày phạm vi dữ liệu, phương pháp thu thập, thống kê corpus và
đánh giá hiệu năng. Các kết quả được gắn với dữ liệu đầu vào và điều kiện chạy
để có thể kiểm tra và tái lập.

- [Repository code](https://github.com/Toan-ten/Arxiv-Project-Group10): nơi phát triển mã nguồn qua các milestone.
- [Repository báo cáo](https://github.com/Toan-ten/Arxiv-Project-Group10-Report): nơi lưu nguồn Book.

## Đọc báo cáo

- [M1: mục tiêu và phạm vi](m1/index.md).
- [M1: phương pháp thu thập](m1/method.md).
- [Kết quả corpus: chưa có dữ liệu](m1/results.md).
- [Benchmark: chưa có dữ liệu](m1/benchmark.md).
- [M2: trang chuẩn bị, chưa triển khai](m2/index.md).
- [Tài liệu tham khảo](sources.md).

## Phạm vi báo cáo

Phạm vi M1 gồm bốn khoảng arXiv ID được phân công cho các thành viên, trình bày
tại [Mục tiêu và phạm vi](m1/index.md). Các khoảng này xác định vị trí ID ứng viên;
số bài báo hợp lệ và mức độ thu thập thành công sẽ được báo cáo từ dữ liệu thực tế.
M2 hiện mới có phần giới thiệu mục tiêu, chưa có kết quả triển khai.
