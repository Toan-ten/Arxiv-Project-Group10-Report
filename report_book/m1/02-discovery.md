# Khám phá bài báo và danh sách phiên bản

## 1. Đối sánh các phương thức tiếp cận danh mục arXiv

Bước đầu tiên của quy trình thu thập là kiểm tra sự tồn tại của 20.000 vị trí số hiệu được giao và xác định danh sách mọi phiên bản công bố của từng bài báo. Đề bài giới thiệu ba phương thức tiếp cận danh mục trên arXiv:

| **Phương thức** | **Giao diện và công cụ** | **Đặc điểm kỹ thuật** | **Đánh giá mức độ phù hợp với đồ án** |
| :--- | :--- | :--- | :--- |
| **arXiv API** [@arxiv-api-manual; @arxiv-api-basics] | HTTP/Atom XML (`http://export.arxiv.org/api/query`), thư viện `arxiv.py` hoặc HTTP client chuẩn | Hỗ trợ truy vấn theo từ khóa (`search_query`), phân trang (`start`, `max_results`), sắp xếp (`sortBy`, `sortOrder`) và tra cứu trực tiếp theo số hiệu qua `id_list`. Trả về hậu tố phiên bản mới nhất (ví dụ `v4`). | **Phù hợp nhất:** Tra cứu chính xác từng số hiệu được giao và cho biết ngay tổng số phiên bản hiện có của bài báo. |
| **Giao thức OAI-PMH** [@oai-pmh-arxiv; @oai-pmh-cutover] | XML Metadata Harvesting (`https://oaipmh.arxiv.org/oai` cập nhật từ 03/2025, thay cho `export.arxiv.org/oai2`), thư viện `Sickle` | Sử dụng các lệnh `Identify`, `ListRecords`, `ListIdentifiers` để thu hoạch hàng loạt theo khoảng ngày (`from`, `until`) và chuyên ngành (`set`). Từ bản cập nhật 03/2025, thẻ phân trang `resumptionToken` không còn chứa kích thước tập hợp và hết hạn sau 24 giờ. | **Không phù hợp cho tra cứu điểm:** Phù hợp để quét toàn kho theo thời gian, không hỗ trợ truy vấn trực tiếp danh sách 20.000 số hiệu chỉ định. |
| **Tập dữ liệu Kaggle** [@kaggle-arxiv] | Tệp tĩnh JSON Lines (`Cornell-University/arxiv`), đọc bằng `pandas` | Cung cấp siêu dữ liệu đóng gói sẵn của hơn hai triệu bài báo, xử lý trực tiếp trên đĩa cục bộ không tốn yêu cầu mạng. | **Phụ trợ đối chiếu:** Phụ thuộc vào thời điểm cập nhật bản chụp (snapshot), không đảm bảo phản ánh tức thời trạng thái phiên bản mới nhất và không thay thế bước tải nguồn theo phiên bản. |

Từ đối sánh trên, nhóm sử dụng trực tiếp **arXiv API** để phân giải từng số hiệu ứng viên trong bốn khoảng được giao.

## 2. Truy vấn đơn lẻ qua arXiv API

