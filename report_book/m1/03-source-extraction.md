# Thu thập và trích xuất mã nguồn LaTeX

## 1. Máy chủ phân phối mã nguồn chuyên dụng

Sau khi xác định danh sách các phiên bản cần tải, hệ thống tiến hành thu thập gói mã nguồn cho từng phiên bản cụ thể.

Mã nguồn được tải thông qua máy chủ phân phối chuyên dụng của arXiv:
$$\text{https://export.arxiv.org/src/}\{arxiv\_id\}\text{v}\{N\}$$

Việc định tuyến trực tiếp đến máy chủ `export.arxiv.org` là yêu cầu kỹ thuật quan trọng. Nếu gửi yêu cầu tải tự động hàng loạt đến tên miền chính `arxiv.org`, cơ chế phát hiện máy chủ tự động sẽ kích hoạt mã phản hồi HTTP 403 nhằm ngăn chặn hành vi thu thập dữ liệu bất thường [@arxiv-mimetypes].

## 2. Nhận diện đa định dạng tệp nén nguồn

Mặc dù phần lớn bài báo trên arXiv được nộp dưới dạng gói nén tar nén gzip (`.tar.gz`), thực tế dữ liệu chứa nhiều định dạng khác nhau. Thay vì giả định một định dạng duy nhất, hệ thống quét 64 KiB đầu tiên của luồng byte để nhận diện định dạng chính xác:

1. Định dạng ZIP: Nhận diện qua chuỗi byte chữ ký `PK` ở đầu tệp, tiến hành giải nén thông qua thư viện `zipfile`.
2. Định dạng tar nén gzip: Nhận diện qua chữ ký chuẩn `ustar` hoặc mở thành công bằng thư viện `tarfile`.
3. Bản ghi chỉ có PDF hoặc PostScript: Nhận diện qua chuỗi ký tự `%PDF-` hoặc `%!PS-Adobe-`. Trường hợp này xảy ra với khoảng 3% bài báo nộp trực tiếp bằng PDF hoặc Word [@arxiv-html-paper, @arxiv-whytex]. Hệ thống trả về danh sách tệp rỗng và ghi nhận trạng thái `no_tex_source` vào tệp `manifest.json`.
4. Mã nguồn TeX đơn lẻ: Nhận diện khi luồng byte không chứa ký tự null và xuất hiện các lệnh mở đầu của LaTeX như `\documentclass`, `\documentstyle`, `\begin{document}` hoặc `\bye`. Tệp đơn này được tự động lưu thành tệp `main.tex`.

## 3. Lọc bỏ tệp hình ảnh khi giải nén

Một gói nguồn đầy đủ của arXiv thường chứa mã nguồn LaTeX, tệp định dạng (`.cls`, `.sty`), tệp trích dẫn (`.bib`), cùng rất nhiều tệp hình ảnh minh họa (`.png`, `.jpg`, `.pdf`, `.eps`). Các tệp hình ảnh chiếm phần lớn dung lượng lưu trữ trên đĩa.

Để tối ưu hóa không gian lưu trữ theo yêu cầu của đề tài:
- Lọc trực tiếp trên luồng đọc: Trình trích xuất đọc dữ liệu nén trong bộ nhớ đệm và chỉ ghi xuống đĩa các tệp thường có phần mở rộng `.tex` và `.bib`. Toàn bộ tệp ảnh và tệp trung gian (`.bbl`, `.aux`, `.log`) bị loại bỏ ngay trong luồng giải nén, không ghi tạm xuống đĩa.
- Bảo toàn cấu trúc mã nguồn: Các tệp `.tex` được giữ nguyên vẹn từng byte, không mã hóa lại và không chỉnh sửa nội dung. Các câu lệnh định nghĩa hình ảnh như môi trường `figure` và lệnh `\includegraphics` được bảo lưu nguyên vẹn để phục vụ việc nhận diện nút lá hình ảnh trên cây phân cấp ở Giai đoạn 2.
- Phân định ranh giới xử lý BibTeX giữa hai giai đoạn: Trong thực tế nộp bài trên arXiv, phần lớn các ấn phẩm không đính kèm tệp `.bib` độc lập mà nhúng trực tiếp danh mục trích dẫn trong môi trường `thebibliography` bằng các lệnh `\bibitem` nội tuyến. Ở Giai đoạn 1, hệ thống trích xuất tệp `.bib` nếu gói nén có sẵn, đồng thời bảo toàn trọn vẹn tệp `.tex`. Việc nhận diện cú pháp `\bibitem` và chuyển đổi sang các mục BibTeX chuẩn hóa được đảm nhiệm bởi đường ống xử lý của Giai đoạn 2 theo đúng yêu cầu đề bài.

## 4. Các lớp kiểm soát an toàn tệp nén

Nhằm đảm bảo an toàn cho môi trường máy chủ và hệ thống tệp cục bộ khi giải nén các tệp lưu trữ từ nguồn mở, hệ thống áp dụng các lớp kiểm tra an ninh nghiêm ngặt:

1. Chống tấn công duyệt đường dẫn: Từ chối giải nén mọi tệp có đường dẫn chứa dấu `..`, đường dẫn tuyệt đối hoặc các ký tự điều khiển bất thường.
2. Chặn liên kết nguy hiểm và tệp thiết bị: Loại bỏ toàn bộ liên kết tượng trưng, liên kết cứng và các tên tệp thiết bị đặc thù trên Windows như `CON`, `PRN`, `AUX`, `NUL`, `COM1` đến `COM9`, `LPT1` đến `LPT9`.
3. Giới hạn kích thước phòng chống bom nén:
   - Giới hạn kích thước tải về tối đa không vượt quá 100 MiB cho mỗi phiên bản.
   - Giới hạn tổng dung lượng sau giải nén không vượt quá 200 MiB.
   - Giới hạn tổng số lượng tệp trong một gói nén không vượt quá 10.000 tệp.

Nếu bất kỳ điều kiện an toàn nào bị vi phạm, hệ thống lập tức ngắt tiến trình giải nén của phiên bản đó, ghi nhận mã lỗi an ninh `unsafe_archive` hoặc `size_limit` vào tệp `manifest.json` và tiếp tục xử lý các ấn phẩm tiếp theo mà không gây treo tiến trình.
