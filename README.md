# Arxiv-Project-Group10-Report

Repository báo cáo **Jupyter Book 2 của Group 10** cho đồ án arXiv trong học phần
Introduction to Data Science. Mã nguồn website nằm trong [`report_book/`](report_book/).
Book gồm phần Milestone 1 và trang chuẩn bị Milestone 2; các mục Results và Benchmark
đang chờ dữ liệu thực nghiệm.

Để bắt đầu, đọc [hướng dẫn trong `report_book/`](report_book/README.md): clone repository,
tạo môi trường Python riêng, cài dependencies, build HTML và xem trên localhost.
Hướng dẫn sử dụng Windows PowerShell và ghi rõ các phiên bản đã kiểm thử.

- Chỉnh nội dung báo cáo trong các trang Markdown của `report_book/`.
- Cấu hình và thứ tự điều hướng nằm trong `report_book/myst.yml`.
- Mã nguồn xử lý dữ liệu nằm ở [repository code của nhóm](https://github.com/Toan-ten/Arxiv-Project-Group10).

Workflow [`deploy-book.yml`](.github/workflows/deploy-book.yml) đã được chuẩn bị để
build và triển khai Book lên GitHub Pages. Xem [hướng dẫn bật Pages và kiểm tra
website](report_book/README.md#triển-khai-github-pages). Website chỉ được xem là
hoạt động sau khi workflow chạy thành công và URL công khai được kiểm tra.
HTML sinh khi build được bỏ qua trong Git.
