# M1 — Benchmark

:::{important}
**Chưa có dữ liệu benchmark.** Các chỉ số hiệu năng đang chờ số liệu từ lượt
chạy scraper trên Google Colab CPU-only.
:::

## Môi trường và phạm vi đo

Theo yêu cầu M1, benchmark cuối được thực hiện trên **Google Colab CPU-only**,
đo wall time end-to-end của toàn hệ thống. Mỗi phép đo cần được diễn giải trong
bối cảnh cấu hình runtime, phiên bản mã nguồn, tham số và phạm vi publication/version.

Điều kiện chạy cụ thể, mức độ xử lý song song và trạng thái cache/resume chưa
được trình bày do phần này chưa có dữ liệu thực nghiệm.

## Các chỉ số hiệu năng

- **Thời gian:** wall time toàn hệ thống, discovery, từng paper và trung bình
  mỗi paper; nêu đơn vị, điểm bắt đầu/kết thúc và cách xử lý lỗi/retry.
- **RAM:** peak và trung bình, tần suất lấy mẫu và phạm vi tiến trình. Nếu chạy
  nhiều process, phải đo phù hợp cho toàn hệ thống thay vì chỉ một process.
- **Disk:** mức sử dụng đỉnh và dung lượng output cuối; nêu đơn vị và phạm vi
  thư mục/file được tính.

Phương pháp đo và công cụ sử dụng sẽ được mô tả cùng số liệu để làm rõ ý nghĩa
và giới hạn của từng chỉ số.

## Diễn giải kết quả

Các bảng và biểu đồ hiệu năng sẽ được tính từ metrics của lượt chạy tương ứng,
kèm đơn vị và phạm vi đo. Hiện chưa có số liệu để so sánh hiệu năng hoặc kết
luận về tác động của các lựa chọn tối ưu hóa.

Xem [yêu cầu hiệu năng M1](../sources.md) trong tài liệu tham khảo.
