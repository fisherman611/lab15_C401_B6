# Worksheet 0 – Learning Timeline

> Thời gian: 08:00–08:40 | Nhóm B6: AI Alumni Networking

## Phần 0 — Mini Reflection (08:00–08:15)

> Mỗi người viết 3 ý cá nhân, sau đó 1 người tổng hợp thành điểm chung của nhóm.

### Bảng reflection cá nhân

| Thành viên | Điều học được nhiều nhất | Điểm yếu nhất | Hướng muốn đi sâu |
|---|---|---|---|
| Hoàng Văn Bắc | Product thinking cho AI — PRD, cost model, go-to-market | Kỹ thuật infra và deployment | AI Business Strategy & Governance |
| Trần Anh Tú | Deploy cloud, monitoring, structured logging | LLM engineering và RAG pipeline | LLMOps, CI/CD cho AI |
| Vũ Phúc Thành | LangGraph agent, RAG pipeline, tool calling | Production eval và benchmark | Advanced agent patterns |
| Nguyễn Như Giáp | RAG pipeline, vector embedding, prompt engineering | Infra và scaling | Fine-tuning & evaluation |
| Vũ Như Đức | Multi-agent, MCP, guardrails | Cost optimization thực tế | Production eval pipeline |
| Lương Hữu Thành | LLM API, ReAct agent, UX trust layer | Monitoring và alerting | Memory & long-term context |
| Nguyễn Tiến Thắng | Evaluation, RAGAS, benchmark | Deployment và cloud infra | GraphRAG & knowledge graphs |

### Tổng hợp nhóm (3–5 ý)

1. Điểm mạnh chung: AI Engineering — cả nhóm tự tin với LLM API, RAG pipeline, LangGraph agent
2. Điểm yếu chung: Production ops — monitoring, CI/CD, cost governance còn mỏng
3. Khoảng trống lớn nhất: chưa có eval pipeline thực tế để đo chất lượng AI output
4. Hướng đi sâu phổ biến nhất: Advanced agent + production eval (5/7 người)
5. Nhóm có đủ 3 track để cover toàn bộ dự án từ product → infra → AI core

---

## Checklist
- [x] Liệt kê 2–3 kỹ năng nhóm tự tin nhất
- [x] Mô tả ngắn sản phẩm/agent đã làm
- [x] Chốt 1 chủ đề xuyên suốt cả ngày
- [x] Trả lời: bài toán gì? ai là user? vì sao phù hợp để phân tích deployment & cost?

---

## Kỹ năng nhóm tự tin nhất
- LLM API + Prompt Engineering (RAG pipeline, LangGraph)
- AI Onboarding & Matching Engine (vector embedding, cosine similarity)
- Deploy lên cloud (AWS), monitoring cơ bản

## Sản phẩm đã làm
AI Alumni Networking Platform — chuyển trải nghiệm networking tại event offline từ "ngẫu nhiên" sang "AI-driven & context-aware".

## Feature được chọn để phân tích xuyên suốt
> **US-06 — LLM Icebreaker (RAG-based)**
> Khi user xem profile người được gợi ý, hệ thống dùng LangGraph RAG pipeline (retrieve → grade → generate → validate) để sinh 3 câu mở đầu cuộc trò chuyện phù hợp với cả hai profile, trong ≤5 giây.

## Bài toán giải quyết
Người tham dự event biết *ai* nên gặp (từ Matching Engine) nhưng không biết *nói gì* — dẫn đến ngại tiếp cận dù đã có gợi ý. Icebreaker Engine giải quyết điểm ma sát cuối cùng trước khi kết nối thật sự xảy ra.

## Người dùng chính
- Persona 1 (Fresh graduate, 18–22): ngại bắt chuyện, cần câu mở đầu đơn giản
- Persona 2 (Mid-level, 22+): muốn networking chất lượng, cần câu mở đầu có chiều sâu

## Vì sao phù hợp để phân tích deployment & cost
- Mỗi lần user xem profile → 1 LLM call (GPT-4o) với RAG context → cost tăng tuyến tính theo traffic
- Có thể regenerate tối đa 3 lần/cặp/event → worst case 3x LLM calls
- Latency SLA nghiêm ngặt (≤5s) → ảnh hưởng trực tiếp đến UX tại event
- Knowledge base (event agenda, speaker bios) cần cập nhật theo từng event → data pipeline cost
