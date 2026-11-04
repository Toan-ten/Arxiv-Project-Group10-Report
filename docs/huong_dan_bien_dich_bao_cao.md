# Hướng dẫn biên dịch báo cáo

Tài liệu này hướng dẫn chi tiết quy trình cài đặt môi trường, biên dịch kiểm thử trang báo cáo Jupyter Book 2 và triển khai tự động lên GitHub Pages cho đồ án arXiv của Nhóm 10.

## 1. Môi trường kiểm thử

Môi trường kiểm thử gồm Python 3.10 hoặc 3.14, Jupyter Book 2.1.7 và Node.js 22.

Cần có Git, Python và Node.js kèm npm trên máy. Jupyter Book 2 dùng Node.js để chạy MyST và dựng trang website. Nếu chưa có Node.js, tải từ trang chính thức của Node.js.

Kiểm tra trước khi bắt đầu:

```bash
git --version
python --version
node --version
npm --version
```

## 2. Cài đặt thư viện

Mở terminal tại thư mục gốc kho mã nguồn và cài đặt thư viện vào môi trường ảo:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r report_book/requirements.txt
```

Tệp `report_book/requirements.txt` chứa Jupyter Book 2.1.7 cùng các thư viện pandas, matplotlib, seaborn và ipykernel để chạy các sổ tay phân tích khi biên dịch.

## 3. Biên dịch HTML cục bộ

Chuyển vào thư mục `report_book` và chạy lệnh dựng trang:

```bash
cd report_book
export JB_ALLOW_NODEENV=no
python -m jupyter_book build --html --strict --ci
```

HTML được tạo trong thư mục `report_book/_build/html/`.

Để xem website trên máy cục bộ, chạy lệnh:

```bash
python -m http.server 8000 --bind 127.0.0.1 --directory _build/html
```

Mở địa chỉ http://127.0.0.1:8000 trên trình duyệt để kiểm tra giao diện.

## 4. Triển khai lên GitHub Pages

Quy trình tự động hóa nằm tại `.github/workflows/deploy-book.yml`. Quy trình tự chạy khi đẩy nhánh `main` lên GitHub nếu có thay đổi trong thư mục `report_book/`.

Các bước thiết lập:
1. Mở mục Settings, vào Pages của kho lưu trữ trên GitHub.
2. Tại mục Build and deployment, chọn nguồn là GitHub Actions.
3. Đẩy các thay đổi lên nhánh `main`.
4. Mở thẻ Actions trên GitHub để xem tiến trình dựng và triển khai.
5. Truy cập địa chỉ website chính thức: https://toan-ten.github.io/Arxiv-Project-Group10-Report/

## 5. Cấu trúc thư mục báo cáo

- `myst.yml`: Cấu hình MyST và mục lục TOC của cuốn sách.
- `index.md`: Trang chủ giới thiệu bài toán, quy trình xử lý và phân công nhóm.
- `m1/`: Tổng quan, phương pháp, sổ tay thống kê và đo hiệu năng của Giai đoạn 1.
- `m2/`: Tổng quan, quy trình, sổ tay thống kê và đo hiệu năng của Giai đoạn 2.
- `data/`: Năm tệp số liệu tổng hợp dùng để vẽ bảng và biểu đồ lúc biên dịch.
- `references.bib`: Danh mục tài liệu tham khảo BibTeX.
- `appendix.md`: Hướng dẫn chạy lại quy trình xử lý và cấu hình môi trường.
