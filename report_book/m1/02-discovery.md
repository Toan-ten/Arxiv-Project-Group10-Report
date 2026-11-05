# Khám phá bài báo và danh sách phiên bản

## 1. Khảo sát các phương thức tiếp cận danh mục arXiv

Bước đầu tiên trong quy trình xử lý dữ liệu là xác định trạng thái tồn tại của 20.000 vị trí số hiệu được giao và truy xuất danh sách mọi phiên bản công bố của từng bài báo.

Kho lưu trữ arXiv cung cấp ba phương thức tiếp cận danh mục dữ liệu chính:

| **Phương thức** | **Định dạng dữ liệu** | **Ưu điểm** | **Hạn chế đối với phạm vi đồ án** |
| :--- | :--- | :--- | :--- |
| arXiv API [@arxiv-api-manual] | HTTP REST, Atom XML | Tra cứu trực tiếp theo danh sách số hiệu cụ thể bằng tham số `id_list`, trả về thông tin phiên bản mới nhất | Giới hạn tần suất tối thiểu 3,0 giây cho mỗi yêu cầu |
| Giao thức OAI-PMH [@oai-pmh-arxiv] | XML Harvest Protocol | Phù hợp để thu hoạch hàng loạt theo khoảng ngày hoặc chuyên ngành, hỗ trợ phân trang tự động | Không hỗ trợ tra cứu trực tiếp theo danh sách số hiệu rời rạc, thẻ tiếp tục hết hạn sau 24 giờ [@oai-pmh-cutover] |
| Bản sao Kaggle [@kaggle-arxiv] | Tệp nén JSON Lines | Đọc trực tiếp trên ổ đĩa cục bộ, không phụ thuộc vào đường truyền mạng hay giới hạn tần suất | Dữ liệu đóng gói tĩnh, không có các bài báo mới xuất bản trong năm 2025 |

Dựa trên đặc thù của đề bài là xử lý danh sách 20.000 số hiệu cụ thể trong năm 2025, nhóm lựa chọn arXiv API làm phương thức thu thập chính.

## 2. Truy vấn trực tiếp qua arXiv API

Điểm cuối truy vấn chính thức của arXiv được triển khai tại địa chỉ:
$$\text{https://export.arxiv.org/api/query}$$

Hệ thống gửi yêu cầu HTTP GET kèm các tham số truy vấn:
- `id_list`: Truyền số hiệu gốc của bài báo cần tra cứu, ví dụ `2501.02642`.
- `start`: Đặt giá trị 0 để đọc từ kết quả đầu tiên.
- `max_results`: Đặt giá trị 1 do mỗi số hiệu đại diện cho một ấn phẩm duy nhất.

Máy chủ phản hồi dữ liệu theo định dạng Atom XML kèm tiêu đề `Content-Type: application/atom+xml`.

Về mặt kỹ thuật, việc truy vấn qua arXiv API có hai phương án tiếp cận:
1. Truy vấn đơn lẻ từng số hiệu: Mỗi yêu cầu chỉ kiểm tra đúng một ID với `max_results = 1`. Ưu điểm là cô lập hoàn toàn trạng thái và nhật ký lỗi của từng bài báo, nhưng thời gian chờ mạng thuần túy với quy mô 20.000 bài sẽ chiếm tối thiểu 16,67 giờ do bắt buộc giãn cách 3,0 giây giữa các lượt gọi.
2. Truy vấn theo cụm: Tham số `id_list` cho phép truyền danh sách nhiều số hiệu phân tách bằng dấu phẩy. Phương án này gom hàng chục số hiệu vào một lượt gọi duy nhất, giúp rút ngắn tổng thời gian khám phá toàn bộ 20.000 bài xuống dưới 30 phút mà vẫn bảo đảm khoảng nghỉ 3,0 giây hợp lệ giữa các yêu cầu. Bộ điều phối hỗ trợ cả hai chế độ tùy thuộc vào cấu hình thử nghiệm.

## 3. Tự xây dựng bộ bóc tách Atom XML qua thư viện chuẩn

Thay vì phụ thuộc vào thư viện bên ngoài `arxiv.py` vốn đã trải qua nhiều thay đổi cấu trúc mã nguồn, nhóm tự thiết kế module kết nối mạng và phân tích cú pháp bằng thư viện chuẩn của Python:

- Module `urllib.request` đảm nhiệm việc gửi yêu cầu HTTP và tiếp nhận luồng dữ liệu byte.
- Module `xml.etree.ElementTree` đảm nhiệm việc bóc tách cây XML với không gian tên chuẩn của Atom (`http://www.w3.org/2005/Atom`) và OpenSearch (`http://a9.com/-/spec/opensearch/1.1/`).

Giải pháp này loại bỏ nguy cơ xung đột phiên bản thư viện bên ngoài, giúp hệ thống vận hành ổn định và độc lập trên mọi môi trường thử nghiệm.

## 4. Xác định toàn bộ danh sách phiên bản

Khi gửi yêu cầu tra cứu một số hiệu gốc như `2501.02642`, máy chủ arXiv trả về thông tin của phiên bản mới nhất hiện có trong phần tử `<entry>`.

Quy trình xác định danh sách mọi phiên bản được thực hiện như sau:
1. Trích xuất định danh phiên bản mới nhất: Bộ phân tích đọc đường dẫn trong thẻ `<id>` hoặc thẻ liên kết `<link rel="alternate">` của phần tử `<entry>`. Đường dẫn này luôn kết thúc bằng số hiệu kèm hậu tố phiên bản, ví dụ `https://arxiv.org/abs/2501.02642v3`.
2. Phân tích biểu thức chính quy: Hệ thống dùng mẫu biểu thức chính quy để tách số hiệu gốc `2501.02642` và chỉ số phiên bản lớn nhất $N = 3$.
3. Khởi tạo danh sách phiên bản đầy đủ: Từ chỉ số lớn nhất $N$, hệ thống tự động suy diễn danh sách tuần tự gồm các phiên bản `v1`, `v2`, ..., `vN` cần tải về.

Với các bài báo chỉ công bố một lần duy nhất, định danh trả về mang hậu tố `v1` và hệ thống ghi nhận đúng một phiên bản duy nhất.

## 5. Xử lý các vị trí số hiệu không tồn tại

Trong tổng số 20.000 vị trí số hiệu được giao, không phải mọi vị trí đều tương ứng với một bài báo thực tế do việc nộp bài bị hủy hoặc số hiệu chưa được cấp.

Khi gặp một số hiệu không tồn tại, máy chủ arXiv phản hồi tài liệu Atom XML không chứa bất kỳ thẻ `<entry>` nào, hoặc trường tiêu đề thông báo lỗi. Bộ phân tích nhận diện trường hợp này và gán trạng thái `not_found`. Trạng thái này được ghi nhận trực tiếp vào tệp `manifest.json` của bài báo nhằm bảo toàn chính xác mẫu số 20.000 khi tính toán tỷ lệ thành công và tỷ lệ bao phủ của đề tài.
