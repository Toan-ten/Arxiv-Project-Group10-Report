# Phụ lục kỹ thuật và hướng dẫn tái lập

## 1. Hướng dẫn chạy quy trình thu thập trên Google Colab

Toàn bộ quy trình thu thập dữ liệu được thiết kế để chạy trực tiếp trên máy ảo Google Colab ở chế độ CPU theo đúng quy định môi trường chuẩn của đề bài.

Các bước thực hiện trên một sổ tay Colab mới:

1. Chuyển môi trường sang CPU: Vào trình đơn Thời gian chạy, chọn Đổi loại thời gian chạy, chọn CPU.
2. Gắn kết Google Drive để lưu trữ dữ liệu bền vững:
```python
from google.colab import drive
drive.mount("/content/drive")
```
Thư mục lưu trữ dữ liệu `working_corpus/` và nhật ký `logs/` nên được đặt trong đường dẫn `/content/drive/MyDrive/arxiv_project/` để tránh mất mát khi phiên làm việc bị ngắt.

3. Tải mã nguồn từ kho GitHub chính thức và cài đặt môi trường:
```bash
git clone https://github.com/Toan-ten/Arxiv-Project-Group10.git
cd Arxiv-Project-Group10
```

4. Khởi chạy tiến trình thu thập cho toàn bộ 20.000 số hiệu:
```bash
python -m src.batch --config config/group10_id_ranges.json --corpus-root /content/drive/MyDrive/arxiv_project/corpus --log-root /content/drive/MyDrive/arxiv_project/logs --batch-id group10_m1
```

5. Phục hồi tiến trình khi phiên Colab bị ngắt quãng:
Khi phiên Colab hết hạn 12 giờ hoặc bị ngắt kết nối mạng, người dùng mở phiên mới, gắn lại Google Drive và chạy lệnh với tham số phục hồi:
```bash
python -m src.batch --config config/group10_id_ranges.json --corpus-root /content/drive/MyDrive/arxiv_project/corpus --log-root /content/drive/MyDrive/arxiv_project/logs --batch-id group10_m1 --resume
```
Hệ thống sẽ tự động quét các tệp `attempts/*.json` và `manifest.json`, bỏ qua những bài báo đã hoàn tất và tiếp tục xử lý các bài báo còn lại.

6. Kế hoạch chạy bộ phân tích cú pháp Giai đoạn 2:
Sau khi hoàn tất dữ liệu thô ở Giai đoạn 1, module phân tích cú pháp phân cấp của Giai đoạn 2, hiện đang trong quá trình phát triển, sẽ tiếp nhận thư mục dữ liệu `corpus/` làm đầu vào và kết xuất sang các bảng Parquet cấu trúc theo quy chuẩn đề bài.

## 2. Nạp dữ liệu tổng hợp vào kho báo cáo

Báo cáo học thuật không trực tiếp thực hiện việc cào dữ liệu qua mạng mà chỉ đọc các tệp chỉ số tổng hợp. Sau khi các đợt chạy trên Colab hoàn tất, các chỉ số đo lường từ thư mục nhật ký `logs/` và các tệp `manifest.json` được tổng hợp và ghi vào năm tệp số liệu chuẩn đặt tại thư mục `report_book/data/`:

Năm tệp số liệu tổng hợp gồm:
- `m1_summary.json`: Tổng hợp số lượng bài báo, phiên bản, độ phủ TeX và tỷ lệ thành công của Giai đoạn 1.
- `m1_distributions.csv`: Mẫu phân bố phiên bản, dung lượng và số lượng tài liệu tham khảo.
- `m1_benchmark.json`: Số đo tổng thời gian, RAM đỉnh, RAM lấy mẫu và dung lượng đĩa của Giai đoạn 1 tổng hợp qua các phiên Colab.
- `m2_summary.json`: Tỷ lệ phân tích cú pháp thành công và mức giảm dung lượng sau khử trùng lặp của Giai đoạn 2.
- `m2_benchmark.json`: Số đo thời gian và tài nguyên của Giai đoạn 2.

Sau khi cập nhật năm tệp trên, tiến hành biên dịch lại báo cáo:
```bash
cd Arxiv-Project-Group10-Report/report_book
python -m jupyter_book build --html --strict --ci
```

## 3. Cấu hình môi trường thử nghiệm chuẩn

Bảng dưới đây liệt kê thông số môi trường và phiên bản của các thư viện được sử dụng:

| **Công cụ hoặc thư viện** | **Phiên bản** | **Ghi chú** |
| :--- | :--- | :--- |
| Hệ điều hành | Ubuntu 22.04 LTS | Máy ảo Google Colab CPU |
| Python | 3.10.12 | Bản cài mặc định trên Colab |
| Jupyter Book | 2.1.7 | Trình biên dịch báo cáo MyST |
| pandas | 2.0.3 | Xử lý dữ liệu bảng và tính phân vị |
| pyarrow | 14.0.2 | Đọc và ghi tệp Parquet kèm nén Snappy |
| matplotlib | 3.8.0 | Vẽ biểu đồ trực quan trong báo cáo |
| seaborn | 0.13.0 | Định dạng nâng cao cho biểu đồ |
| psutil | 5.9.5 | Theo dõi mức dùng RAM của tiến trình nền |

## 4. Danh mục tài liệu tham khảo

```{bibliography}
:filter: docname in docnames
```
