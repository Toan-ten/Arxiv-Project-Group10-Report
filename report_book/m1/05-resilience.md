# Khả năng chịu lỗi và tính kiên cố lưu trữ

## 1. Kiểm soát tốc độ truy cập bằng đồng hồ đơn điệu

Quy định sử dụng tài nguyên của arXiv [@arxiv-api-tou] yêu cầu duy trì khoảng nghỉ tối thiểu 3,0 giây giữa hai yêu cầu liên tiếp và chỉ thiết lập một kết nối duy nhất tại một thời điểm. Việc vi phạm quy định này sẽ dẫn đến việc máy chủ từ chối phục vụ và khóa địa chỉ IP truy cập.

Nhóm cài đặt cơ chế kiểm soát tốc độ nghiêm ngặt bằng thư viện chuẩn:
- Đồng hồ đơn điệu: Hệ thống sử dụng hàm `time.monotonic()` thay cho đồng hồ thời gian thực `time.time()`. Đồng hồ đơn điệu luôn tăng tuyến tính và không bị trôi lệch khi hệ điều hành đồng bộ thời gian mạng qua giao thức NTP, đảm bảo khoảng cách giữa các yêu cầu luôn chính xác.
- Khóa tiểu trình: Toàn bộ các yêu cầu gửi đến arXiv đều đi qua một khối khóa `threading.Lock()`. Mọi tiến trình hoặc tiểu trình muốn phát yêu cầu đều phải kiểm tra và cập nhật biến mốc thời gian `_next_request_at`.
- Chặn tự động chuyển hướng: Bộ điều phối sử dụng lớp `_NoRedirect` để vô hiệu hóa tính năng tự động chuyển hướng HTTP của thư viện mạng. Nếu máy chủ phản hồi mã chuyển hướng HTTP 301 hoặc 302, yêu cầu kế tiếp không được gửi đi tức thì mà phải trải qua đúng chu kỳ chờ 3,0 giây.

## 2. Giải thuật thử lại với Backoff hàm mũ và Jitter

Trong môi trường thu thập dữ liệu phân tán trên mạng Internet, các lỗi ngắt quãng đường truyền, độ trễ mạng và quá tải máy chủ là hiện tượng thường xuyên xảy ra.

Hệ thống phân loại và xử lý lỗi mạng theo quy trình:
- Nhận diện lỗi có thể thử lại: Bao gồm các lỗi hết thời gian chờ, lỗi mất kết nối tạm thời, mã lỗi quá tải HTTP 429 và các mã lỗi máy chủ nội bộ từ HTTP 500 đến 599.
- Giải thuật Backoff hàm mũ: Khi gặp lỗi có thể thử lại, thời gian chờ trước lần thử tiếp theo được tính toán theo công thức:
  $$\text{backoff} = \min\left(3,0 \times 2^{\min(\text{attempt}-1,\, 5)},\, 60,0\right) + \text{jitter}$$
  trong đó $\text{jitter}$ là một giá trị số thực ngẫu nhiên được lấy mẫu phân bố đều trong khoảng từ 0,0 đến 1,0 giây. Yếu tố ngẫu nhiên này giúp phân tán lưu lượng và ngăn chặn hiện tượng nhiều tiến trình cùng đồng loạt thử lại tại cùng một thời điểm.
- Ưu tiên chỉ dẫn tiêu đề Retry-After: Nếu phản hồi từ máy chủ gửi kèm tiêu đề `Retry-After`, hệ thống phân tích giá trị này (hỗ trợ cả định dạng số nguyên giây và định dạng ngày tháng chuẩn RFC 2822 HTTP-date). Thời gian chờ thực tế sẽ lấy giá trị lớn nhất giữa thời gian tính toán của thuật toán và yêu cầu từ máy chủ.

## 3. Chính sách dừng khẩn cấp bảo vệ địa chỉ IP

Nhằm bảo vệ an toàn cho dải địa chỉ IP của nhóm không bị đưa vào danh sách đen của máy chủ arXiv, hệ thống cài đặt chính sách ngắt khẩn cấp:

