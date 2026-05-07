# Day 17 Submission
**Student:** Nguyễn Bình Thành
**Date:** 25/04/2026
**Product idea:** Engine chẩn đoán điểm mù kiến thức từ tài liệu cá nhân — giúp sinh viên biết *chính xác* nên học gì trong thời gian còn lại, thay vì đoán mò.

---

## 1. MVP Boundary Sheet

**Riskiest Assumption:**
> Sinh viên *không biết mình không biết chỗ nào* — và họ sẵn sàng để hệ thống chỉ ra điểm mù đó thông qua một bài quiz chẩn đoán nhanh, thay vì tự quyết định học gì trước theo cảm tính.

*(Nếu assumption này sai — tức là sinh viên thực ra đã biết rõ mình yếu chỗ nào — thì toàn bộ giá trị của sản phẩm sụp đổ. Đây mới là điều cần test trước tiên, không phải vấn đề hallucination.)*

**In-Scope** (tối đa 3):
- [x] **Diagnostic Quiz Loop:** Sau khi upload tài liệu, hệ thống tự động sinh một bộ câu hỏi chẩn đoán nhanh (15–20 câu). Mục tiêu không phải để học — mà để *đo*: sinh viên đang chắc chỗ nào, mù chỗ nào.
- [x] **Study Priority Map:** Output sau quiz không phải flashcard deck — mà là một bản đồ ưu tiên: *"Bạn còn N tiếng. Đây là 3 chủ đề bạn nên tập trung, theo thứ tự."* Được tính dựa trên điểm quiz + phân bổ nội dung trong tài liệu.
- [x] **Targeted Remediation:** Với mỗi chủ đề bị đánh dấu yếu, hệ thống mở một môi trường học tập có cả hai chế độ: flashcard để drill (active recall), và chatbot để giải thích sâu — *giới hạn trong file tài liệu gốc của người dùng*.

**Out-of-Scope:**
- **Spaced Repetition (SRS):** SRS được thiết kế cho horizon tháng → năm. Với sinh viên còn 3 ngày trước thi, lịch "review sau 4 ngày" không có nghĩa gì. SRS sẽ được xem xét lại khi sản phẩm phục vụ người học dài hạn.
- **Cold database giáo trình dùng chung:** AI chỉ làm việc với tài liệu người dùng upload — không phải bách khoa toàn thư Y tế.
- **Community / Chia sẻ bộ thẻ:** Tập trung làm mượt luồng cá nhân trước.

**Non-Goals:**
- Thay thế việc đọc sâu tài liệu. Sản phẩm này giúp người dùng *phân bổ thời gian*, không thay thế việc học.
- Chatbot được phép dùng kiến thức ngoài file upload. Mọi câu trả lời phải được giới hạn trong tài liệu gốc — không phải vì kỹ thuật, mà vì đây là cam kết an toàn với sinh viên Y khoa.

---

## 2. PRD Skeleton

### Problem Statement
> Sinh viên ôn thi không thiếu tài liệu — họ thiếu khả năng biết *nên dùng thời gian còn lại vào đâu*. Khi còn 3 ngày (hoặc 3 tiếng) trước kỳ thi, quyết định "học phần nào trước" thường được đưa ra bằng cảm tính hoặc theo thứ tự tài liệu — không phải theo điểm yếu thật sự. Kết quả: sinh viên ôn đi ôn lại chỗ đã biết, bỏ qua chỗ thực sự mù, vào phòng thi vẫn hụt điểm ở những phần tưởng đã ổn.

### Target User
> Sinh viên có tài liệu PDF/slide của môn học, đang trong giai đoạn ôn thi (1–14 ngày trước thi), và chưa có cách hệ thống nào để biết mình đang yếu chỗ nào *tương đối với lượng nội dung còn lại cần cover*.

*(Không giới hạn ở sinh viên Y khoa — nhưng Y khoa là vertical đầu tiên vì: lượng tài liệu cực lớn, chi phí sai cao, và hành vi ôn thi bằng flashcard đã có sẵn.)*

### User Stories

