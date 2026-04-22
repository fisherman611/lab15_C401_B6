# Worksheet 4 – Scaling & Reliability Tabletop

> Thời gian: 10:50–11:25 | Nhóm B6: AI Alumni Networking | Feature: LLM Icebreaker

## Checklist
- [x] Phân tích 3 tình huống: traffic tăng đột biến, provider timeout, response chậm
- [x] Với mỗi tình huống: tác động tới user, phản ứng ngắn hạn, giải pháp dài hạn
- [x] Chọn các metric cần monitor
- [x] Viết fallback proposal
- [x] Xác định request nào cần real-time, request nào có thể async
- [x] Quyết định có cần queue, circuit breaker, retry policy không

---

## 3 tình huống failure

### Tình huống 1 — Traffic tăng đột biến (Event khai mạc)

**Bối cảnh:** 500 người check-in trong 15 phút đầu, đồng loạt mở app xem gợi ý → 200+ Icebreaker requests cùng lúc.

**Tác động tới user:**
- Latency tăng vượt SLA 5s → spinner kéo dài → user bỏ qua tính năng
- OpenAI rate limit (TPM/RPM) bị chạm → 429 error → request thất bại hoàn toàn

**Phản ứng ngắn hạn:**
- Retry với exponential backoff (1s → 2s → 4s, tối đa 3 lần)
- Trả về cached icebreaker của cặp tương tự nếu có (semantic cache hit)
- Hiển thị skeleton UI + message "Đang tạo gợi ý..." thay vì màn hình trắng

**Giải pháp dài hạn:**
- Request queue (Redis Queue / Bull): nhận request ngay, xử lý tuần tự, push kết quả qua polling hoặc SSE
- Pre-generate icebreaker cho top-20 matches của mỗi user ngay khi check-in (background job) → khi user mở profile đã có sẵn kết quả

---

### Tình huống 2 — OpenAI API timeout / down

**Bối cảnh:** OpenAI có sự cố (xảy ra ~2–3 lần/năm), toàn bộ LLM calls trả về 503/timeout trong giữa event.

**Tác động tới user:**
- Icebreaker Engine hoàn toàn không hoạt động
- Matching Engine vẫn chạy được (không phụ thuộc OpenAI) nhưng thiếu icebreaker
- UX bị hỏng một phần tại thời điểm quan trọng nhất của event

**Phản ứng ngắn hạn:**
- Circuit breaker: sau 5 lỗi liên tiếp trong 30s → mở circuit, không gọi OpenAI nữa
- Kích hoạt fallback ngay lập tức (xem Fallback Proposal bên dưới)
- Alert tới organizer dashboard: "Icebreaker tạm thời dùng gợi ý mặc định"

**Giải pháp dài hạn:**
- Fallback sang Gemini Flash hoặc Claude Haiku (cấu hình sẵn, chỉ cần đổi API key)
- Half-open circuit: sau 60s thử lại 1 request, nếu thành công → đóng circuit

---

### Tình huống 3 — Response chậm (Latency vượt SLA 5s)

**Bối cảnh:** RAG pipeline có 4 nodes (retrieve → grade → generate → validate), mỗi node có latency riêng. Khi load cao, tổng latency dễ vượt 5s.

**Tác động tới user:**
- User chờ quá lâu → đóng màn hình → mất cơ hội networking
- Đặc biệt tệ trên mobile với mạng wifi hội trường không ổn định

**Phản ứng ngắn hạn:**
- Streaming response: trả về từng câu icebreaker ngay khi generate xong thay vì chờ đủ 3 câu
- Timeout cứng 4.5s: nếu pipeline chưa xong → trả về kết quả partial (1–2 câu) + "Đang tải thêm..."

**Giải pháp dài hạn:**
- Bỏ `grade_docs_node` ở MVP (dùng top-2 chunks cố định thay vì grade động) → giảm 1 LLM call trong pipeline
- Prompt Compression (W3) giúp giảm latency generate node
- Cache embedding của knowledge base, không re-embed mỗi request

---

## Fallback Proposal

| Tình huống | Fallback | Chất lượng |
|---|---|---|
| OpenAI down | Gọi Gemini Flash (cấu hình sẵn) | ~85% so với GPT-4o |
| Gemini cũng down | Template-based icebreaker: "Bạn và [tên] đều quan tâm đến [shared_skill]. Bạn có thể hỏi về kinh nghiệm của họ trong lĩnh vực này." | ~50% — vẫn dùng được |
| Latency > 4.5s | Trả partial result (1 câu) + loading indicator | Chấp nhận được |
| Cache miss + queue đầy | Hiển thị profile match mà không có icebreaker, user tự bắt chuyện | Degraded nhưng không crash |

---

## Real-time vs Async

| Request | Xử lý | Lý do |
|---|---|---|
| Xem profile → sinh icebreaker lần đầu | **Async với streaming** | Cần kết quả nhanh nhưng có thể stream từng câu |
| Pre-generate top-20 matches khi check-in | **Background async** | Không cần ngay, chạy nền sau check-in |
| Regenerate icebreaker (user bấm "không phù hợp") | **Real-time** | User đang chờ, tối đa 3 lần/cặp |
| Update knowledge base (event agenda mới) | **Background async** | Chạy trước event, không cần real-time |
| Logging / tracing | **Async fire-and-forget** | Không được block main flow |

---

## Quyết định infra

| Pattern | Dùng không | Lý do |
|---|---|---|
| Request Queue (Redis/Bull) | **Có** — Growth phase | Xử lý spike check-in đầu event |
| Circuit Breaker | **Có** — ngay từ MVP | OpenAI down là rủi ro thực tế |
| Retry + Exponential Backoff | **Có** — ngay từ MVP | Rate limit 429 xảy ra thường xuyên khi peak |
| WebSocket / SSE | **SSE** — MVP | Streaming từng câu icebreaker, đơn giản hơn WebSocket |
| Pre-generation background job | **Có** — Growth phase | Giải quyết latency tại thời điểm check-in |

---

## Metrics cần monitor

| Metric | Ngưỡng cảnh báo | Tool |
|---|---|---|
| Icebreaker latency (p95) | > 4s | LangSmith + AWS CloudWatch |
| OpenAI error rate | > 5% trong 1 phút | Custom alert |
| Cache hit rate | < 20% (bất thường) | Redis metrics |
| Queue depth | > 50 requests pending | Bull dashboard |
| Regenerate rate | > 40% (chất lượng kém) | Custom logging |
