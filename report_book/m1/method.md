# M1 — Phương pháp thu thập

Phần này trình bày các nội dung phương pháp của M1. Chi tiết triển khai thực tế
sẽ được bổ sung cùng mã nguồn và nhật ký chạy; bản hiện tại chưa mô tả một
pipeline đã được kiểm chứng.

## Discovery và coverage theo version

Discovery có mục tiêu xác định publication hợp lệ trong bốn khoảng ID ở
[phạm vi M1](index.md), sau đó liệt kê mọi version, kể cả `v1`. Các khoảng ID
được xét cả hai đầu. Danh sách vị trí ứng viên được phân biệt với danh sách
publication hợp lệ và danh sách version cần thu thập.

Lựa chọn công cụ discovery và cách đối soát phiên bản chưa được trình bày
trong bản báo cáo hiện tại.

## Tải và xử lý nguồn

Theo yêu cầu M1, dữ liệu nguồn chỉ giữ các file `.tex` và `.bib` khi có. Việc
loại file phụ cần bảo toàn lệnh LaTeX và đường dẫn dùng trong các bước xử lý
sau, gồm `figure`/`includegraphics`. Không phải nguồn nào cũng có file `.bib`.

Trạng thái không có TeX hợp lệ được phân biệt với thất bại tải, chưa xử lý và
thiếu trường dữ liệu. Sự phân biệt này giúp diễn giải độ phủ và tỷ lệ thành
công mà không loại bỏ các version không có TeX khỏi thống kê.

## Metadata và references

M1 yêu cầu thu thập metadata paper và reference metadata qua Semantic Scholar.
Độ phủ metadata và mức độ liên hệ với từng version là những yếu tố cần xem xét
khi phân tích dữ liệu. Bản hiện tại chưa có kết quả thu thập metadata để đánh giá.

Dữ liệu làm việc cần giữ các references kể cả thiếu arXiv ID. Trường chưa biết
được giữ ở trạng thái thiếu để phản ánh đúng thông tin có trong nguồn.

## Khả năng phục hồi và tái lập

Retry, checkpoint và resume là các khía cạnh cần mô tả khi đánh giá khả năng
phục hồi của quá trình thu thập. Cách lưu corpus, metadata và trạng thái xử lý
cũng ảnh hưởng đến việc kiểm tra kết quả và tiếp tục một lượt chạy bị gián đoạn.
Các lựa chọn triển khai này chưa được báo cáo trong bản hiện tại.

Để tái lập, mỗi kết quả cần gắn với phiên bản mã nguồn, cấu hình, mã lượt chạy
và phạm vi publication/version tương ứng. Phần phân công ID được giữ cùng
dữ liệu để truy xuất phạm vi của từng thành viên.

Xem [yêu cầu M1](../sources.md) trong tài liệu tham khảo.
