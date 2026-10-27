# Milestone 2 — Parsing & Standardization

:::{warning}
**Chưa triển khai.** Phần này giới thiệu mục tiêu M2; chưa có kết quả parsing,
hierarchy, chuẩn hóa hoặc deduplication.
:::

## Mục tiêu theo đề

M2 sử dụng corpus của M1 để phân tích nội dung LaTeX theo compilation path:
từ root có `documentclass`, lần theo `input`/`include`, dựng hierarchy và chuẩn
hóa nội dung/toán. Công việc còn gồm chuyển `bibitem` sang BibTeX và xử lý
trùng lặp giữa các version.

Theo yêu cầu M2, quá trình xử lý cần giữ acknowledgements/appendices và tách
references khỏi hierarchy. Khi hợp nhất references trùng, các fields và cite keys
tương ứng cũng cần được cập nhật.

## Dữ liệu đầu vào và kết quả dự kiến

M2 phụ thuộc vào corpus và metadata M1 đã được đối soát. Đầu ra dự kiến gồm
các elements dùng chung, hierarchy theo version và BibTeX hợp nhất.

Các đầu ra trên là mục tiêu theo đề, chưa phải kết quả của nhóm. Báo cáo hiện
tập trung vào M1; phần phương pháp và đánh giá M2 sẽ dựa trên dữ liệu thực tế
khi giai đoạn này được triển khai.

Xem [yêu cầu M2](../sources.md) trong tài liệu tham khảo.