Hệ thống gửi yêu cầu HTTP GET tới điểm cuối truy vấn:
$$\text{https://export.arxiv.org/api/query}$$

Các tham số truyền vào gồm:
- `id_list`: Số hiệu gốc cần tra cứu, ví dụ `2501.02642`.
- `start = 0` và `max_results = 1`: Giới hạn đọc đúng bản ghi của số hiệu đang xét.

Máy chủ phản hồi tài liệu Atom XML kèm tiêu đề `Content-Type: application/atom+xml`. Dù tham số `id_list` hỗ trợ truyền nhiều số hiệu cách nhau bằng dấu phẩy, nhóm chọn phương án **mỗi yêu cầu chỉ tra cứu đúng một số hiệu (`max_results = 1`)** vì ba lý do kỹ thuật:
1. Cô lập trạng thái: Kết quả khám phá của từng số hiệu được ghi độc lập vào `manifest.json` tương ứng, không bị ảnh hưởng nếu một số hiệu khác trong lô gặp lỗi.
2. Kiểm tra chặt chẽ phản hồi: Bộ phân tích kiểm tra số lượng thẻ `<entry>`. Nếu phản hồi chứa nhiều hơn một `<entry>` hoặc `totalResults` mâu thuẫn, hệ thống từ chối bản ghi để tránh ghép nhầm siêu dữ liệu giữa các bài báo.
3. Khớp với cơ chế phục hồi từng bài: Tiến trình có thể dừng hoặc tiếp tục tại bất kỳ bài báo nào mà không phải chạy lại cả lô.

## 3. Xây dựng bộ phân tích Atom XML bằng thư viện chuẩn

Thư viện bên thứ ba `arxiv.py` [@arxiv-py-github] đã trải qua các thay đổi không tương thích ngược từ phiên bản 2.x lên 4.0.0 (tháng 05/2026), loại bỏ các phương thức cũ như `search.results()`, `Result.download_pdf()` và `Result.download_source()`. Để không phụ thuộc vào thay đổi của thư viện ngoài, nhóm xây dựng trực tiếp module `src.arxiv_client` và `src.arxiv_discovery` bằng thư viện chuẩn Python:

- Module `urllib.request` gửi yêu cầu HTTP kèm định danh `User-Agent` của Nhóm 10 và nhận luồng dữ liệu byte.
- Module `xml.etree.ElementTree` phân tích cây XML theo không gian tên chuẩn của Atom (`http://www.w3.org/2005/Atom`), OpenSearch (`http://a9.com/-/spec/opensearch/1.1/`) và arXiv (`http://arxiv.org/schemas/atom`).

## 4. Xác định danh sách phiên bản từ hậu tố mới nhất

Khi tra cứu một số hiệu gốc như `2501.02642`, arXiv API trả về thông tin của phiên bản mới nhất trong thẻ `<entry>`. Quy trình suy diễn danh sách phiên bản trong `parse_arxiv_response()` gồm ba bước:
1. Đọc định danh phiên bản mới nhất từ thẻ `<id>` hoặc `<link rel="alternate">` (ví dụ `http://arxiv.org/abs/2501.02642v3`).
2. Dùng biểu thức chính quy tách số hiệu gốc `2501.02642` (kiểm tra khớp với số hiệu đã hỏi) và lấy số thứ tự phiên bản lớn nhất $N = 3$.
3. Sinh danh sách đầy đủ các phiên bản từ `v1`, `v2` đến `vN` với trạng thái `version_status = "resolved"`. Nếu bài báo chỉ nộp một lần, hậu tố trả về là `v1` và danh sách gồm đúng một phiên bản `2501.02642v1`. Nếu phản hồi không có hậu tố phiên bản rõ ràng, hệ thống đánh dấu `version_status = "needs_verification"` thay vì tự đoán là `v1`.

## 5. Xử lý các vị trí số hiệu không tồn tại

Trong 20.000 vị trí số hiệu được giao, các số hiệu vượt quá tổng số bài nộp thực tế của tháng hoặc bị hủy trước khi công bố sẽ không tồn tại trên hệ thống.

Khi tra cứu một số hiệu không tồn tại, arXiv API trả về tài liệu Atom XML hợp lệ (HTTP 200) nhưng không chứa thẻ `<entry>` nào. Hàm `parse_arxiv_response()` nhận diện trường hợp này và trả về trạng thái `not_found`. Lưu ý rằng lỗi HTTP 404 hoặc lỗi đứt kết nối mạng được phân loại riêng thành `http_error` / `connection_error`, không bị gộp nhầm thành `not_found`. Trạng thái `not_found` được lưu vào `manifest.json` để giữ nguyên mẫu số 20.000 vị trí khi thống kê toàn cục.
