# Thu thập tài liệu tham khảo và siêu dữ liệu Semantic Scholar

## 1. Mục tiêu và kiến trúc hai giao diện chuyên biệt

Bên cạnh việc thu thập tệp nguồn LaTeX từ arXiv, đề tài yêu cầu thu thập danh mục tài liệu tham khảo có cấu trúc và bổ sung siêu dữ liệu học thuật (như nơi công bố `venue`) cho từng bài báo thông qua Semantic Scholar Graph API [@semantic-scholar-api, @semantic-scholar-platform]. Dữ liệu trích dẫn này là nguồn đầu vào trực tiếp để xây dựng bảng `references` làm tập ứng viên cho bài toán đối sánh trích dẫn tự động ở Giai đoạn 3.

Trong module `src.semantic_scholar`, nhóm thiết kế hai giao diện tách biệt dùng chung một bộ điều tiết tốc độ `RateLimiter` và tầng vận chuyển HTTP an toàn bằng thư viện chuẩn Python:
1. Truy vấn siêu dữ liệu theo lô (`fetch_metadata`): Thu thập thông tin bài báo theo từng lô lên tới 500 số hiệu mỗi yêu cầu qua điểm cuối `POST /graph/v1/paper/batch`.
2. Truy vấn danh mục trích dẫn có phân trang (`fetch_references`): Thu thập toàn bộ danh sách tài liệu tham khảo của từng bài báo qua điểm cuối `GET /graph/v1/paper/ARXIV:{arxiv_id}/references` và chuẩn hóa trực tiếp sang cấu trúc snapshot hợp đồng phiên bản 2 (`contract_version = 2`).

## 2. Truy vấn siêu dữ liệu theo lô và xác minh danh tính bài báo

