# Milestone 1 — Data Crawling

:::{warning}
Phần này trình bày mục tiêu và phạm vi M1. Kết quả thu thập và đánh giá hiệu năng
đang chờ dữ liệu thực nghiệm.
:::

## Mục tiêu và phạm vi

M1 hướng đến thu thập corpus và metadata làm đầu vào cho các milestone sau.
Phạm vi của Group 10 được phân công như sau:

- **Nguyễn Văn Toàn:** `2502.08235` đến `2502.13234`.
- **Trần Văn Tấn Khôi:** `2508.07872` đến `2508.12871`.
- **Lê Thành Đạt:** `2507.16659` đến `2507.21658`.
- **Trương Tuấn Anh:** `2501.02642` đến `2501.07641`.

Mỗi khoảng gồm **5.000 vị trí ID**, tính cả hai đầu; tổng cộng **20.000 vị trí ID
ứng viên**. Con số này không đồng nghĩa với 20.000 bài báo hợp lệ, cũng không
phản ánh số bài đã thu thập. Số publication hợp lệ và số version chỉ được xác
định sau khi đối soát dữ liệu.

Với mỗi publication hợp lệ, yêu cầu M1 là phát hiện và thu thập **mọi version**,
có hậu tố version kể cả `v1`, thay vì chỉ lấy phiên bản mới nhất.

## Dữ liệu đầu ra

M1 yêu cầu lưu nguồn `.tex` và `.bib` khi có, cùng metadata publication/version
và references. Nội dung và đường dẫn trong nguồn LaTeX cần được bảo toàn.
Version không có TeX vẫn có record và metadata phù hợp để được ghi nhận trong
thống kê. Dữ liệu làm việc giữ cả references thiếu arXiv ID và các trường còn thiếu.

Hai bảng `papers` và `references` cần giữ đủ thông tin để xuất ở cuối đồ án;
đề không bắt buộc tạo Parquet ngay trong M1. Chỉ references có arXiv ID được
đưa vào bảng `references` xuất bản cuối cùng. Thống kê corpus và benchmark
được trình bày cùng điều kiện đo và thông tin tái lập.

## Nội dung báo cáo M1

- [Phương pháp](method.md): discovery, xử lý nguồn, metadata và khả năng tái lập.
- [Kết quả](results.md): độ phủ và các đặc điểm của corpus; hiện đang chờ dữ liệu.
- [Benchmark](benchmark.md): thời gian, bộ nhớ và dung lượng lưu trữ; hiện đang chờ dữ liệu.

Yêu cầu chi tiết được dẫn trong [tài liệu tham khảo](../sources.md).