**Story 1: The Triage Seeker**
> As a sinh viên năm 3 Y khoa còn 4 ngày trước thi Nội khoa, I want to upload slide bài giảng và trả lời nhanh 20 câu hỏi chẩn đoán, So that hệ thống chỉ ra cho tôi: tôi đang yếu nhất ở Suy tim và Đái tháo đường type 2 — và gợi ý tôi nên dành 60% thời gian còn lại vào hai phần này, không phải theo thứ tự slide.

**Story 2: The Skeptic (giữ lại, nhưng đặt đúng vị trí)**
> As a sinh viên cực kỳ cẩn thận với độ chính xác lâm sàng, I want to khi hệ thống chỉ ra tôi yếu ở "Cơ chế bệnh sinh Suy tim trái", tôi có thể bấm vào và thấy đúng đoạn text trong slide gốc mà hệ thống dùng để tạo câu hỏi, So that tôi tin rằng bài quiz đang đo đúng nội dung tôi cần học — không phải AI bịa ra câu hỏi từ kiến thức ngoài luồng.

*(Traceability là cơ chế xây dựng trust cho Diagnostic Loop — không phải tính năng độc lập.)*

**Story 3: The Focused Driller**
> As a sinh viên đã biết mình yếu phần Nội tiết, I want to vào đúng phần đó và có hai chế độ sẵn: flashcard để tự drill active recall, và chatbot để hỏi lại khi không hiểu — tất cả đều giới hạn trong slide của tôi, So that tôi không bị phân tâm bởi kết quả Google hay câu trả lời không liên quan đến đề thi của trường mình.

### AI-Specific

**Model Selection:**
- Model: GPT-4o-mini (generation) + text-embedding-3-small (retrieval cho chatbot).
- Lý do: Đủ năng lực sinh câu hỏi chẩn đoán chất lượng tốt; chi phí kiểm soát được; context window đủ cho tài liệu vừa phải.
- Trade-off chấp nhận: Giới hạn số trang/lần upload để đảm bảo tốc độ.
- Trade-off không chấp nhận: Một câu trả lời đúng không đồng nghĩa với việc sinh viên thực sự hiểu — hệ thống phải capture confidence trước khi reveal để phân biệt mastery thật với lucky guess.
- 
**Data Requirements:**
- Nguồn: File cá nhân (PDF, slide) do user tải lên.
- Owner: User hoàn toàn sở hữu và kiểm soát.
- Chatbot context: Chỉ dùng chunks từ file đã upload — không call internet, không dùng pretrained knowledge ngoài luồng.

**Fallback UX:**
- Khi chatbot nhận câu hỏi ngoài phạm vi file: từ chối rõ ràng — *"Tài liệu bạn cung cấp không đề cập đến điều này. Để đảm bảo chính xác, hãy đối chiếu với giáo trình chuẩn."*
- Khi quiz chẩn đoán không đủ câu hỏi để phân loại chủ đề: thông báo rõ và cho phép người dùng tự chọn focus area thủ công.

### Success Metrics
- **Primary metric:** Tỷ lệ người dùng hoàn thành ít nhất 1 Diagnostic Quiz *và* sau đó tiếp tục học ít nhất 20 phút trong phần được hệ thống gợi ý là yếu nhất.
- **Ngưỡng thành công:** > 40% user đạt được hành vi này trong session đầu tiên.
- **Timeframe:** 30 ngày sau launch.

*(Nếu người dùng làm quiz nhưng không học theo gợi ý → hệ thống chẩn đoán đúng nhưng output không đủ thuyết phục. Nếu không làm quiz → assumption về meta-ignorance sai. Cả hai case đều cho signal rõ.)*

**Vanity Metrics sẽ không dùng:**
- Tổng số câu hỏi được tạo.
- Lượt upload tài liệu.
- Thời gian dùng app (có thể cao vì người dùng lạc lối, không phải vì engaged).

---

## 3. Hypothesis Table

### Hypothesis 1 — Core Assumption
> "Chúng tôi tin rằng [sinh viên không biết chính xác mình yếu chỗ nào tương đối với thời gian còn lại] là một pain point thực sự — không phải chỉ là cảm nhận của nhóm làm sản phẩm.
> Chúng tôi sẽ biết mình đúng khi [> 40% user tiếp tục học theo Study Priority Map sau khi hoàn thành Diagnostic Quiz] trong 30 ngày đầu."

