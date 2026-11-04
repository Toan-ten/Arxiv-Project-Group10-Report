# Phương pháp và cài đặt

## 1. Khám phá bài báo và phiên bản

Bước đầu tiên là kiểm tra trạng thái tồn tại của 20.000 vị trí ID và tìm danh sách mọi phiên bản của từng bài báo.

### 1.1. Lựa chọn công cụ và cách tìm phiên bản

Kho lưu trữ arXiv cung cấp ba cách tiếp cận danh mục: arXiv API, giao thức OAI-PMH và bản sao dữ liệu trên Kaggle [@kaggle-arxiv]. Bảng dưới đây tóm tắt đặc điểm của từng công cụ:

| **Công cụ** | **Định dạng dữ liệu** | **Ưu điểm** | **Hạn chế đối với đồ án** |
| :--- | :--- | :--- | :--- |
| arXiv API [@arxiv-api-manual] | HTTP REST, Atom XML | Tra cứu trực tiếp theo danh sách ID bằng tham số `id_list`, trả về thông tin bản mới nhất | Bị giới hạn tốc độ, tối thiểu 3 giây mỗi yêu cầu |
| OAI-PMH [@oai-pmh-arxiv] | XML Harvest Protocol | Phù hợp để thu hoạch hàng loạt theo ngày hoặc chuyên ngành, phân trang bằng `resumptionToken` | Không tra cứu trực tiếp theo danh sách ID rời rạc, token hết hạn sau 24 giờ [@oai-pmh-cutover] |
| Kaggle Dataset [@kaggle-arxiv] | Tệp nén JSON Lines | Đọc trực tiếp trên máy, không phụ thuộc mạng hay giới hạn tần suất | Dữ liệu tĩnh, không có các bài báo mới xuất bản trong năm 2025 |

Nhóm chọn arXiv API vì đề bài giao danh sách 20.000 vị trí ID cụ thể trong năm 2025.

Quy trình tìm mọi phiên bản dựa vào tham số `id_list`: khi gửi yêu cầu tra cứu một số hiệu gốc như `2501.02642`, API trả về định danh của phiên bản mới nhất, ví dụ `2501.02642v3`. Dựa vào số phiên bản này, bộ thu thập tạo danh sách các phiên bản cần tải gồm `v1`, `v2` và `v3`.

### 1.2. Giới hạn tần suất và thư viện sử dụng

Quy định của arXiv [@arxiv-api-tou] yêu cầu giữ khoảng nghỉ tối thiểu 3 giây giữa hai yêu cầu liên tiếp và chỉ dùng một kết nối tại một thời điểm. Bộ thu thập của nhóm cài đặt khoảng trễ này và gửi kèm chuỗi nhận diện người dùng.

Thư viện `arxiv.py` [@arxiv-py-github] từ phiên bản 4.0.0 đã bỏ các hàm tải nguồn trực tiếp và cú pháp `search.results()`. Nhóm dùng `arxiv.Client().results(search)` để đọc XML và tự viết hàm tải gói nén qua thư viện mạng tiêu chuẩn.

## 2. Tải mã nguồn LaTeX

Sau khi có danh sách phiên bản, bộ thu thập tải gói nguồn tương ứng cho từng phiên bản.

### 2.1. Máy chủ phân phối và định dạng tệp

Theo thống kê của arXiv [@arxiv-html-paper] [@arxiv-whytex], khoảng 97% bài báo có mã nguồn TeX, 3% còn lại nộp dưới dạng PDF hoặc Word.

