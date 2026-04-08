# SPEC draft - NhomXX-Room

## Track: VinFast

## Problem statement
Khách hàng đang cần mua xe VinFast hoặc đặt lịch bảo dưỡng thường phải tìm thông tin qua nhiều kênh (website, showroom, hotline, cộng đồng), mất thời gian và dễ rơi bỏ giữa chừng. AI chatbot có thể tư vấn xe phù hợp nhu cầu, hỗ trợ đặt hẹn bảo dưỡng, đồng thời phân tích review để cải thiện chất lượng vận hành.

---

## 1) Canvas

|   | Value | Trust | Feasibility |
|---|-------|-------|-------------|
| **Trả lời** | **User chính:** (1) người mua xe lần đầu, (2) chủ xe cần đặt lịch bảo dưỡng, (3) team CSKH/marketing VinFast. **Pain:** thông tin phân tán, phản hồi chậm, khó so sánh dòng xe, khó tổng hợp review thủ công. **AI value:** chatbot trả lời nhanh 24/7, gợi ý xe theo nhu cầu/ngân sách, hướng dẫn bảo dưỡng và đặt lịch; dashboard tóm tắt xu hướng review để team xử lý vấn đề sớm. | Sai gợi ý có thể gây mua nhầm, đặt lịch sai nhu cầu, hoặc tư vấn kỹ thuật sai. Cần hiển thị rõ mức độ tự tin, trích nguồn thông tin, và luôn có fallback "nói chuyện với tư vấn viên". User sửa được câu trả lời/sửa lịch trong 1-2 bước. | MVP sử dụng RAG từ knowledge base (catalog xe, chính sách bảo hành, bảng giá, FAQ) + model phân loại sentiment review. Chi phí ước tính API và hạ tầng trong ngưỡng pilot; latency mục tiêu < 4 giây/câu hỏi. Rủi ro: dữ liệu chưa cập nhật, hallucination, review bias. |

**Automation hay augmentation?** Augmentation.

**Justify:** Quyết định mua xe và xử lý kỹ thuật bảo dưỡng có tác động lớn, AI nên đề xuất và giải thích, còn user/nhân viên ra quyết định cuối.

### Learning signal

| # | Câu hỏi | Trả lời |
|---|---------|---------|
| 1 | User correction đi vào đâu? | Log reject/sửa gợi ý xe, sửa lịch hẹn, escalation sang người thật; đưa vào kho huấn luyện prompt + cập nhật FAQ/RAG chunk. |
| 2 | Product thu signal gì để biết tốt lên hay tệ đi? | Tỷ lệ câu hỏi giải quyết trong 1 lần, tỷ lệ user chấp nhận gợi ý, tỷ lệ chuyển sang human handoff, CSAT sau hỏi đáp/đặt lịch. |
| 3 | Data thuộc loại nào? | User-specific (lịch sử xe, lịch hẹn), domain-specific (catalog/chính sách), real-time (slot bảo dưỡng), human-judgment (nhân viên gán nhãn root cause review). |

**Marginal value:** Dữ liệu correction theo bối cảnh xe điện tại Việt Nam (dòng xe, trạm sạc, vấn đề sau bán) có giá trị cao, đối thủ không dễ có dữ liệu nội bộ giống hệt.

---

## 2) User stories - 4 paths

### Path 1 - AI đúng (happy path)
- User: "Tôi cần xe đi làm 40km/ngày, ngân sách 700 triệu."
- Bot gợi ý 2-3 mẫu phù hợp, so sánh tổng chi phí sở hữu, đề xuất test drive.
- User bấm "Đặt lịch lái thử" và hoàn tất trong cùng màn hình chat.

### Path 2 - AI không chắc (low confidence)
- User hỏi trường hợp không rõ: "Xe tôi nghe tiếng lạ ở tốc độ cao."
- Bot hiển thị: "Tôi chưa đủ tự tin, đề xuất 2 hướng kiểm tra ban đầu" + nút "Nói với cố vấn kỹ thuật".
- UI bắt buộc show disclaimer không thay thế chẩn đoán kỹ thuật tại xưởng.

### Path 3 - AI sai (failure path)
- Bot đề xuất lịch bảo dưỡng không đúng mốc km/năm sử dụng.
- User bấm "Câu trả lời này chưa đúng", chọn lý do, bot đưa phương án sửa ngay.
- Hệ thống ghi correction và ưu tiên handoff nếu user gặp lỗi 2 lần liên tiếp.

### Path 4 - User mất niềm tin (trust breakdown)
- Sau vài lần trả lời sai, user muốn bỏ qua AI.
- Luôn có CTA "Gặp tư vấn viên ngay", "Gọi hotline", "Đặt lịch tại showroom" hiển thị rõ ở cuối mỗi phiên chat.
- User có thể tắt gợi ý tự động và chỉ dùng menu thủ công.

---

## 3) Eval metrics và threshold (MVP)

