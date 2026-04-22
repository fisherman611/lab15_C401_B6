# Mini Project Proposal – AI Alumni Networking
## Enterprise Readiness Review: LLM Icebreaker Feature

> Nhóm B6: AI Alumni Networking | 7 thành viên | Ngày tổng kết Phase 1

---

## Slide 1 — Tên dự án, User, Bài toán

**Dự án:** AI Alumni Networking Platform
**Feature phân tích:** US-06 — LLM Icebreaker (RAG-based)

**User:**
- Fresh graduate (18–22): biết *ai* nên gặp nhưng không biết *nói gì*, ngại bắt chuyện
- Mid-level professional (22+): muốn networking chất lượng, cần câu mở đầu có chiều sâu

**Bài toán:**
Matching Engine đã giải quyết "gặp ai" — nhưng điểm ma sát cuối cùng vẫn còn: người dùng ngại tiếp cận dù đã có gợi ý. Icebreaker Engine sinh 3 câu mở đầu cá nhân hóa dựa trên profile 2 người + event context, trong ≤5 giây.

```
User xem profile → LangGraph RAG pipeline
  retrieve → grade → generate → validate
→ 3 câu icebreaker phù hợp ngữ cảnh
```

---

## Slide 2 — Bối cảnh Enterprise & Ràng buộc

**Khách hàng:** Trường đại học / tổ chức alumni, event 200–1.000 người, 1–4 lần/năm

**3 ràng buộc enterprise lớn nhất:**

| # | Ràng buộc | Tác động |
|---|---|---|
| 1 | PDPA Việt Nam — dữ liệu cá nhân không được chuyển ra nước ngoài nếu không có consent | Phải anonymize profile trước khi đưa vào prompt GPT-4o |
| 2 | Latency SLA ≤5s tại event — wifi hội trường đông người, mạng không ổn định | Pipeline 4 nodes phải hoàn thành trong 5s, cần streaming |
| 3 | Traffic spike theo event schedule — 90% traffic trong 2–3h cao điểm, idle còn lại | Hạ tầng phải scale nhanh và không tốn tiền khi không có event |

---

## Slide 3 — Kiến trúc Triển khai Đề xuất

**Lựa chọn: Cloud API (AWS + OpenAI) với data residency control**

```
[Mobile App]
     │
     ▼
[AWS ECS Fargate — Icebreaker Service]
     │
     ├── Redis Cache (semantic cache theo cặp user)
     │
     ├── LangGraph Pipeline
     │     retrieve → grade → generate → validate
     │
     ├── pgvector (profile embeddings)
     │
     └── OpenAI GPT-4o API
           (profile đã anonymized trước khi gửi)
```

**Lý do chọn Cloud thay vì On-prem:**
- Scale-to-zero khi không có event → không tốn chi phí GPU idle (On-prem utilization chỉ ~5%)
- GPT-4o chất lượng icebreaker vượt trội so với self-hosted model cùng size ở quy mô MVP
- Self-hosted chỉ có lợi khi > 1M tokens/ngày — ngưỡng này cần hàng chục event lớn đồng thời

**Trade-off chấp nhận:**
- Anonymize profile (tên thật → ID) trước khi đưa vào prompt + ký DPA với OpenAI
- Cần circuit breaker + fallback khi OpenAI down

---

## Slide 4 — Cost Anatomy

**Ước lượng traffic:**

| Tier | Người/event | LLM calls/event | Thời gian |
|---|---|---|---|
| MVP | 200 | ~600 | 2h cao điểm |
| Growth | 1.000 | ~5.760 | 2h cao điểm |

**Bóc tách cost (GPT-4o, ~1.250 tokens/call, giá 2025: input $2.50/1M, output $10/1M):**

| Lớp | MVP/event | Growth/event |
|---|---|---|
| API Tokens (40%) | ~$2.55 | ~$24.5 |
| Compute (AWS ECS Fargate) | ~$0.05 | ~$0.15 |
| Storage (RDS + ElastiCache) | ~$0.50 | ~$1 |
| Logging / Monitoring (CloudWatch) | ~$0.20 | ~$1 |
| **Tổng trực tiếp** | **~$3.30** | **~$26.65** |
| Hidden costs (×1.5) | ~$1.65 | ~$13.35 |
| **Tổng thực tế** | **~$5** | **~$40** |

