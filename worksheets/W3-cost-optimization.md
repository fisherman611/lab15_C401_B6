# Worksheet 3 – Cost Optimization Debate

> Thời gian: 10:15–10:50 | Nhóm B6: AI Alumni Networking | Feature: LLM Icebreaker

## Checklist
- [x] Chọn 3 chiến lược optimization phù hợp nhất
- [x] Với mỗi chiến lược: tiết kiệm phần nào, lợi ích, trade-off, thời điểm áp dụng
- [x] Chốt: chiến lược nào làm ngay, chiến lược nào để sau

---

## 3 chiến lược được chọn

### 1. Semantic Caching — LÀM NGAY

**Cơ chế:**
Cache kết quả icebreaker theo cặp (user_A_id, user_B_id, event_id). Nếu user A đã xem profile B và đã sinh icebreaker → lần sau B xem A, hoặc A xem lại B → trả cache thay vì gọi LLM.

**Tiết kiệm phần nào:**
- Tại event, nhiều cặp user sẽ match lẫn nhau (A→B và B→A đều được gợi ý)
- Ước tính 30–40% calls là duplicate pairs → tiết kiệm 30–40% token cost
- MVP: tiết kiệm ~$1–2/event | Growth: ~$12–16/event

**Lợi ích thêm:**
- Giảm latency cho lần xem thứ 2 (cache hit < 50ms thay vì ≤5s)
- Giảm tải OpenAI API trong peak

**Trade-off:**
- Cache stale nếu profile user thay đổi trong event (hiếm nhưng có thể xảy ra)
- Cần Redis với TTL hợp lý — đề xuất TTL = duration của event (4–8h)
- Không cache được regenerate request (user đã đánh dấu "không phù hợp")

**Thời điểm áp dụng:** Ngay từ MVP — implementation đơn giản, ROI cao nhất.

---

### 2. Prompt Compression — LÀM NGAY

**Cơ chế:**
Thay vì đưa toàn bộ profile vào prompt, chỉ extract các fields liên quan nhất:
- Thay vì: full profile text (~300 tokens/người)
- Dùng: `{skills_top3, networking_goal, industry}` (~80 tokens/người)

Với RAG context: chỉ lấy top-2 chunks thay vì top-3, và truncate mỗi chunk xuống 150 tokens.

**Tiết kiệm phần nào:**
- Profile: 300 → 80 tokens × 2 người = tiết kiệm ~440 tokens/call
- RAG context: 600 → 400 tokens = tiết kiệm ~200 tokens/call
- Tổng: ~640/1.100 tokens = **tiết kiệm ~58% input tokens**
- MVP: tiết kiệm ~$1.5/event | Growth: ~$14/event

**Lợi ích thêm:**
- Prompt ngắn hơn → LLM xử lý nhanh hơn → giảm latency
- Ít nhiễu context → chất lượng output có thể tốt hơn

**Trade-off:**
- Mất một số context → icebreaker có thể kém cá nhân hóa hơn
- Cần A/B test để xác nhận chất lượng không giảm đáng kể
- Cần viết lại prompt template và test kỹ trước khi deploy

**Thời điểm áp dụng:** Ngay từ MVP — thay đổi prompt template, không cần infra mới.

---

### 3. Model Routing — ĐỂ SAU (Growth phase)

**Cơ chế:**
Phân loại request trước khi gọi LLM:
- **Simple pair** (2 người cùng ngành, goals tương tự) → dùng GPT-4o-mini (~10x rẻ hơn)
- **Complex pair** (khác ngành, goals bổ sung nhau, cần icebreaker sáng tạo) → dùng GPT-4o

Complexity classifier: rule-based đơn giản dựa trên `same_industry` flag + `intent_complementarity` score từ Matching Engine (đã có sẵn).

**Tiết kiệm phần nào:**
- Ước tính 60% pairs là "simple" → 60% calls dùng GPT-4o-mini
- GPT-4o-mini: $0.15/1M input vs GPT-4o: $2.50/1M input → ~17x rẻ hơn
- Tổng tiết kiệm: ~50–60% token cost ở Growth tier

**Lợi ích thêm:**
- GPT-4o-mini latency thấp hơn → dễ đạt SLA ≤5s hơn trong peak

**Trade-off:**
- Cần đánh giá chất lượng icebreaker từ GPT-4o-mini — có thể kém sáng tạo hơn
- Complexity classifier có thể phân loại sai → simple pair nhận icebreaker kém chất lượng
- Thêm 1 bước routing → tăng complexity code và latency nhỏ (~50ms)

**Thời điểm áp dụng:** Growth phase khi có đủ data để calibrate classifier và đủ volume để ROI rõ ràng (> 2.000 calls/event).

---

## Tổng hợp quyết định

| Chiến lược | Làm khi nào | Tiết kiệm ước tính | Độ phức tạp |
|---|---|---|---|
| Semantic Caching | Ngay (MVP) | 30–40% cost | Thấp |
| Prompt Compression | Ngay (MVP) | ~50% input tokens | Thấp |
| Model Routing | Growth phase | 50–60% cost | Trung bình |

> Kết hợp Semantic Caching + Prompt Compression ngay từ MVP có thể giảm cost thực tế từ ~$5 xuống còn ~$2–3/event mà không ảnh hưởng đáng kể đến chất lượng.
