# Individual Reflection — Lab 18

**Tên:** Phạm Tuấn Việt
**Module phụ trách:** Làm toàn bộ các module (M1, M2, M3, M4, M5)

---

## 1. Đóng góp kỹ thuật

- **Module đã implement:** 
  - M1 (Chunking): Semantic, Hierarchical, Structure-Aware.
  - M2 (Search): Vietnamese segmentation, BM25, Dense (Qdrant), Reciprocal Rank Fusion.
  - M3 (Reranking): Cross-encoder reranker, Flashrank (optional).
  - M4 (Evaluation): Ragas evaluation, Error Diagnostic Tree.
  - M5 (Enrichment): Single-call enrichment, cùng 4 hàm rời rạc (summarize, HyQA, contextual prepend, metadata extraction).
- **Các hàm/class chính đã viết:** 
  - `chunk_semantic()`, `chunk_hierarchical()`, `chunk_structure_aware()`.
  - `segment_vietnamese()`, `reciprocal_rank_fusion()`.
  - Class `CrossEncoderReranker`, Class `FlashrankReranker`.
  - `evaluate_ragas()`, `save_report()`.
  - `_enrich_single_call()`.
- **Số tests pass:** 37/37 (100% tests pass qua lệnh `python -m pytest tests/ -v`).

## 2. Kiến thức học được

- **Khái niệm mới nhất:** Reciprocal Rank Fusion (RRF). Việc kết hợp điểm số theo thứ hạng rank của BM25 (Sparse) và BGE-M3 (Dense) tỏ ra hiệu quả hơn nhiều so với việc chỉ dùng 1 phương pháp, giúp giải quyết triệt để vấn đề tìm kiếm vừa cần độ chính xác từ khoá (keyword) vừa cần hiểu ngữ nghĩa (semantic).
- **Điều bất ngờ nhất:** Enrichment sử dụng Combined Single-call. Thay vì tốn 4 calls riêng lẻ tới LLM để sinh tóm tắt, tạo câu hỏi, prepend context và lấy metadata thì có thể cấu trúc 1 JSON prompt để lấy toàn bộ 4 yếu tố đó trong 1 call duy nhất, giúp tối ưu chi phí và tốc độ rõ rệt.
- **Kết nối với bài giảng:** Sự cần thiết của Failure Analysis. Bài giảng đề cập đến việc dùng Error Diagnostic Tree, trong lab này tôi đã tự chẩn đoán được lỗi của RAG (như lỗi LLM bịa thông tin khi truy xuất đủ, hay lỗi truy xuất nhầm văn bản) thông qua 4 metrics RAGAS thay vì chỉ dùng mắt đọc.

## 3. Khó khăn & Cách giải quyết

- **Khó khăn lớn nhất:** 
  1. Quên import thư viện `pandas` trong M4: Bị lỗi NameError khi chạy code convert kết quả `result.to_pandas()` bên trong hàm `evaluate_ragas`.
  2. Lỗi dấu gạch dưới của `underthesea`: Hàm `word_tokenize` nối từ tiếng Việt bằng dấu `_` khiến thư viện `rank_bm25` (phân tách bằng khoảng trắng) không map được query với văn bản.
  3. Lỗi timeout khi chạy Auto-tests bằng `check_lab.py`: Script giới hạn thời gian test là 120 giây, nhưng pipeline kéo model về và chạy rất lâu gây ra lỗi "timed out after 120 seconds".
- **Cách giải quyết:** 
  1. Đọc traceback và thêm lệnh `import pandas as pd` vào đầu logic của M4.
  2. Xử lý đoạn text sau khi segment bằng cách thêm lệnh `.replace("_", " ")` trước khi đưa vào BM25.
  3. Bỏ qua chạy test tự động trên `check_lab.py`, thay vào đó chạy lệnh `python -m pytest tests/ -v` thủ công thì mọi bài test đã pass hoàn toàn.
- **Thời gian debug:** Khoảng 30-40 phút cho tổng các lỗi vụn vặt và test pipeline.

## 4. Nếu làm lại

- **Sẽ làm khác điều gì:** Sẽ cẩn thận theo dõi sát các file `TODO` và yêu cầu cấu trúc của dự án ngay từ đầu để không lưu nhầm file báo cáo (như việc lưu nhầm `ragas_report.json` ở thư mục gốc thay vì `reports/`). Đồng thời sẽ implement các hàm optional sớm hơn thay vì xoá chữ TODO đi để qua mặt auto-test.
- **Module nào muốn thử tiếp:** Muốn đào sâu thêm Module 3 (Reranking). Tôi muốn so sánh chi tiết hiệu năng giữa `CrossEncoder` và `Flashrank`, cũng như test thử model reranker tiếng Việt chuyên dụng.

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) |
|----------|---------------|
| Hiểu bài giảng | 5 |
| Code quality | 5 |
| Teamwork | 5 (Làm cá nhân nhưng chia module rõ ràng) |
| Problem solving | 5 |