**Cost driver lớn nhất:** API tokens — chiếm ~77% tổng cost, tăng tuyến tính theo số calls

**Hidden costs hay bị quên:** validate node + retry overhead + RAG grading node trong LangGraph → cộng thêm ~50% so với API cost thuần

---

## Slide 5 — Cost Optimization

**2 chiến lược làm ngay (MVP):**

| Chiến lược | Cơ chế | Tiết kiệm | Trade-off |
|---|---|---|---|
| Semantic Caching | Cache icebreaker theo (user_A, user_B, event_id) trong Redis, TTL = duration event | 30–40% cost | Cache stale nếu profile thay đổi trong event (hiếm) |
| Prompt Compression | Chỉ lấy top-3 fields thay vì full profile, truncate RAG chunks xuống 150 tokens | ~50% input tokens | Icebreaker kém cá nhân hóa hơn một chút, cần A/B test |

**1 chiến lược để Growth phase:**

| Chiến lược | Cơ chế | Tiết kiệm | Điều kiện áp dụng |
|---|---|---|---|
| Model Routing | Simple pair → GPT-4o-mini (100x rẻ hơn), Complex pair → GPT-4o | 50–60% cost | Cần calibrate complexity classifier, > 2.000 calls/event |

**Kết quả kỳ vọng sau khi áp dụng 2 chiến lược đầu:**
> MVP cost: ~$5 → **~$2–3/event** mà không ảnh hưởng đáng kể đến chất lượng

---

## Slide 6 — Reliability & Scaling

**3 failure scenarios và cách xử lý:**

| Tình huống | Tác động | Phản ứng ngắn hạn | Giải pháp dài hạn |
|---|---|---|---|
| Traffic spike check-in | Latency vượt 5s, 429 rate limit | Retry exponential backoff, trả cache nếu có | Request queue (Bull/Redis), pre-generate top-20 matches khi check-in |
| OpenAI API down | Icebreaker hoàn toàn không hoạt động | Circuit breaker → fallback Gemini Flash | Cấu hình sẵn multi-provider, half-open circuit sau 60s |
| Response chậm > 4.5s | User bỏ qua tính năng | Streaming từng câu, timeout cứng 4.5s trả partial | Bỏ grade_docs_node ở MVP, cache KB embedding |

**Fallback 3 tầng:**
```
GPT-4o → Gemini Flash → Template-based icebreaker
(100%)     (~85%)          (~50% — vẫn dùng được)
```

**Metrics cần monitor:**

| Metric | Ngưỡng alert |
|---|---|
| Icebreaker latency p95 | > 4s |
| OpenAI error rate | > 5% / phút |
| Cache hit rate | < 20% |
| Regenerate rate | > 40% (chất lượng kém) |

---

## Slide 7 — Track Đề xuất & Bước Đi Tiếp Theo

**Phân bổ track theo thành viên:**

| Thành viên | Track | Đóng góp cho dự án |
|---|---|---|
| Hoàng Văn Bắc | Track 1 – Business & Product | Cost model, go-to-market, AI governance |
| Trần Anh Tú | Track 2 – Infrastructure & Data | CI/CD pipeline, LLMOps, monitoring dashboard |
| 5 thành viên còn lại | Track 3 – AI Application | Advanced agent, memory, fine-tuning, RAGAS eval |

**Lý do phân bổ:** Dự án cần cả 3 track để đưa lên production thật — Track 1 định hướng product, Track 2 lo hạ tầng vận hành, Track 3 nâng chất lượng AI core.

**3 kỹ năng cần bù:**
1. LLMOps / CI-CD cho AI — hiện deploy thủ công, cần automated pipeline
2. Production Evaluation (RAGAS) — chưa có benchmark chất lượng icebreaker
3. Cost governance & compliance — cần khi mở rộng ra nhiều tổ chức

**Bước đi tiếp theo (Phase 2):**
- Tuần 1: Implement semantic caching + prompt compression → đo cost thực tế
- Tuần 2: Xây eval pipeline với RAGAS, benchmark chất lượng icebreaker
- Tuần 3: CI/CD pipeline + production monitoring dashboard
