# E402-Day17

# Day 17 Submission
**Student:** Nguyễn Bình Thành
**Date:** 25/04/2026
**Product idea:** Công cụ AI tự động trích xuất flashcard y khoa từ tài liệu cá nhân với khả năng truy xuất ngược (traceability) chuẩn xác tới từng dòng text để tối ưu chu trình ôn thi nước rút.

---

## 1. MVP Boundary Sheet

**Riskiest Assumption:**
> Chúng tôi tin rằng rủi ro lớn nhất là sinh viên Y khoa (những người có zero-tolerance với kiến thức sai) sẽ không đủ tin tưởng để học theo Flashcard do AI tạo ra, sợ hãi việc AI bịa đặt (hallucination) dẫn đến sai lệch kiến thức lâm sàng.

**In-Scope** (tối đa 3):
- [x] Luồng tạo Flashcard 1-chạm từ PDF/Text/Ảnh — test giả định: Giải quyết triệt để thời gian chết do phải tạo tài liệu thủ công.
- [x] Nút "Xem nguồn" (Traceability / Source Citation) chia đôi màn hình highlight text gốc — test giả định: Xây dựng sự tin tưởng tuyệt đối, phá vỡ rào cản hoài nghi AI.
- [x] Contextual Chatbot giới hạn (sandbox) trong file gốc đính kèm thẻ — test giả định: Đáp ứng nhu cầu đào sâu (deep-dive) kiến thức an toàn mà không cần thoát khỏi ngữ cảnh ôn tập.

**Out-of-Scope:**
- Xây dựng Cold Cache / Database giáo trình tĩnh (Medical Ontology) — lý do bỏ: Tốn kém, rủi ro compliance y tế cao, làm chậm thời gian ra mắt MVP. Chỉ tập trung Local RAG.
- Tính năng chia sẻ bộ thẻ (Community Hub) — lý do bỏ: MVP cần chứng minh user có thể nhận giá trị từ việc tự học bộ thẻ của chính mình trước. Tránh rắc rối bản quyền tài liệu.

**Non-Goals:**
- Tuyệt đối không trở thành "Bách khoa toàn thư Y tế".
- Cấm AI tự ý sử dụng kiến thức ngoài internet để trả lời hoặc tạo thẻ nếu thông tin đó không xuất hiện trong tài liệu user upload.

---

## 2. PRD Skeleton

### Problem Statement
> Sinh viên Y khoa tốn quá nhiều thời gian và năng lượng chuyển đổi thủ công tài liệu (slide, handnote) thành thẻ ghi nhớ Anki/Quizlet, khiến họ kiệt sức trước khi thực sự bước vào chu trình học thuộc (active recall), dẫn đến kém hiệu quả và hoảng loạn sát kỳ thi.

### Target User
> Sinh viên Y khoa năm 1-4 tại các trường Đại học (như Y Hà Nội, Y Dược TP.HCM), quen thuộc với phương pháp Active Recall, đang trong trạng thái quá tải khối lượng bài vở trước kỳ thi 1-3 tuần.

### User Stories
**Story 1: The Generator**
> As a sinh viên Y khoa đang ôn thi, I want to tải lên 1 file PDF bài giảng dài 50 trang và bấm tạo thẻ, so that tôi có ngay 100 thẻ flashcard chia theo chủ đề lâm sàng mà không mất hàng giờ copy-paste.

**Story 2: The Skeptic**
> As a sinh viên vô cùng cẩn thận với kiến thức, I want to bấm nút "Xem nguồn" trên một thẻ flashcard về liều lượng thuốc, so that màn hình ngay lập tức trích xuất và highlight đúng dòng tài liệu gốc, giúp tôi an tâm 100% học mà không sợ AI bịa kiến thức.

### AI-Specific

**Model Selection:**
- Model: GPT-4o-mini (hoặc Claude 3.5 Haiku)
- Lý do chọn: Tối ưu chi phí, tốc độ cao (latency thấp) và cực kỳ xuất sắc trong việc tuân thủ JSON format để bóc tách cặp Q&A kèm metadata trích dẫn.
- Trade-offs chấp nhận: Giới hạn độ dài file upload trong 1 lần (ví dụ: tối đa 30 trang) để đảm bảo tốc độ tạo thẻ dưới 60 giây.
- Trade-offs không chấp nhận: Hallucination. AI thà báo lỗi "Không trích xuất được nội dung" còn hơn tự bịa ra câu hỏi/đáp án.

**Data Requirements:**
- Nguồn: Dữ liệu cá nhân user tự tải lên (PDF, Slide, Note tĩnh).
- Owner: User hoàn toàn sở hữu và quản lý.
- Update frequency: Dữ liệu tĩnh theo từng phiên upload.

**Fallback UX:**
- Chiến lược: Graceful Handover kết hợp Quản trị kỳ vọng.
- Trigger: Khi Confidence score của hệ thống trích xuất thấp, hoặc user đặt câu hỏi cho Chatbot ngoài phạm vi file gốc.
- Hành động: Gắn cờ (flag) màu cam "Cần review" trên các thẻ AI không tự tin. Chatbot tự động trả lời: "Tài liệu bạn cung cấp không chứa thông tin này, vui lòng kiểm tra lại giáo trình chuẩn."
- User options: Cho phép user Edit trực tiếp mặt thẻ, hoặc bấm nút "Báo lỗi" để xóa thẻ.

