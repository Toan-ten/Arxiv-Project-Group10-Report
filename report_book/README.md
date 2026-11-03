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
nhấn **Ctrl+C** để dừng server. Đây là bản xem cục bộ.

## Triển khai GitHub Pages

Workflow nằm tại [`.github/workflows/deploy-book.yml`](../.github/workflows/deploy-book.yml).
Workflow build khi push lên `main` có thay đổi trong `report_book/` hoặc chính workflow;
cũng có thể chạy thủ công bằng **Actions → Deploy Jupyter Book to GitHub Pages →
Run workflow**, chọn nhánh `main`.

CI dùng **Python 3.14.1, Node.js 22.17.0** và cài từ `requirements.txt`, giữ
**Jupyter Book 2.1.7**. Lệnh build là `python -m jupyter_book build --html --strict --ci`
với thư mục làm việc `report_book/`, `JB_ALLOW_NODEENV=no` và
`BASE_URL=/Arxiv-Project-Group10-Report`. `BASE_URL` là đường dẫn con của website,
không phải URL đầy đủ. Dependencies gián tiếp và theme chưa được khóa toàn bộ;
build CI đầu tiên vẫn cần được xác nhận trên GitHub.

Chỉ `report_book/_build/html/` được upload làm artifact `github-pages`.
Tài liệu nội bộ ở root (`AGENTS.md`, `PROJECT_BRIEF.md`, `M1_CHECKLIST.md`) nằm ngoài
nguồn Book và thư mục upload. Giữ tài liệu nội bộ ngoài `report_book/`, không thêm
chúng vào TOC, `include` hoặc tài nguyên website. Job build có quyền đọc source;
job deploy có `pages: write`, `id-token: write` và dùng environment `github-pages`.
Workflow dùng `GITHUB_TOKEN` do GitHub cấp, không cần tạo personal access token.

Các bước chủ repository thực hiện sau khi review thay đổi local:

1. Mở [Settings → Pages của repository](https://github.com/Toan-ten/Arxiv-Project-Group10-Report/settings/pages).
   Trong **Build and deployment → Source**, chọn **GitHub Actions**.
   Nếu giao diện hiển thị nút **Save**, lưu lựa chọn. Giữ **Custom domain** trống
   khi dùng URL mặc định dưới đây.
2. Tự commit và push các file đã review lên `main`. Workflow sẽ tự chạy;
   không cần tạo thêm workflow bằng trình chọn template trên GitHub.
3. Mở tab **Actions**, chọn **Deploy Jupyter Book to GitHub Pages** và kiểm tra
   cả hai job `build` và `deploy` thành công. Nếu lần push trước khi bật Pages
   đã thất bại ở deploy, bật Pages rồi dùng **Run workflow** trên `main` để chạy lại.
4. Mở URL được trả trong environment `github-pages` hoặc mục Pages:
   <https://toan-ten.github.io/Arxiv-Project-Group10-Report/>.
   Kiểm tra trang chủ, menu M1, CSS/JavaScript và truy cập trực tiếp rồi tải lại
   các trang M1 sau:

   - [M1 — Data Crawling](https://toan-ten.github.io/Arxiv-Project-Group10-Report/index-1/).
   - [Phương pháp](https://toan-ten.github.io/Arxiv-Project-Group10-Report/method/).
   - [Kết quả](https://toan-ten.github.io/Arxiv-Project-Group10-Report/results/).
   - [Benchmark](https://toan-ten.github.io/Arxiv-Project-Group10-Report/benchmark/).
5. Chỉ nộp URL sau khi kiểm tra truy cập công khai thành công; giữ nguyên URL
   khi cập nhật nội dung cho các milestone tiếp theo.

Các URL con trên được lấy từ bản HTML hiện có; kiểm tra lại sau lần build CI
đầu tiên. Khi dùng GitHub Actions làm Source, không cần nhánh `gh-pages`.
Chuẩn bị workflow local chưa triển khai website; build khung thành công cũng
chưa xác nhận đã đủ dữ liệu nghiệm thu M1.

Nguồn cấu hình: [GitHub Pages custom workflows](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages)
và [MyST GitHub Pages / BASE_URL](https://mystmd.org/guide/deployment-github-pages).

## Nguồn và kết quả build

- `myst.yml`: cấu hình và mục lục; `index.md`: trang chủ.
- `m1/`: phạm vi, phương pháp, kết quả và benchmark của Milestone 1.
- `m2/index.md`: trang chuẩn bị Milestone 2; `sources.md`: tài liệu tham khảo.
- `_build/`: HTML, assets và cache sinh ra khi build, được bỏ qua trong Git.

Khi bổ sung kết quả, Book cần đọc dữ liệu đã thu thập với nguồn gốc rõ ràng.
Build khung thành công chưa có nghĩa là báo cáo M1 đã hoàn tất.