### Chatbot mua xe/bảo dưỡng
- Intent routing accuracy >= 92% (mua xe / bảo dưỡng / bảo hành / giá cả / khác).
- Grounded answer rate >= 90% (câu trả lời có trích dẫn từ KB hợp lệ).
- Escalation appropriateness precision >= 85% (khi bot chuyển người thật thì đúng trường hợp).
- Task completion rate:
  - Đặt lịch lái thử thành công >= 35% trên user có ý định mua.
  - Đặt lịch bảo dưỡng thành công >= 45% trên user có nhu cầu.
- CSAT sau phiên chat >= 4.2/5.

### Phân tích review
- Sentiment classification macro-F1 >= 0.82.
- Topic extraction quality (human judged useful) >= 80%.
- Time-to-insight cho team CSKH giảm >= 50% so với tổng hợp thủ công.

---

## 4) Failure modes và giải pháp giảm rủi ro

1. **Hallucination thông số/chính sách**
   - Mitigation: chỉ trả lời từ RAG đã whitelist; nếu không có nguồn thì từ chối trả lời + handoff.
2. **Thông tin hết hạn (giá, ưu đãi, lịch showroom)**
   - Mitigation: đồng bộ dữ liệu hằng ngày; gắn timestamp "cập nhật lúc ...".
3. **Tư vấn sai mức độ nghiêm trọng lỗi kỹ thuật**
   - Mitigation: keyword safety trigger (phanh, pin, lỗi đèn cảnh báo) => escalate ngay cho kỹ thuật viên.
4. **Bias review (sample lech)**
   - Mitigation: tách kênh review, trọng số theo xác thực người dùng, dashboard show confidence.
5. **Under-trust hoặc over-trust**
   - Mitigation: hiện confidence + nguồn, cho phép so sánh với lựa chọn khác, luôn có manual fallback.

---

## 5) ROI - 3 kịch bản (tháng, pilot 6 tháng)

### Giả định chung
- 10,000 phiên chat/tháng.
- 30% liên quan mua xe, 50% bảo dưỡng/hỗ trợ sau bán, 20% khác.
- Cost trung bình: 900-1,200 VND/lượt (LLM + embedding + hạ tầng cơ bản).

| Kịch bản | Conservative | Realistic | Optimistic |
|---|---:|---:|---:|
| Tỷ lệ giải quyết không cần người | 20% | 35% | 50% |
| Giờ CSKH tiết kiệm/tháng | 120 | 260 | 420 |
| Giá trị tiết kiệm (VND) | 24,000,000 | 52,000,000 | 84,000,000 |
| Chi phí vận hành AI (VND) | 12,000,000 | 14,000,000 | 18,000,000 |
| Net value (VND) | +12,000,000 | +38,000,000 | +66,000,000 |

**Upside bổ sung:** tăng conversion đặt lái thử -> mua xe, giảm tỷ lệ bỏ lịch bảo dưỡng, phát hiện sớm vấn đề chất lượng qua review trends.

**Kill criteria:** dừng/đổi hướng nếu 2 tháng liên tiếp `Net value < 0` hoặc `CSAT < 3.8` dù đã tối ưu prompt và KB.

---

## 6) Mini AI spec

### Product scope (MVP)
- Kênh: web chat widget + Zalo OA (nếu kịp).
- Tính năng:
  1) Tư vấn mua xe theo nhu cầu/ngân sách.
  2) Hỗ trợ đặt lịch lái thử/bảo dưỡng.
  3) Dashboard sentiment + top issues từ review.

### Inputs/outputs
- Input: câu hỏi user, profile cơ bản (nếu đồng ý), dữ liệu xe và lịch sử bảo dưỡng, review text.
- Output: câu trả lời có nguồn, đề xuất hành động (đặt lịch/gọi tư vấn), nhãn sentiment-topic theo tuần.

### Model và pipeline đề xuất
- LLM cho hoi dap + orchestration function call.
- RAG pipeline: ingest tài liệu VinFast (catalog, FAQ, bảo hành, quy trình bảo dưỡng).
- Classifier sentiment-topic (fine-tune nhẹ hoặc model API + rule post-processing).

### Non-functional requirements
- P95 latency < 4s cho Q&A thông thường.
- Uptime >= 99% trong giờ hành chính.
- Logging đầy đủ cho audit và cải tiến model.
- Bảo mật: ẩn danh PII trong logs, phân quyền dashboard theo vai trò.

### Experiment plan (2-3 tuần)
- Tuần 1: dựng KB + prompt baseline, test 100 câu hỏi mẫu.
- Tuần 2: thêm fallback, confidence gating, handoff flow; đo metric v1.
- Tuần 3: pilot nhỏ với user thật, so sánh trước/sau về CSAT và completion rate.

### Phân công đề xuất
- Trần Gia Khánh: Canvas + user stories + UX flow.
- Vũ Đức Minh: Eval metrics + test set + analysis.
- Trần Sỹ Minh Quân: ROI + business assumptions.
- Nguyễn Lâm Tùng : Mini AI spec + implementation plan/prompt.

---

## Ghi chú nộp bài
- Đổi tên folder `NhomXX-Room` theo đúng tên nhóm và phòng (ví dụ: `Nhom01-403`) trước khi nộp.
- Nếu cần, thêm 1 mục "Hướng đi chính" tóm tắt 3 dòng ngay đầu file để dễ present trên lớp.