### Success Metrics
- Primary metric: Tỷ lệ (%) user hoàn thành ôn tập (review) ít nhất 50 thẻ flashcard trong vòng 48 giờ sau khi tạo Deck đầu tiên.
- Ngưỡng thành công: > 30% user upload tài liệu đạt được mốc này.
- Timeframe đo lường: 30 ngày kể từ lúc launch MVP.

### Dependencies & Constraints
- Giải pháp Parsing PDF y khoa đủ tốt để không làm vỡ cấu trúc các bảng biểu phân tích chỉ số cận lâm sàng.
- Chi phí API LLM + Vector DB phải được kiểm soát dưới mức LTV (Life-time Value) dự kiến của sinh viên.

---

## 3. Hypothesis Table

### Hypothesis 1 (cho tính năng In-Scope #2 - Traceability)
> "Chúng tôi tin rằng [tính năng truy xuất ngược highlight file gốc] sẽ giúp [sinh viên Y khoa hoài nghi] đạt được [sự tin tưởng tuyệt đối để học theo AI].
> Chúng tôi sẽ biết mình đúng khi thấy [tần suất bấm nút "Xem nguồn"] giảm dần đều [sau 3 session ôn tập đầu tiên của user]."

Riskiest assumption: Nếu không có nút "Xem nguồn", user sẽ vứt bỏ app ngay từ thẻ flashcard đầu tiên họ cảm thấy "hơi lạ".
Cách test cheapest: Wizard of Oz — Tự làm thủ công 1 deck Anki 20 thẻ, kèm theo ảnh chụp màn hình crop thẳng từ slide đính vào phần giải thích. Đưa cho 5 bạn cùng nhóm học thử xem tốc độ ghi nhớ và sự tự tin của họ có tăng lên không.

---

## 4. PMF Scorecard

**Aha Moment:**
> Đọc một thẻ flashcard khó → nghi ngờ → bấm nút "Xem Nguồn" → màn hình chia đôi chỉ đúng dòng chữ trong file giáo trình của mình → thở phào nhẹ nhõm, xác nhận AI đáng tin và quẹt thẻ (swipe) liên tục không dừng.

**Actionable Metric:**
> Actionable Retention: Số lượng user thực hiện thao tác quẹt (review) tối thiểu 50 thẻ flashcard trong 48h đầu tiên.

**PMF Method:**
> Aha Moment tracking.
> Ngưỡng thành công: 30% user đăng ký mới đạt được Aha Moment trong 2 ngày đầu.

**Vanity Metrics tôi sẽ không dùng:**
- Số lượng tài liệu (PDF) được upload (Upload rác mà không học thì vô nghĩa).
- Tổng số thẻ Flashcard được AI generate (Generate dễ, Consume mới khó).

---

## 5. AI Critique Log

**Điểm AI chỉ ra:**
1. Scope Creep ở "Shareable Link/Cộng đồng" — Action: Accept — Lý do: Đã đẩy hoàn toàn sang Out-of-Scope để tập trung làm mượt luồng Personal Use và Traceability.
2. Fallback UX mơ hồ khi AI thiếu context — Action: Accept — Lý do: Đã bổ sung cơ chế Graceful Handover (Chatbot từ chối trả lời nếu thông tin nằm ngoài Sandbox file gốc).
3. Vanity Metric "Số lượng thẻ tạo ra" — Action: Accept — Lý do: Đã chuyển hướng sang tracking "Số thẻ được Review/Học" (Actionable Metric).

**Thay đổi lớn nhất giữa Version A và Version B:**
> Thay đổi hoàn toàn định vị từ một "Hệ thống quản lý lộ trình tĩnh" sang "AI Flashcard Generator tích hợp Traceability". Quyết định cắt bỏ việc xây dựng Medical Ontology (Cold Cache) giúp MVP nhẹ hơn, khả thi về mặt kỹ thuật, và giải quyết trực diện, tức thì nỗi đau "Active Recall" của sinh viên Y.

---

## 6. Self-assessment

Mắt xích nào trong [MVP Boundary → PRD → Hypothesis → PMF] bạn đang yếu nhất?
> Kỹ thuật thực thi tính năng Traceability: Làm sao để AI trích xuất được tọa độ chính xác (Bounding Box) của đoạn text trên file PDF, sau đó Render ngược lại lên UI để highlight mượt mà cho sinh viên xem. 

Open questions bạn muốn giải đáp tiếp:
1. Nên áp dụng chiến lược thu phí (Pricing) như thế nào ở giai đoạn PMF? Thu theo gói Subscription tháng hay bán Token theo dung lượng PDF upload?
2. Trong MVP, có nên loại bỏ hoàn toàn đầu vào là "Ảnh chụp vở ghi chép tay" để đảm bảo Accuracy cao nhất, chấp nhận hy sinh một phần User Experience không?
