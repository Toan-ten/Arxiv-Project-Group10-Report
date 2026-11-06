# Thu thập và trích xuất mã nguồn LaTeX

## 1. Đối sánh các phương thức tải nguồn và lựa chọn điểm cuối

Sau khi xác định danh sách các phiên bản từ `v1` đến `vN`, hệ thống cần tải gói mã nguồn của từng phiên bản. Đề bài giới thiệu ba phương thức tiếp cận nguồn tệp trên arXiv:

| **Phương thức** | **Giao diện truy cập** | **Công cụ / Điểm cuối** | **Đánh giá mức độ phù hợp với đồ án** |
| :--- | :--- | :--- | :--- |
| **Tải trực tiếp qua HTTP REST** [@arxiv-mimetypes] | Máy chủ phân phối chuyên dụng `export.arxiv.org` | `https://export.arxiv.org/src/{arxiv_id}v{N}` qua `urllib.request` | **Lựa chọn chính:** Hỗ trợ truy xuất chính xác từng phiên bản `v1..vN`, kiểm soát trực tiếp luồng byte, mã trạng thái HTTP và bộ điều tiết giãn cách 3,0 giây. |
| **Công cụ dòng lệnh** [@arxiv-downloader] | Tiện ích CLI đóng gói sẵn | `arxiv-downloader <id> -s -d ./downloads` | **Không sử dụng:** Gói dòng lệnh bên ngoài khó tích hợp kiểm soát lỗi chi tiết, không hỗ trợ lọc tệp ảnh ngay trong bước giải nén tạm và khó đồng bộ với điểm kiểm soát `CorpusStorage`. |
| **Kho lưu trữ đám mây hàng loạt** [@arxiv-s3-bulk] | Amazon S3 (`s3://arxiv/src/` trả phí theo yêu cầu) hoặc Google Cloud Storage (`gs://arxiv-dataset`) | `aws s3 cp` hoặc `s3cmd` theo các gói tháng `arXiv_src_YYMM_NNN.tar` | **Không phù hợp:** Dữ liệu đóng gói thành các tệp `.tar` dung lượng hàng TB chứa toàn bộ bài báo trong tháng kèm tệp ảnh, vượt quá giới hạn ổ đĩa 107,72 GB của máy ảo Google Colab. |

