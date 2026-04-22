# Worksheet 1 – Enterprise Deployment Clinic

> Thời gian: 08:40–09:25 | Nhóm B6: AI Alumni Networking | Feature: LLM Icebreaker

## Checklist
- [x] Xác định bối cảnh tổ chức/khách hàng
- [x] Liệt kê dữ liệu hệ thống sẽ động đến và mức độ nhạy cảm
- [x] Liệt kê 3 ràng buộc enterprise lớn nhất
- [x] Chọn Cloud / On-prem / Hybrid
- [x] Viết 2 lý do rõ ràng cho lựa chọn đó, có trade-off

---

## Bối cảnh tổ chức
Trường đại học / tổ chức alumni tổ chức event định kỳ (1–4 lần/năm), quy mô 200–1000 người/event. Khách hàng triển khai là ban tổ chức sự kiện hoặc phòng quan hệ cựu sinh viên.

## Dữ liệu hệ thống sẽ động đến

| Dữ liệu | Mức độ nhạy cảm |
|---|---|
| Tên, email, ngành học, kỹ năng, mục tiêu networking | Trung bình |
| Vector embedding từ profile (512-dim) | Thấp (đã anonymized) |
| Lịch sử kết nối, ai gặp ai tại event | Trung bình |
| Event agenda, speaker bios (knowledge base RAG) | Thấp |
| Câu icebreaker đã sinh (log LLM output) | Thấp |

## 3 ràng buộc enterprise lớn nhất

1. **Dữ liệu cá nhân sinh viên/alumni** — phải tuân thủ PDPA Việt Nam, không được chuyển ra nước ngoài nếu không có consent rõ ràng. Ảnh hưởng trực tiếp đến việc dùng GPT-4o (OpenAI server ở Mỹ).
2. **Latency SLA ≤5s tại event** — event diễn ra offline, mạng không ổn định (wifi hội trường đông người), nếu LLM call timeout thì UX hỏng ngay tại thời điểm quan trọng nhất.
3. **Traffic spike theo event schedule** — 90% traffic dồn vào 2–3 giờ cao điểm của event, còn lại gần như 0. Hạ tầng phải scale nhanh và không tốn tiền khi idle.

## Lựa chọn triển khai: Cloud API (với data residency control)

**Lý do 1 — Phù hợp với traffic pattern:**
Event diễn ra không liên tục, dùng Cloud (AWS ECS + OpenAI API) cho phép scale-to-zero khi không có event, tránh chi phí GPU idle. On-prem sẽ tốn capex GPU server nhưng utilization chỉ ~5% thời gian.

**Lý do 2 — Tốc độ triển khai và chất lượng LLM:**
GPT-4o cho chất lượng icebreaker tốt hơn hẳn model self-hosted cùng size. Với quy mô MVP (vài trăm user/event), chi phí API hoàn toàn chấp nhận được. Self-hosted chỉ có lợi khi > 1M tokens/ngày — ngưỡng này chỉ đạt khi có hàng chục event lớn đồng thời.

## Trade-off phải chấp nhận
- Dữ liệu profile gửi lên OpenAI → cần anonymize trước khi đưa vào prompt (thay tên thật bằng ID), thêm điều khoản DPA với OpenAI
- Phụ thuộc vào uptime của OpenAI API → cần fallback (xem W4-reliability.md)
