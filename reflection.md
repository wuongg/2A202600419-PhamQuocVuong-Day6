# Individual reflection — Phạm Quốc Vương (AI20K001)

## 1. Role
UX designer + prompt engineer. Phụ trách thiết kế conversation flow cho chatbot **XanhSM Help Center AI** và viết **system prompt** cho pipeline RAG (phân role → retrieve KB → trả lời/stream), kèm guardrails safety & handoff.

## 2. Đóng góp cụ thể
- Thiết kế conversation flow theo đúng kiến trúc dự án: **role classification** (`user/driver/merchant` + nhánh safety) → **KB retrieval top-k** từ `raw/` → **compose prompt** → trả lời (hoặc **preview mode** khi thiếu `OPENAI_API_KEY`) → khuyến nghị **handoff/escalation** khi KB không khớp hoặc có dấu hiệu safety.
- Viết và test 3 phiên bản system prompt trong `app/prompting.py`, chuẩn hoá:
  - Ưu tiên **trích dẫn/đi theo KB hits**, hạn chế bịa khi KB không có
  - Không yêu cầu thông tin nhạy cảm (OTP, thông tin cá nhân)
  - Giọng điệu khác nhau theo role (người dùng / tài xế / nhà hàng)
  Chọn v3 vì **bám KB tốt nhất** và ít “hallucination” nhất trên 10 test cases.
- Đóng góp vào demo UX: mô tả rõ 2 mode **answer vs preview**, cách UI hiển thị **SSE streaming** và metadata (role decision, KB hits, handoff, metrics).

## 3. SPEC mạnh/yếu
- Mạnh nhất: failure modes — nhóm bắt được case:
  - **Câu hỏi chung chung** (“không dùng được app”, “không đặt được xe”) → retrieve trượt, prompt dễ trả lời rộng
  - **Sai role** (câu hỏi của tài xế nhưng bị hiểu là user) → KB bucket sai
  Mitigation: thêm follow-up tối thiểu + rule “không đoán nếu KB score thấp”, và đẩy sang **handoff** khi cần.
- Yếu nhất: ROI — phần ROI/rollout ban đầu còn chưa tách assumption rõ. Lần sau nên chia kịch bản theo mức rollout thật:
  - Conservative: chỉ chạy 1 kênh/1 khu vực + chỉ 1 role (VD: `user`)
  - Base: 3 roles + có handoff rules + logging/feedback
  - Optimistic: rollout toàn hệ thống + liên thông kênh CSKH

## 4. Đóng góp khác
- Test prompt với 10 tình huống Help Center theo từng role, ghi log vào `prompt-tests.md` theo checklist:
  - role decision đúng chưa
  - có bám KB hits không
  - khi nào nên preview/handoff
- Giúp team rà lại metrics hiển thị: ngoài “accuracy” chung, tách theo role và failure type (sai role / retrieve fail / hallucination / handoff sai) để dễ debug pipeline.

## 5. Điều học được
Trước hackathon mình nghĩ “metrics” chỉ là phần kỹ thuật.
Sau khi làm chatbot Help Center mới thấy metric là **product decision** gắn với pipeline: với case safety/khiếu nại cần ưu tiên **recall cho handoff** (đừng bỏ sót), còn với Q&A thường gặp thì ưu tiên **precision bám KB** (trả lời sai gây hiểu nhầm và tốn thời gian).

## 6. Nếu làm lại
Sẽ test prompt sớm hơn và test theo role ngay từ đầu — ngày đầu nhóm tập trung SPEC/flow, đến gần cuối mới chạy test cases nên số vòng iterate chưa đủ. Nếu test sớm hơn thì có thể:
- Tinh chỉnh guardrails “KB-first” rõ hơn cho trường hợp retrieve fail
- Chuẩn hoá follow-up questions cho câu hỏi mơ hồ
- Cân bằng lại ngưỡng handoff để giảm false alarm mà vẫn an toàn

## 7. AI giúp gì / AI sai gì
- **Giúp:** dùng AI để brainstorm failure modes và biến thể câu hỏi (thiếu từ khoá, phrasing lạ, hỏi vòng vo), từ đó bổ sung follow-up + handoff rules. Dùng AI để smoke-test nhanh nhiều prompt variants (đặc biệt các case dễ hallucination khi KB mỏng).
- **Sai/mislead:** AI hay gợi ý thêm feature “đẹp” như đặt lịch/CSKH đa kênh ngay trong chatbot — nghe hay nhưng vượt scope prototype. Bài học: AI brainstorm tốt nhưng không tự giới hạn scope, cần giữ mục tiêu: **role → KB → answer/preview (SSE) → handoff + metrics**.
