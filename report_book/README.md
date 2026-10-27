# Nguồn Jupyter Book 2 của Group 10

`report_book/` chứa mã nguồn website báo cáo đồ án arXiv của Group 10,
dùng Jupyter Book 2 và cấu hình `myst.yml`. Results và Benchmark đang chờ
dữ liệu thực nghiệm; Milestone 2 mới có trang chuẩn bị.

## Môi trường

Môi trường đã kiểm thử: **Python 3.14.1, Jupyter Book 2.1.7 và Node.js 22.17.0**.
Các phiên bản khác có thể hoạt động nhưng chưa được kiểm thử trong hướng dẫn này.

Cần có Git, Python và Node.js kèm npm trên máy. Jupyter Book 2 dùng Node.js
để chạy MyST và build website; venv Python không thay thế Node.js.
Nếu chưa có Node.js, cài từ [trang chính thức](https://nodejs.org/en/download),
rồi mở PowerShell mới để các lệnh được nhận diện trong `PATH`.

Kiểm tra trước khi bắt đầu:

```powershell
git --version
python --version
node --version
npm.cmd --version
```

## Clone và cài dependencies

Chạy trong PowerShell tại thư mục muốn lưu repository. Nếu đã clone,
bỏ qua hai lệnh đầu và mở terminal ở thư mục gốc repository.

```powershell
git clone https://github.com/Toan-ten/Arxiv-Project-Group10-Report.git
Set-Location .\Arxiv-Project-Group10-Report
python -m venv .venv
& .\.venv\Scripts\python.exe -m pip install -r .\report_book\requirements.txt
```

Các lệnh sau gọi trực tiếp Python trong `.venv`, không cần chạy `Activate.ps1`.
`requirements.txt` ghim Jupyter Book 2.1.7, chưa khóa toàn bộ dependencies gián tiếp.

## Build HTML

Tiếp tục từ thư mục gốc repository:

```powershell
$env:JB_ALLOW_NODEENV = "no"
& .\.venv\Scripts\python.exe -m jupyter_book --version
Set-Location .\report_book
& ..\.venv\Scripts\python.exe -m jupyter_book build --html --strict --ci
```

`JB_ALLOW_NODEENV=no` ngăn Jupyter Book tự cài Node.js khi thiếu runtime phù hợp.
Nếu lệnh báo lỗi Node.js, kiểm tra lại `node --version` và `PATH`.
Lần build đầu có thể cần mạng để tải theme và dependencies của website.

HTML được tạo trong `report_book/_build/html/`. Khung hiện tại gồm các trang
Markdown, không chạy crawler hoặc gọi API lấy dữ liệu.

## Xem website trên localhost

Sau khi build thành công, giữ terminal ở `report_book/` và chạy:

```powershell
& ..\.venv\Scripts\python.exe -B -m http.server 8000 --bind 127.0.0.1 --directory .\_build\html
```

Mở <http://127.0.0.1:8000/> bằng trình duyệt. Giữ terminal chạy trong lúc xem,
nhấn **Ctrl+C** để dừng server. Đây là bản xem cục bộ; GitHub Pages chưa được triển khai.

## Nguồn và kết quả build

- `myst.yml`: cấu hình và mục lục; `index.md`: trang chủ.
- `m1/`: phạm vi, phương pháp, kết quả và benchmark của Milestone 1.
- `m2/index.md`: trang chuẩn bị Milestone 2; `sources.md`: tài liệu tham khảo.
- `_build/`: HTML, assets và cache sinh ra khi build, được bỏ qua trong Git.

Khi bổ sung kết quả, Book cần đọc dữ liệu đã thu thập với nguồn gốc rõ ràng.
Build khung thành công chưa có nghĩa là báo cáo M1 đã hoàn tất.