**Riskiest assumption:** Sinh viên *chưa* có cách tự biết điểm yếu — tức là họ đang ra quyết định học theo cảm tính, không theo data.

**Cheapest test:** Phỏng vấn 5 sinh viên Y bằng một câu duy nhất: *"Lần gần nhất ôn thi, bạn quyết định học phần nào trước như thế nào?"* — nếu câu trả lời là "tôi đọc slide từ đầu" hoặc "tôi học phần nào cảm giác không chắc" → hypothesis được ủng hộ. Nếu câu trả lời là "tôi có checklist cụ thể" → cần revisit.

---

## 4. PMF Scorecard

**Aha Moment:**
> Sinh viên làm xong 20 câu quiz chẩn đoán → màn hình hiện ra: *"Bạn chắc 85% phần Tim mạch. Bạn đang mù ở Nội tiết và Thận. Còn 3 ngày — đây là kế hoạch."* → Sinh viên nhìn vào và nghĩ: *"Đây đúng là thứ mình cần."* → Bắt đầu học ngay phần Nội tiết mà không cần suy nghĩ thêm.

**Actionable Metric:**
> Tỷ lệ user hoàn thành Diagnostic Quiz *và* tiếp tục học theo Study Priority Map ít nhất 20 phút trong cùng session.

**PMF Method:**
> Behavioral funnel: Upload → Quiz Complete → Priority Map viewed → Remediation started → 20-min session.
> Drop-off ở bước nào = signal rõ cần fix ở bước đó.

**Vanity Metrics tôi sẽ không dùng:**
- Tổng lượt upload (upload xong bỏ không học = không có PMF).
- Số flashcard được tạo.
- Thời gian trung bình trong app.

---

## 5. AI Critique Log

**Điểm AI chỉ ra:**
1. *"Bounded chatbot from your docs đã có NotebookLM — miễn phí, Google build."* — Action: Accept — Lý do: Đúng. Chatbot không phải core differentiator. Đã hạ chatbot xuống thành tool bên trong Priority Map, không phải sản phẩm chính.
2. *"SRS giải quyết 'nhớ lâu' — nhưng sinh viên ôn thi không cần nhớ lâu, họ cần pass ngày mai."* — Action: Accept — Lý do: SRS đã bị loại khỏi scope. Diagnostic Loop thay thế vì giải quyết đúng constraint thời gian.
3. *"Riskiest assumption không phải hallucination trust mà là: sinh viên có thực sự không biết mình yếu chỗ nào không?"* — Action: Accept — Lý do: Đã thay thế toàn bộ riskiest assumption và thiết kế cheapest test xung quanh câu hỏi này.

**Thay đổi lớn nhất giữa v1 và v2:**
> v1 framing: *"App tạo flashcard nhanh hơn Anki + chatbot giới hạn file."*
> v2 framing: *"Engine chẩn đoán điểm mù — flashcard và chatbot là tool bên trong, không phải sản phẩm."*
> Thay đổi này kéo theo toàn bộ PRD: problem statement, user stories, metric, và aha moment đều được viết lại.

---

## 6. Self-Assessment

**Mắt xích yếu nhất:**
> Hypothesis 1 chưa được test bằng data thật. Toàn bộ sản phẩm đứng trên assumption rằng sinh viên đang ra quyết định học theo cảm tính — chứ không phải theo một hệ thống cá nhân nào đó mà tôi chưa biết. Cần 5 user interview trước khi viết thêm bất cứ dòng code nào cho Diagnostic Loop.

**Open questions:**
1. Sinh viên có *tin* vào kết quả chẩn đoán của AI hay họ sẽ override theo cảm tính? Nếu override → Diagnostic Loop không có giá trị dù đúng.
2. Study Priority Map nên output dưới dạng gì để actionable nhất: danh sách chủ đề? Timeline? Phần trăm thời gian? Cần test với user thật.
3. Với sinh viên không phải Y khoa (luật, kế toán, kỹ thuật) — pain point tương tự không? Vertical mở rộng cần được validate riêng, không assume.
