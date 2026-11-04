# Phụ lục kỹ thuật và hướng dẫn tái lập

## 1. Hướng dẫn chạy lại quy trình xử lý

Toàn bộ quy trình xử lý chạy được trên Google Colab ở chế độ CPU, cũng là môi trường đo hiệu năng mà đề yêu cầu. Các bước dưới đây bắt đầu từ một sổ tay Colab mới. Vào trình đơn Thời gian chạy, chọn Đổi loại thời gian chạy, rồi chọn CPU.

Tải mã nguồn từ kho mã nguồn và cài các thư viện:

```bash
git clone https://github.com/Toan-ten/Arxiv-Project-Group10.git
cd Arxiv-Project-Group10
pip install -r requirements.txt
```

Chạy chương trình thu thập cho phạm vi 20.000 ID theo tệp cấu hình:

```bash
python -m src.crawler --config config/group10_id_ranges.json --output-dir ./working_corpus --log-interval 50
```

Chương trình thu thập giữ khoảng nghỉ tối thiểu 3 giây giữa hai yêu cầu, tải tệp nguồn từ `export.arxiv.org`, lọc bỏ tệp ảnh và chỉ giữ `.tex` cùng `.bib`, rồi lấy trích dẫn từ Semantic Scholar.

Chạy bộ phân tích cú pháp sau khi có dữ liệu thô:

```bash
python -m src.parser --input-dir ./working_corpus --output-dir ./processed_parquet
```

Đầu ra gồm ba bảng Parquet: `elements.parquet`, `hierarchy.parquet` và `bibtex.parquet`.

## 2. Chuyển số liệu sang kho báo cáo

Báo cáo không chạy lại quy trình thu thập và xử lý thô. Sau khi chương trình thu thập và bộ phân tích cú pháp chạy xong, lệnh dưới đây xuất năm tệp số liệu tổng hợp sang thư mục `report_book/data`:

```bash
python -m src.export_summary --corpus-dir ./working_corpus --parquet-dir ./processed_parquet --output-dir ../Arxiv-Project-Group10-Report/report_book/data
```

Năm tệp số liệu gồm:
- `m1_summary.json`: số lượng bài báo, phiên bản, độ phủ TeX và tỉ lệ thành công của Giai đoạn 1.
- `m1_distributions.csv`: mẫu phân bố phiên bản, dung lượng và số lượng trích dẫn.
- `m1_benchmark.json`: số đo thời gian, RAM đỉnh, RAM trung bình và dung lượng đĩa của Giai đoạn 1.
- `m2_summary.json`: tỉ lệ phân tích cú pháp thành công, mức giảm dung lượng sau khử trùng lặp và phân loại ngoại lệ của Giai đoạn 2.
- `m2_benchmark.json`: số đo thời gian và tài nguyên của Giai đoạn 2.

Sau khi cập nhật năm tệp trên, chạy lệnh biên dịch để cập nhật báo cáo:

```bash
cd Arxiv-Project-Group10-Report/report_book
python -m jupyter_book build --html --strict --ci
```

## 3. Cấu hình môi trường thử nghiệm

Bảng dưới đây liệt kê phiên bản của các công cụ và thư viện được dùng trên Colab:

| **Công cụ hoặc thư viện** | **Phiên bản trên Colab** | **Ghi chú** |
| :--- | :--- | :--- |
| Hệ điều hành | Ubuntu 22.04 LTS | Máy ảo Colab CPU |
| Python | 3.10.12 | Bản cài mặc định của Colab |
| Jupyter Book | 2.1.7 | Trình biên dịch báo cáo MyST |
| arxiv.py | 4.0.0 | Thư viện gọi arXiv API |
| requests | 2.31.0 | Gửi yêu cầu HTTP đến Semantic Scholar |
| pandas | 2.0.3 | Xử lý dữ liệu bảng và tính phân vị |
| pyarrow | 14.0.2 | Đọc và ghi tệp Parquet kèm nén Snappy |
| matplotlib | 3.8.0 | Vẽ biểu đồ cho báo cáo |
| seaborn | 0.13.0 | Định dạng biểu đồ |
| psutil | 5.9.5 | Lấy mẫu RAM theo chu kỳ |

## 4. Danh mục tài liệu tham khảo

```{bibliography}
:filter: docname in docnames
```