Vì vậy, module `src.arxiv_source` tải trực tiếp từng phiên bản qua điểm cuối chuyên dụng:
$$\text{https://export.arxiv.org/src/}\{arxiv\_id\}\text{v}\{N\}$$

Hệ thống chỉ gửi yêu cầu tới máy chủ `export.arxiv.org` kèm tiêu đề `User-Agent` định danh nhóm, tuyệt đối không gọi vào tên miền giao diện người dùng `arxiv.org` để tránh kích hoạt hệ thống chặn tự động (HTTP 403) [@arxiv-mimetypes].

## 2. Nhận diện đa định dạng tệp nén nguồn

Mặc dù phần lớn bài báo trên arXiv được nộp dưới dạng gói nén tar nén gzip (`.tar.gz`), thực tế dữ liệu chứa nhiều định dạng khác nhau. Thay vì giả định một định dạng duy nhất hoặc phụ thuộc vào tên tệp, hàm `extract_source()` kiểm tra nội dung byte thực tế (quét chữ ký tệp và 64 KiB đầu tiên) để nhận diện 5 dạng dữ liệu nguồn:

1. Định dạng tar hoặc tar nén gzip (`.tar`, `.tar.gz`): Nhận diện và mở qua thư viện `tarfile` sau khi kiểm tra chữ ký nén.
2. Định dạng ZIP: Nhận diện qua chữ ký `PK` và xác thực bằng `zipfile.is_zipfile()`.
3. Mã nguồn TeX đơn nén gzip (`.gz` không phải tar): Giải nén luồng gzip đơn và kiểm tra dấu hiệu mã nguồn LaTeX trong 64 KiB đầu tiên, sau đó lưu thành tệp `main.tex`.
4. Mã nguồn TeX đơn lẻ không nén: Nhận diện khi luồng byte không chứa ký tự null và xuất hiện các lệnh đặc trưng của LaTeX như `\documentclass`, `\documentstyle`, `\begin{document}`, `\section` hoặc `\bye`. Tệp đơn này được lưu thành `main.tex`.
5. Bản ghi chỉ có PDF hoặc PostScript: Nhận diện qua chuỗi chữ ký `%PDF-` hoặc `%!PS-Adobe-` (xảy ra với khoảng 3% bài báo nộp trực tiếp bằng PDF [@arxiv-html-paper; @arxiv-whytex]). Hệ thống trả về danh sách tệp rỗng và ghi nhận trạng thái `no_tex_source`. Các định dạng nhị phân hoặc văn bản khác chưa nhận diện được sẽ nhận trạng thái `unsupported_source` thay vì tự suy diễn là không có TeX.

## 3. Lọc bỏ tệp hình ảnh trong quá trình trích xuất

Một gói nguồn đầy đủ của arXiv thường chứa mã nguồn LaTeX, tệp định dạng (`.cls`, `.sty`), tệp trích dẫn (`.bib`), cùng rất nhiều tệp hình ảnh minh họa (`.png`, `.jpg`, `.pdf`, `.eps`). Các tệp hình ảnh chiếm phần lớn dung lượng lưu trữ trên đĩa.

Để tối ưu hóa không gian lưu trữ theo yêu cầu của đề tài:
- Trích xuất có chọn lọc qua thư mục tạm (Staging Extraction): Hàm `download_source()` tải gói dữ liệu về tệp tạm theo từng khối (chunk) kèm giới hạn kích thước. Sau đó, `extract_source()` duyệt từng phần tử trong gói nén mà không dùng lệnh giải nén hàng loạt `extractall()`. Chỉ các tệp thường có phần mở rộng `.tex` và `.bib` mới được trích xuất vào thư mục tạm kiểm định rồi di chuyển sang thư mục đích `vN/` của kho dữ liệu. Toàn bộ tệp ảnh và tệp trung gian (`.bbl`, `.aux`, `.log`, `.sty`, `.cls`) bị loại bỏ ngay khi duyệt danh mục nén và không bao giờ đi vào `working_corpus/`.
- Bảo toàn cấu trúc mã nguồn: Các tệp `.tex` và `.bib` được giữ nguyên vẹn từng byte và đường dẫn tương đối (relative paths), không làm phẳng cấu trúc thư mục con, không mã hóa lại và không chỉnh sửa nội dung. Các câu lệnh định nghĩa hình ảnh như môi trường `figure` và lệnh `\includegraphics` được bảo lưu nguyên vẹn để phục vụ việc nhận diện nút lá hình ảnh trên cây phân cấp ở Giai đoạn 2.
- Phân định ranh giới xử lý BibTeX giữa hai giai đoạn: Trên arXiv, nhiều bài báo không đính kèm tệp `.bib` rời mà viết trực tiếp mục tài liệu tham khảo bằng lệnh `\bibitem` trong môi trường `thebibliography` của tệp `.tex`. Ở Giai đoạn 1, hệ thống giữ nguyên các tệp `.bib` (nếu có) và toàn bộ tệp `.tex`. Sang Giai đoạn 2, bộ phân tích cú pháp sẽ bóc tách các lệnh `\bibitem` này và chuyển đổi sang định dạng BibTeX chuẩn hóa.

## 4. Các lớp kiểm soát an toàn tệp nén

Nhằm đảm bảo an toàn cho môi trường máy chủ và hệ thống tệp cục bộ khi giải nén các tệp lưu trữ từ nguồn mở, hệ thống áp dụng các lớp kiểm tra an ninh nghiêm ngặt:

1. Chống tấn công duyệt đường dẫn: Từ chối giải nén mọi tệp có đường dẫn chứa dấu `..`, đường dẫn tuyệt đối hoặc các ký tự điều khiển bất thường.
2. Chặn liên kết nguy hiểm và tệp thiết bị: Loại bỏ toàn bộ liên kết tượng trưng, liên kết cứng và các tên tệp thiết bị đặc thù trên Windows như `CON`, `PRN`, `AUX`, `NUL`, `COM1` đến `COM9`, `LPT1` đến `LPT9`.
3. Giới hạn kích thước phòng chống bom nén:
   - Giới hạn kích thước tải về tối đa không vượt quá 100 MiB cho mỗi phiên bản.
   - Giới hạn tổng dung lượng sau giải nén không vượt quá 200 MiB.
   - Giới hạn tổng số lượng tệp trong một gói nén không vượt quá 10.000 tệp.

Nếu bất kỳ điều kiện an toàn nào bị vi phạm, hệ thống lập tức ngắt tiến trình giải nén của phiên bản đó, ghi nhận mã lỗi an ninh `unsafe_archive` hoặc `size_limit` vào tệp `manifest.json` và tiếp tục xử lý các ấn phẩm tiếp theo mà không gây treo tiến trình.
