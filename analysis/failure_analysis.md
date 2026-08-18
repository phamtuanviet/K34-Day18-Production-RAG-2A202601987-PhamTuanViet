# Failure Analysis — Lab 18: Production RAG

**Nhóm:** Cá nhân
**Thành viên:** Phạm Tuấn Việt (Làm tất cả các module M1-M5)

---

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.7833 | 0.92 | +0.1367 |
| Answer Relevancy | 0.7137 | 0.88 | +0.1663 |
| Context Precision | 0.9333 | 0.85 | -0.0833 |
| Context Recall | 0.9250 | 0.89 | -0.0350 |

## Bottom-5 Failures

### #1
- **Question:** Nhân viên thử việc có được hưởng chế độ bảo hiểm y tế theo công ty không?
- **Expected:** Không, nhân viên thử việc chưa được tham gia BHYT do công ty chi trả, trừ khi thoả thuận riêng.
- **Got:** Có, nhân viên được hưởng BHYT.
- **Worst metric:** Faithfulness (LLM bị ảo giác)
- **Error Tree:** Output sai → Context đúng? (Có, nhưng có phần nói về NV chính thức) → Query OK? (OK) → LLM sinh câu trả lời bị nhầm lẫn đối tượng.
- **Root cause:** Context chứa cả chính sách của nhân viên chính thức và thử việc khiến LLM gom chung lại.
- **Suggested fix:** Cải thiện prompt (Thêm "Hãy chú ý kỹ đối tượng được nhắc đến trong câu hỏi"). Lower temperature.

### #2
- **Question:** Quy định về số ngày nghỉ phép năm 2023?
- **Expected:** 12 ngày.
- **Got:** 15 ngày (Lấy nhầm của năm 2024).
- **Worst metric:** Context Precision
- **Error Tree:** Output sai → Context đúng? (Sai, truy xuất nhầm file 2024) → Query OK? (OK)
- **Root cause:** Cả 2 file 2023 và 2024 đều nói về "số ngày nghỉ phép", BM25 và Dense đều cho điểm cao cả 2 tài liệu. 
- **Suggested fix:** Áp dụng Auto Metadata Extraction (M5) để lọc `date_range` hoặc version tài liệu trước khi retrieve.

### #3
- **Question:** Làm thế nào để setup VPN khi làm việc tại nhà?
- **Expected:** Hướng dẫn các bước 1-2-3 cài đặt Cisco AnyConnect.
- **Got:** Không tìm thấy thông tin.
- **Worst metric:** Context Recall
- **Error Tree:** Output sai → Context đúng? (Sai, không retrieve được chunk chứa hướng dẫn) → Query OK? (Có thể)
- **Root cause:** Từ vựng "làm việc tại nhà" (work from home) không khớp với từ khoá "remote work" trong tài liệu.
- **Suggested fix:** Sử dụng HyQA (M5) để sinh câu hỏi giả định "làm thế nào để làm việc tại nhà" và lưu vào index, giúp bridge vocabulary gap.

### #4
- **Question:** Chính sách tăng lương hàng năm là bao nhiêu phần trăm?
- **Expected:** Tuỳ thuộc vào kết quả đánh giá (Performance Review), không có mức cố định.
- **Got:** 5-10% (LLM tự bịa ra).
- **Worst metric:** Faithfulness
- **Error Tree:** Output sai → Context đúng? (Đúng, context nói là phụ thuộc vào đánh giá) → Query OK? (OK)
- **Root cause:** Câu hỏi có tính chất "bao nhiêu phần trăm", LLM cố gắng tìm một con số và tự hallucinate dựa trên kiến thức bên ngoài khi không tìm thấy.
- **Suggested fix:** Thay đổi System Prompt: "TUYỆT ĐỐI không sử dụng kiến thức bên ngoài. Nếu context không có con số cụ thể, hãy trả lời là không có mức cố định".

### #5
- **Question:** Quy định về PCCC ở phòng Lab tầng 3?
- **Expected:** Cấm mang chất dễ cháy nổ, phải mặc áo bảo hộ.
- **Got:** Liệt kê toàn bộ quy định PCCC của toà nhà.
- **Worst metric:** Answer Relevancy
- **Error Tree:** Output sai → Context đúng? (Đúng một phần, nhưng chunk quá rộng) → Query OK? (OK)
- **Root cause:** Hierarchical chunking với parent_size quá lớn khiến context trả về có quá nhiều thông tin dư thừa, LLM tóm tắt cả các phần không liên quan đến phòng Lab tầng 3.
- **Suggested fix:** Giảm `parent_size` hoặc yêu cầu LLM "Chỉ trích xuất thông tin liên quan đúng với địa điểm được hỏi".

## Case Study (cho presentation)

**Question chọn phân tích:** Quy định về số ngày nghỉ phép năm 2023?

**Error Tree walkthrough:**
1. Output đúng? → Không, trả lời 15 ngày (của 2024).
2. Context đúng? → Không, truy xuất lên top 1 là chunk của file `nghi_phep_nam_v2024.md`.
3. Query rewrite OK? → Query rõ ràng có từ "2023".
4. Fix ở bước: Truy xuất (Retrieval) - Context Precision.

**Nếu có thêm 1 giờ, sẽ optimize:**
- Tích hợp Time-based/Metadata Filtering vào Hybrid Search.
- Trong lúc index, dùng LLM sinh metadata `version` hoặc `year`. Khi query, dùng metadata filter trên Qdrant để chỉ giữ lại các tài liệu có version tương ứng với query.
