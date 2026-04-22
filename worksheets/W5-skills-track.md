# Worksheet 5 – Skills Map & Track Direction

> Thời gian: 11:25–11:45 | Nhóm B6: AI Alumni Networking

## Checklist
- [x] Mỗi thành viên tự chấm 1–5 ở 3 mảng
- [x] Thảo luận nhóm về điểm mạnh chung
- [x] Chọn Track Phase 2 phù hợp
- [x] Ghi 2–3 kỹ năng cần bù nếu tiếp tục dự án

---

## Bảng tự chấm thành viên (1 = mới học, 5 = tự tin)

| Thành viên | Business/Product | Infra/Data/Ops | AI Engineering |
|---|---|---|---|
| Hoàng Văn Bắc | **4** | 2 | 3 |
| Trần Anh Tú | 2 | **4** | 3 |
| Vũ Phúc Thành | 2 | 3 | **4** |
| Nguyễn Như Giáp | 2 | 2 | **4** |
| Vũ Như Đức | 2 | 3 | **4** |
| Lương Hữu Thành | 2 | 2 | **4** |
| Nguyễn Tiến Thắng | 3 | 2 | **4** |
| **Trung bình nhóm** | **2.4** | **2.6** | **3.7** |

---

## Điểm mạnh chung của nhóm

Nhóm nghiêng rõ về **AI Engineering (CP3)** — 5/7 thành viên đã đăng ký Track 3. Các kỹ năng đã build trong Phase 1 phù hợp trực tiếp với dự án:

- LangGraph pipeline (RAG, multi-node agent) → core của Icebreaker Engine
- Vector embedding + pgvector → Matching Engine
- LLM API integration + prompt engineering → toàn bộ AI features
- Deploy trên AWS + monitoring cơ bản → production baseline

Điểm yếu cần nhận diện: Infra/Data/Ops còn mỏng — chưa có kinh nghiệm với queue (Bull/Redis), CI/CD pipeline, và production-grade monitoring.

---

## Track Phase 2 được chọn

### Phân bổ theo thành viên

| Thành viên | Track Phase 2 | Lý do |
|---|---|---|
| Hoàng Văn Bắc | **Track 1 – AI Business & Product** | Điểm Business cao nhất nhóm, phù hợp vai Product lead |
| Trần Anh Tú | **Track 2 – AI Infrastructure & Data** | Điểm Infra cao nhất nhóm, nhóm cần người lo LLMOps |
| Vũ Phúc Thành | **Track 3 – AI Application** | Core AI engineer, build LangGraph pipeline |
| Nguyễn Như Giáp | **Track 3 – AI Application** | Core AI engineer, build RAG + evaluation |
| Vũ Như Đức | **Track 3 – AI Application** | Core AI engineer, build Matching Engine |
| Lương Hữu Thành | **Track 3 – AI Application** | Core AI engineer, fine-tuning & memory |
| Nguyễn Tiến Thắng | **Track 3 – AI Application** | Core AI engineer, production eval pipeline |

### Lý do phân bổ có căn cứ

Dự án AI Alumni Networking cần cả 3 track để đưa lên production thật:
- **Track 1** (Bắc): định hướng product, cost model, go-to-market cho tổ chức alumni
- **Track 2** (Tú): CI/CD pipeline, vLLM nếu scale, monitoring dashboard production-grade
- **Track 3** (5 người còn lại): advanced agent patterns, memory dài hạn, fine-tuning icebreaker model, production eval với RAGAS

---

## Kỹ năng cần bù để tiếp tục dự án

1. **LLMOps / CI/CD cho AI** — hiện tại deploy thủ công lên AWS, cần automated pipeline để test → staging → production an toàn (Track 2 sẽ cover)
2. **Production Evaluation (RAGAS)** — chưa có benchmark chất lượng icebreaker, cần eval pipeline để đo faithfulness, relevance, và user satisfaction score (Track 3 sẽ cover)
3. **Cost governance & AI Act compliance** — khi mở rộng ra nhiều trường/tổ chức, cần hiểu regulatory landscape và financial model rõ hơn (Track 1 sẽ cover)
