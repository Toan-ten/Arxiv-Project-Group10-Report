# Đồ án arXiv — Group 10

*Báo cáo học phần Introduction to Data Science — HCMUS*

Nhóm 10 thực hiện đồ án xử lý dữ liệu bài báo khoa học từ arXiv. Đồ án gồm ba giai đoạn: thu thập dữ liệu, phân tích và chuẩn hóa nội dung, sau đó đối chiếu thông tin tài liệu tham khảo.

Báo cáo trình bày cách nhóm thực hiện từng giai đoạn, những vấn đề gặp phải trong quá trình triển khai và kết quả thực nghiệm tương ứng.

## Thành viên nhóm

- **Nguyễn Văn Toàn** — MSSV 24120232
- **Trần Văn Tấn Khôi** — MSSV 24120010
- **Lê Thành Đạt** — MSSV 24120035
- **Trương Tuấn Anh** — MSSV 24120260

## Tổng quan đồ án

Đồ án được chia thành ba milestone:

1. **Milestone 1 — Data Crawling:** Thu thập thông tin bài báo từ arXiv, các phiên bản được công bố, tệp nguồn TeX/BibTeX và dữ liệu tham khảo cần thiết.
2. **Milestone 2 — Parsing & Standardization:** Phân tích cấu trúc nội dung LaTeX/BibTeX, chuẩn hóa dữ liệu và xử lý sự khác biệt giữa các phiên bản.
3. **Milestone 3 — Reference Matching:** Xây dựng phương pháp đối chiếu các mục tài liệu tham khảo trong BibTeX với thông tin tham chiếu tương ứng.

Dữ liệu thu thập ở M1 là đầu vào cho quá trình xử lý ở M2 và đối chiếu tham khảo ở M3.

## Phạm vi thực hiện Milestone 1

Nhóm được phân công **bốn khoảng arXiv ID**, mỗi thành viên phụ trách một khoảng gồm 5.000 ID, tương ứng **20.000 ID ứng viên**.

Đây là số lượng ID cần kiểm tra, không phải số bài báo chắc chắn tồn tại. Số bài hợp lệ, số phiên bản và kết quả thu thập sẽ được thống kê từ dữ liệu thực tế.

Chi tiết các khoảng ID được trình bày tại [M1 — Mục tiêu và phạm vi](m1/index.md).

:::{admonition} Tiến độ hiện tại
:class: note

Nhóm đã xây dựng và kiểm thử các thành phần quản lý ID, tra cứu thông tin bài báo và phiên bản, tải tệp nguồn, lưu trữ dữ liệu và hỗ trợ tiếp tục công việc sau khi bị gián đoạn. Một số chức năng đã được kiểm tra với dữ liệu thật từ arXiv.

Quá trình thu thập trên toàn bộ phạm vi được giao vẫn đang được thực hiện. Thống kê corpus và kết quả benchmark cuối trên Google Colab sẽ được bổ sung sau khi có dữ liệu và log chạy thực tế.
:::

## Nội dung báo cáo

**Milestone 1 — Data Crawling**

- [Mục tiêu và phạm vi](m1/index.md)
- [Phương pháp thu thập dữ liệu](m1/method.md)
- [Kết quả thu thập](m1/results.md)
- [Đánh giá hiệu năng (Benchmark)](m1/benchmark.md)

**Milestone 2 — Parsing & Standardization**

- [Tổng quan Milestone 2](m2/index.md)

**Tài liệu liên quan**

- [Tài liệu tham khảo](sources.md)
- [Repository mã nguồn](https://github.com/Toan-ten/Arxiv-Project-Group10)
- [Repository báo cáo](https://github.com/Toan-ten/Arxiv-Project-Group10-Report)