Khi máy chủ trả về mã lỗi HTTP 403 hoặc HTTP 429 ở lần thử cuối cùng, hệ thống không tiếp tục gửi yêu cầu cho các bài báo tiếp theo. Tiến trình lập tức kích hoạt lỗi dừng `PipelineStopped`, lưu lại toàn bộ tiến độ đã đạt được xuống đĩa và ngắt chương trình.

Chính sách này ngăn ngừa việc một lỗi cấu hình vô tình gửi hàng ngàn yêu cầu lỗi lên máy chủ trong thời gian ngắn. Khi muốn khởi động lại hệ thống sau khi kiểm tra nguyên nhân, người vận hành bắt buộc phải kích hoạt cờ xác nhận `resume=True` một cách chủ động.

## 4. Cơ chế ghi tệp an toàn bảo toàn tính toàn vẹn dữ liệu

Quá trình thu thập 20.000 bài báo kéo dài nhiều giờ và có nguy cơ bị gián đoạn do sự cố mất điện, hết hạn phiên tính toán hoặc ngắt tiến trình từ người dùng. Để bảo đảm dữ liệu trên đĩa không bị hư hỏng:

- Ghi tệp an toàn chống sự cố: Mọi tệp dữ liệu JSON gồm `manifest.json`, `metadata.json` và `references.json` đều được ghi bằng cơ chế ghi an toàn. Dữ liệu được ghi trước vào một tệp tạm mang đuôi `.tmp` nằm cùng thư mục, sau đó gọi phương thức `flush()` và lệnh đồng bộ phần cứng `os.fsync()`. Khi dữ liệu đã nằm an toàn trên phiến đĩa vật lý, hệ thống dùng hàm `os.replace()` để đổi tên tệp tạm đè lên tệp chính thức. Thao tác đổi tên này được hệ điều hành xử lý trọn vẹn trong một bước duy nhất, loại bỏ hoàn toàn nguy cơ tệp bị ghi dở dang khi xảy ra sự cố dừng tiến trình đột ngột.
- Khóa an toàn thư mục: Hệ thống từ chối ghi đè lên các thư mục phiên bản đã có sẵn tệp mã nguồn để bảo toàn tính toàn vẹn của dữ liệu đã tải trước đó.

## 5. Mô hình điểm phục hồi ba tầng và cơ chế cào nhiều lần

Trong kiến trúc thu thập dữ liệu phân tán, khả năng chịu lỗi và cơ chế thực thi nhiều đợt là yêu cầu kỹ thuật trọng tâm nhằm tối ưu hóa độ bao phủ của tập dữ liệu khi đối mặt với sự cố ngắt quãng mạng. Hệ thống thiết lập mô hình điểm phục hồi ba tầng:

1. Tầng đợt chạy: Quản lý qua tệp nhật ký `batch.json` và các tệp vết `attempts/<id>.json`. Mỗi lần thực thi được cấp một mã định danh duy nhất, ghi nhận chính xác danh sách các số hiệu đã hoàn thành và số hiệu còn tồn đọng.
2. Tầng bài báo: Quản lý qua tệp `manifest.json` trong từng thư mục ấn phẩm. Tệp này lưu trữ trạng thái tổng hợp của bài báo (`complete`, `complete_with_no_tex`, `partial`, `failed`).
3. Tầng phiên bản: Quản lý qua bảng mã băm SHA256 của từng tệp mã nguồn `.tex` và `.bib` đã giải nén. Khi phục hồi phiên chạy, hệ thống đối chiếu lại mã băm trên đĩa để xác minh dữ liệu không bị sửa đổi ngoài ý muốn.

Nhờ mô hình ba tầng này, người vận hành có thể chạy lại tiến trình thu thập nhiều lần cho cùng một thư mục đích. Hệ thống sẽ tự động bỏ qua các bài báo và phiên bản đã hoàn thành hợp lệ, chỉ tập trung xử lý các phiên bản bị lỗi mạng tạm thời trong các lần chạy trước, từ đó nâng cao tỷ lệ thành công của toàn bộ tập dữ liệu mà không làm phát sinh lưu lượng mạng dư thừa.
