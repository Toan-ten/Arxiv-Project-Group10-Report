# Thu thập tài liệu tham khảo qua Semantic Scholar API

## 1. Mục tiêu và vai trò đối với đề tài

Bên cạnh việc thu thập tệp nguồn mã LaTeX, đề tài yêu cầu thu thập danh mục tài liệu tham khảo có cấu trúc cho từng bài báo thông qua Semantic Scholar Graph API [@semantic-scholar-api, @semantic-scholar-platform].

Dữ liệu trích dẫn này đóng vai trò quyết định trong việc xây dựng tập ứng viên cho bài toán đối sánh tài liệu tham khảo tự động ở Giai đoạn 3. Việc thu thập qua API đồ thị học thuật giúp trích xuất trực tiếp các định danh chuẩn hóa như số hiệu arXiv hoặc mã định danh Semantic Scholar mà không cần giải mã cú pháp phức tạp trong phần thư mục của mã nguồn LaTeX.

## 2. Điểm cuối và cấu hình tham số trường phẳng

Semantic Scholar cung cấp điểm cuối tra cứu học thuật theo số hiệu arXiv:
$$\text{https://api.semanticscholar.org/graph/v1/paper/arXiv:}\{arxiv\_id\}$$

Khi gửi yêu cầu HTTP GET, hệ thống thiết lập chuỗi tham số truy vấn ở dạng danh sách trường phẳng:
`fields = "references.title,references.authors,references.year,references.publicationDate,references.externalIds"`

Quy tắc bắt buộc của Semantic Scholar là các trường dữ liệu phải được liệt kê ở dạng phẳng, phân tách bằng dấu phẩy. Nếu sử dụng cú pháp lồng nhau như `references.authors.name`, máy chủ sẽ từ chối xử lý và trả về mã lỗi HTTP 400.

Mỗi phần tử trong mảng `references` trả về chứa các trường thông tin cơ bản: tiêu đề bài báo, danh sách tác giả, năm công bố, ngày công bố và từ điển chứa các định danh bên ngoài `externalIds`.

## 3. Quy tắc bóc tách số hiệu arXiv và xử lý định danh cũ

Dữ liệu trả về từ Semantic Scholar bao gồm nhiều nguồn trích dẫn khác nhau như sách, bài báo tạp chí truyền thống và kỷ yếu hội thảo. Để chuẩn bị cho lược đồ dữ liệu của đề tài:

- Lọc khóa số hiệu arXiv: Bộ xử lý kiểm tra sự tồn tại của khóa `ArXiv` bên trong từ điển `externalIds`. Chỉ những tài liệu tham khảo có chứa số hiệu arXiv hợp lệ mới được đưa vào bảng dữ liệu xuất bản `references` ở Giai đoạn 3.
- Lưu trữ cục bộ toàn bộ dữ liệu: Toàn bộ danh mục trích dẫn thô nhận được từ API được lưu trữ bền vững trong tệp `references.json` tại thư mục của từng bài báo. Việc này phục vụ cho việc tính toán độ phủ trích dẫn và đối chiếu độc lập sau này.
- Xử lý định danh trước năm 2007: Với các bài báo xuất bản trên arXiv trước năm 2007, định danh sử dụng quy ước phân loại theo tên chuyên ngành như `hep-th/9901001` thay vì chuỗi số `YYMM.NNNNN`. Hệ thống nhận diện và bảo toàn nguyên vẹn chuỗi định danh đặc thù này.

## 4. Kiểm soát tần suất và độ trễ chỉ mục

Hệ thống tuân thủ các quy định vận hành của Semantic Scholar API:
- Phân tích giới hạn tần suất và bài toán thời gian: Điểm cuối công khai áp dụng giới hạn 1 yêu cầu mỗi giây và tối đa 100 yêu cầu trong mỗi cửa sổ 5 phút cho các truy vấn không kèm khóa xác thực. Với quy mô 20.000 bài báo, việc truy vấn không xác thực sẽ tiêu tốn tối thiểu 16,67 giờ, tương ứng 3,0 giây cho mỗi bài. Ngược lại, việc đăng ký khóa API miễn phí bảo đảm tốc độ ổn định 1 yêu cầu mỗi giây, giúp rút ngắn tổng thời gian thu thập xuống còn khoảng 5,56 giờ. Nhóm thiết kế đường ống ưu tiên nạp khóa API và điều phối gọi trích dẫn gối đầu trong khi giải nén tệp TeX nhằm tối ưu hóa tổng thời gian chạy. Khi vượt ngưỡng tần suất hoặc gặp phản hồi HTTP 429, hệ thống kích hoạt giải thuật Backoff hàm mũ tương tự như với arXiv.
- Độ trễ lập chỉ mục: Semantic Scholar có độ trễ cập nhật từ 1 đến 7 ngày đối với các ấn phẩm mới nộp trên arXiv. Do phạm vi của nhóm bao gồm các bài báo trong năm 2025, một số bài báo rất mới có thể chưa được lập chỉ mục danh mục trích dẫn. Khi gặp tình huống này, tệp `references.json` ghi nhận danh sách rỗng và đánh dấu trạng thái tương ứng.

## 5. Trạng thái khung mẫu và kế hoạch thực nghiệm

Trong giai đoạn xây dựng khung đề cương báo cáo, module thu thập tài liệu tham khảo được thiết kế tách biệt và hoàn thiện cấu trúc lược đồ dữ liệu. Toàn bộ các chỉ số thực nghiệm về độ phủ trích dẫn, tỷ lệ có số hiệu arXiv và thời gian phản hồi của API hiện được giữ ở trạng thái khung mẫu chờ kết quả, và sẽ được nạp tự động sau khi hoàn tất đợt thực thi toàn diện trên máy ảo Google Colab.