Mã nguồn được tải qua máy chủ phân phối:
$$\text{https://export.arxiv.org/src/}\{arxiv\_id\}\text{v}\{N\}$$

Bộ thu thập gửi yêu cầu đến máy chủ `export.arxiv.org` thay vì trang chủ `arxiv.org`. Tên miền chính sẽ chặn yêu cầu hàng loạt bằng mã lỗi 403 [@arxiv-mimetypes].

### 2.2. Lọc tệp ảnh khi giải nén

Gói nguồn arXiv có định dạng tệp tar nén gzip, chứa tệp `.tex`, `.bib`, các tệp định kiểu và tệp ảnh. Tệp ảnh chiếm phần lớn dung lượng gói nén.

Để tiết kiệm đĩa theo yêu cầu của đề:
- Lọc khi giải nén: Trình giải nén đọc gói nén tar trong bộ nhớ đệm và chỉ ghi xuống đĩa các tệp `.tex` cùng `.bib`. Tệp ảnh bị bỏ qua ngay trong luồng đọc, không lưu xuống đĩa.
- Giữ nguyên văn bản LaTeX: Các lệnh `figure` và `\includegraphics` trong tệp `.tex` được giữ nguyên để Giai đoạn 2 nhận diện nút lá dạng hình ảnh trên cây phân cấp.

### 2.3. Xử lý bài báo đặc thù và cơ chế thử lại

Với các phiên bản không có mã TeX, bộ thu thập tạo một bản ghi rỗng và lưu thông tin vào tệp metadata, không xóa bài báo khỏi danh sách. Cách làm này giúp giữ đúng mẫu số khi tính tỉ lệ bao phủ.

Khi gặp mã lỗi 429 hoặc 503, bộ thu thập tăng thời gian chờ từ 5 giây, 15 giây lên 60 giây và thử lại tối đa ba lần. Tiến trình chạy được ghi vào tệp lưu trạng thái sau mỗi bài báo để tiếp tục khi bị gián đoạn.

## 3. Lấy dữ liệu trích dẫn qua Semantic Scholar API

Danh mục tài liệu tham khảo được lấy từ Semantic Scholar [@semantic-scholar-platform] để chuẩn bị cho Giai đoạn 3.

### 3.1. Điểm cuối và tham số truy vấn

Semantic Scholar cung cấp API đồ thị học thuật [@semantic-scholar-api] cho phép tra cứu bài báo qua tiền tố arXiv:
$$\text{https://api.semanticscholar.org/graph/v1/paper/arXiv:}\{arxiv\_id\}$$

Nhóm cấu hình tham số truy vấn ở dạng trường phẳng:
`fields = "references.title,references.authors,references.year,references.publicationDate,references.externalIds"`

Semantic Scholar yêu cầu các trường viết ở dạng phẳng. Các trường lồng nhau như `references.authors.name` sẽ bị máy chủ trả về mã lỗi 400.

### 3.2. Bóc tách arXiv ID và các trường hợp đặc thù

Trong dữ liệu trả về từ Semantic Scholar, mỗi tài liệu tham khảo nằm trong mảng `references`:
- Lọc khóa `ArXiv`: Bộ thu thập kiểm tra khóa `externalIds.ArXiv`. Chỉ các tài liệu tham khảo có arXiv ID mới đưa vào bảng `references` ở Giai đoạn 3. Các tài liệu trích dẫn sách hoặc tạp chí khác vẫn được lưu ở tệp cục bộ để tính độ phủ.
- Bài báo trước năm 2007: Với các bài báo cũ dùng định dạng theo tên chuyên ngành như `hep-th/9901001`, bộ thu thập giữ nguyên chuỗi định danh này.

### 3.3. Giới hạn tần suất và độ trễ lập chỉ mục

API của Semantic Scholar giới hạn 1 yêu cầu mỗi giây và tối đa 100 yêu cầu trong 5 phút khi không dùng khóa xác thực. Nhóm dùng API key miễn phí để giữ tốc độ 1 yêu cầu mỗi giây ổn định.

Semantic Scholar có độ trễ từ 1 đến 7 ngày để lập chỉ mục các bài báo mới [@sorc]. Đây là giới hạn dữ liệu từ phía dịch vụ ngoài và được ghi nhận trong phần phân tích kết quả.
