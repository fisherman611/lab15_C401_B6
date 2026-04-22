# Worksheet 2 – Cost Anatomy Lab

> Thời gian: 09:35–10:15 | Nhóm B6: AI Alumni Networking | Feature: LLM Icebreaker

## Checklist
- [x] Ước lượng: số user/ngày, request/ngày, peak traffic
- [x] Ước lượng input/output tokens nếu dùng LLM API
- [x] Bóc tách cost theo từng lớp
- [x] Tính sơ bộ cost ở mức MVP
- [x] Phân tích khi user tăng 5x–10x thì phần nào tăng mạnh nhất
- [x] Nhận diện hidden costs

---

## Ước lượng traffic

| Thông số | MVP (1 event nhỏ) | Growth (1 event lớn) |
|---|---|---|
| Người tham dự | 200 người | 1.000 người |
| % dùng Icebreaker | 50% | 60% |
| Lượt xem profile/người | 5 lần | 8 lần |
| Regenerate rate | 20% | 20% |
| **Tổng LLM calls/event** | **~600** | **~5.760** |
| Peak (trong 2h cao điểm) | ~300 calls/2h = 2,5 calls/phút | ~2.880 calls/2h = 24 calls/phút |

> Regenerate tính: mỗi lượt xem có 20% khả năng regenerate 1 lần → calls = base × 1.2

---

## Ước lượng tokens mỗi LLM call (GPT-4o)

| Thành phần prompt | Tokens ước tính |
|---|---|
| System prompt + instruction | ~200 |
| Profile user A (anonymized) | ~150 |
| Profile user B (anonymized) | ~150 |
| RAG context (event agenda + speaker bios, top-3 chunks) | ~600 |
| **Tổng input tokens** | **~1.100** |
| Output (3 câu icebreaker) | ~150 |
| **Tổng mỗi call** | **~1.250 tokens** |

> Giá GPT-4o hiện tại (2025): input $2.50/1M tokens, output $10/1M tokens

---

## Bóc tách cost theo lớp

### MVP — 1 event 200 người (~600 LLM calls)

| Lớp cost | Chi tiết | Ước tính/event |
|---|---|---|
| **API Tokens** | 600 × 1.100 × $2.50/1M (input) + 600 × 150 × $10/1M (output) | ~$1.65 + ~$0.90 = **~$2.55** |
| **Compute** | AWS ECS Fargate: 1 task × 0.5 vCPU × $0.04048/vCPU-h × 2h + 1GB × $0.004445/GB-h × 2h | **~$0.05** |
| **Storage** | Amazon RDS pgvector (db.t3.micro) + ElastiCache Redis (cache.t3.micro) — chia sẻ với toàn platform | ~$0.50 |
| **Logging / Monitoring** | AWS CloudWatch Logs + LangSmith trace | ~$0.20 |
| **Embedding (knowledge base)** | Cập nhật event KB: ~50 chunks × 500 tokens = 25K tokens × $0.02/1M (text-embedding-3-small) | ~$0.001 |
| **Tổng trực tiếp/event** | | **~$3.30** |
| **Hidden costs (×1.5)** | Retry overhead, validate node calls, eval pipeline, guardrails | **~$1.65** |
| **Tổng thực tế/event** | | **~$5** |

### Growth — 1 event 1.000 người (~5.760 LLM calls)

| Lớp cost | Ước tính/event |
|---|---|
| API Tokens | 5.760 × 1.100 × $2.50/1M + 5.760 × 150 × $10/1M ≈ **$15.84 + $8.64 = ~$24.5** |
| Compute (AWS ECS Fargate: 3 tasks × 0.5 vCPU × 2h) | ~$0.15 |
| Storage (RDS + ElastiCache, chia sẻ platform) | ~$1 |
| Logging / Monitoring (CloudWatch + LangSmith) | ~$1 |
| **Tổng trực tiếp** | **~$26.65** |
| Hidden costs (×1.5) | **~$13.35** |
| **Tổng thực tế/event** | **~$40** |

---

## Phân tích khi scale 5x–10x

| Lớp | Tăng theo | Ghi chú |
|---|---|---|
| API Tokens | **Tuyến tính** (×5–10) | Cost driver lớn nhất, không có ceiling |
| Compute | **Bậc thang** | Scale-to-zero giúp giảm, nhưng peak spike tăng mạnh |
| Storage | **Gần như cố định** | Vector DB và Redis không tăng nhiều theo số calls |
| Logging | **Tuyến tính nhẹ** | Tăng theo số traces |
| Hidden costs | **Tuyến tính** | Retry và validate node tăng cùng tỷ lệ với calls |

> **Kết luận:** API token cost là cost driver lớn nhất và tăng tuyến tính. Đây là nơi cần optimize đầu tiên.

---

## Hidden costs dễ bị quên

1. **Validate node** trong LangGraph pipeline — mỗi call có thêm 1 LLM call nhỏ để kiểm tra đúng 3 câu, đúng ngôn ngữ → thực tế ~1.1–1.2x số calls
2. **Retry overhead** — OpenAI API timeout hoặc rate limit → retry tự động, có thể gấp đôi calls trong peak
3. **RAG grading node** — `grade_docs_node` dùng LLM để lọc chunks → thêm ~200 tokens/call
4. **Eval pipeline** — nếu chạy RAGAS để đánh giá chất lượng icebreaker → thêm LLM calls ngoài production
5. **Knowledge base update** — mỗi event mới phải re-embed toàn bộ agenda + speaker bios