Hàm `fetch_metadata(arxiv_ids, *, batch_size=100, timeout=30.0, max_attempts=3, limiter=None, max_response_bytes=10_000_000, require_arxiv_identity=True)` nhận danh sách các số hiệu arXiv gốc (hỗ trợ cả định dạng mới `YYMM.NNNNN` và định dạng lịch sử trước năm 2007 như `hep-th/9901001`) và chia thành các lô (mặc định 100, tối đa 500 số hiệu mỗi yêu cầu) gửi tới:
$$\text{POST https://api.semanticscholar.org/graph/v1/paper/batch}$$

Cơ chế kiểm tra tính toàn vẹn phản hồi được thiết lập nghiêm ngặt:
- Các trường yêu cầu được gửi qua tham số truy vấn phẳng duy nhất: `paperId, corpusId, externalIds, title, abstract, authors, year, venue, publicationVenue, publicationTypes, journal, citationCount, influentialCitationCount, referenceCount, fieldsOfStudy, s2FieldsOfStudy, openAccessPdf, url`.
- Trước khi ánh xạ kết quả, hệ thống kiểm tra mảng JSON trả về phải khớp chính xác số lượng phần tử đã gửi trong lô.
- Xác minh danh tính bắt buộc (`require_arxiv_identity = True`): Mặc định, mỗi bản ghi bài báo trả về phải có trường `externalIds.ArXiv` khớp với số hiệu gốc tại vị trí tương ứng. Nếu bất kỳ phần tử nào thiếu xác nhận hoặc lệch số hiệu, toàn bộ lô đó bị từ chối với mã lỗi `invalid_response / arxiv_identity_unverified` thay vì gán ghép mù quáng theo thứ tự vị trí. Chỉ phần tử `null` hợp lệ trong phản hồi HTTP 200 mới được ghi nhận là `not_found`.

## 3. Truy vấn tài liệu tham khảo phân trang và chuẩn hóa Snapshot v2

Để lấy đầy đủ danh sách trích dẫn cho các bài báo có hàng trăm tài liệu tham khảo, hàm `fetch_references(arxiv_id, *, fields=None, limit=100, max_pages=1000, timeout=30.0, max_attempts=3, limiter=None, max_response_bytes=10_000_000, resume_from=None)` truy vấn điểm cuối phân trang:
$$\text{GET https://api.semanticscholar.org/graph/v1/paper/ARXIV:}\{arxiv\_id\}\text{/references}$$

Đặc tả kỹ thuật của quy trình thu thập và chuẩn hóa:
- Tham số trường phẳng: Các trường truy vấn được gửi dưới dạng phẳng `paperId, corpusId, title, authors, year, venue, publicationVenue, publicationDate, externalIds, url, contexts, intents, isInfluential` (không gắn tiền tố `citedPaper.` vào chuỗi truy vấn để tránh lỗi HTTP 400). Trong dữ liệu phản hồi, thông tin bài báo được trích dẫn nằm tại `data[].citedPaper`, còn ngữ cảnh trích dẫn (`contexts`, `intents`, `isInfluential`) nằm ở cấp quan hệ.
- Kiểm soát phân trang chặt chẽ: Hệ thống bắt đầu từ `offset = 0` với kích thước trang mặc định `limit = 100` (tối đa 1.000). Hệ thống chỉ chuyển trang dựa trên trường `next` hợp lệ do máy chủ trả về (kiểm tra `next` phải bằng đúng `offset + len(data)`, tiến lên đơn điệu và không trùng lặp). Độ phủ `coverage.complete = True` chỉ được xác nhận khi trang cuối có `next` bằng `null` hoặc vắng mặt và tổng số phần tử thu được khớp với `reported_total` (nếu có).
- Quy tắc chuẩn hóa từng mục trích dẫn (`items`): Mỗi tài liệu tham khảo được chuẩn hóa về đúng 10 trường của hợp đồng lưu trữ v2 gồm `arxiv_id`, `title`, `authors`, `submission_date`, `publication_date`, `year`, `venue`, `s2_paper_id`, `missing_fields` và `raw`:
  - `arxiv_id`: Trích xuất từ `externalIds.ArXiv`, giữ nguyên định danh hợp lệ (bao gồm cả định danh cũ có dấu gạch chéo như `hep-th/9901001`). Nếu tài liệu tham khảo không có số hiệu arXiv, trường này nhận giá trị `null` nhưng bản ghi **vẫn được giữ lại đầy đủ** trong `references.json` (chỉ khi kết xuất bảng `references` cuối cùng cho Giai đoạn 3 mới lọc các dòng có `ref_arxiv_id != null`).
  - `s2_paper_id`: Lấy từ `citedPaper.paperId` (phân biệt rõ với `corpusId` chỉ lưu trong `raw`).
  - Phân biệt ngày công bố và ngày nộp: Trường `publication_date` chỉ nhận giá trị từ `publicationDate` chuẩn ISO hợp lệ (không tự biến năm `year` thành ngày tháng). Trường `submission_date` luôn được đặt là `null` vì điểm cuối Semantic Scholar không cung cấp bằng chứng về ngày nộp bản đầu tiên trên arXiv.
  - Toàn bộ bản ghi gốc từ API được bảo lưu nguyên vẹn trong trường `raw` để phục vụ kiểm toán.

## 4. Điều tiết tần suất, bảo mật khóa API và phục hồi dữ liệu

- Bộ điều tiết tốc độ dùng chung (`RateLimiter`): Cả hai hàm `fetch_metadata()` và `fetch_references()` đều đi qua cùng một khóa điều tiết `DEFAULT_LIMITER` dùng đồng hồ đơn điệu `time.monotonic()`. Khi gặp lỗi HTTP 429, 5xx hoặc lỗi kết nối, hệ thống thử lại hữu hạn với thời gian chờ lấy giá trị lớn nhất giữa công thức $\min(2^{\text{attempt}-1}, 60) + \text{uniform}(0, 1)$ giây và chỉ dẫn `Retry-After` của máy chủ. Nếu vẫn gặp HTTP 429 sau lần thử cuối, tiến trình dừng ngay lập tức các yêu cầu tiếp theo trong lời gọi và đánh dấu các phần tử còn lại là `not_attempted_after_rate_limit`.
- Bảo mật khóa API và giới hạn kích thước: Khóa truy cập được đọc từ biến môi trường `SEMANTIC_SCHOLAR_API_KEY` và gắn vào tiêu đề `x-api-key`. Hệ thống chặn mọi chuyển hướng HTTP (redirect) để không làm rò rỉ khóa sang máy chủ khác, không ghi khóa hoặc nội dung phản hồi vào nhật ký lỗi, từ chối trang dữ liệu nếu phát hiện phản hồi bất thường chứa chuỗi khóa (`response_contains_secret`), và giới hạn kích thước mỗi phản hồi tối đa 10 MB (`max_response_bytes = 10_000_000`).
- Phục hồi và đối chiếu Snapshot (`resume_from`): Khi truyền một snapshot dở dang (`partial` hoặc `failed`) vào tham số `resume_from`, hệ thống thực hiện đọc lại từ `offset = 0` và đối chiếu từng phần tử ở phần đầu (prefix) với dữ liệu đã lưu. Nếu dữ liệu khớp và đọc vượt qua mốc cũ, hệ thống trả về snapshot mới đầy đủ hơn; nếu phát hiện máy chủ thay đổi thứ tự hoặc rút ngắn dữ liệu gây mâu thuẫn prefix, hệ thống bảo toàn dữ liệu cũ và ghi nhận mã chẩn đoán vào trường `error` để kiểm toán.

## 5. Hiện trạng kiểm thử và lộ trình kết nối đường ống

Hiện tại, toàn bộ module `src.semantic_scholar` đã được hoàn thiện mã nguồn và vượt qua bộ kiểm thử đơn vị ngoại tuyến gồm hơn 1.500 dòng mã kiểm thử (`tests/test_semantic_scholar.py` và `tests/test_semantic_scholar_references.py`), bao phủ các kịch bản phân trang, lỗi mạng, giới hạn tần suất HTTP 429, kiểm tra lược đồ `CorpusStorage` và đối chiếu phục hồi snapshot. Bước nối trực tiếp module này vào tiến trình tự động của `src.pipeline` và chạy thu thập thực tế trên máy ảo Google Colab sẽ cung cấp các số đo thực nghiệm cho chương Thống kê dữ liệu.
