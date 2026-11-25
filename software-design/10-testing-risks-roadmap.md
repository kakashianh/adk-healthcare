# 10. Kiểm thử, rủi ro và lộ trình

## 10.1 Chiến lược kiểm thử

| Level | Nội dung | Ví dụ |
|---|---|---|
| Unit | Pure functions và validation | Normalize phone, parse ISO date, MRN format |
| Tool contract | Tool schema và output | Missing field, duplicate Patient, FHIR OperationOutcome |
| FHIR integration | Sandbox FHIR store | Patient create/search, PractitionerRole query |
| Agent evaluation | Intent routing và prompt | Registration vs appointment vs advice |
| Safety | Clinical guardrails | Chest pain/red flags, self-harm, emergency escalation |
| Security | Auth/session/PHI | IDOR, enumeration, prompt injection, log redaction |
| E2E | Browser-to-FHIR flow | Register → verify → book → check |
| Load/resilience | Concurrent and dependency failures | Concurrent booking, 429, timeout, Cloud Run scale |

## 10.2 Acceptance scenarios quan trọng

1. Bệnh nhân mới đăng ký với dữ liệu hợp lệ và nhận MRN.
2. Email/phone trùng bị từ chối mà không lộ hồ sơ hiện hữu.
3. Bệnh nhân chưa xác thực không thể truy cập PHI dù prompt injection yêu cầu.
4. Nhiều Patient khớp cùng email/phone được xử lý an toàn.
5. Tên bác sĩ trùng yêu cầu người dùng chọn đúng practitioner.
6. Lịch nằm ngoài availableTime không thể đặt.
7. Thời gian quá khứ và ngoài cửa sổ 30 ngày bị từ chối ở service layer.
8. Hai request đồng thời cùng slot không thể tạo hai lịch.
9. Retry sau timeout không tạo Appointment duplicate.
10. Check appointment chỉ trả dữ liệu của principal sở hữu hồ sơ.
11. Dấu hiệu cấp cứu được chuyển hướng ngay.
12. Không có PHI trong logs, error traces hoặc analytics.

## 10.3 Rủi ro ưu tiên

| Mức | Rủi ro | Lý do |
|---|---|---|
| Critical | Email/phone lookup bị xem như xác thực | Có thể truy cập PHI trái phép |
| Critical | Tool authorization phụ thuộc prompt | LLM không phải policy enforcement |
| High | Double-booking/queue race | Không có atomic reservation |
| High | PHI bị ghi vào logs | Print/debug và callback logging |
| High | CORS wildcard và public service | Tăng attack surface |
| High | SQLite local trên Cloud Run | Session có thể mất/không chia sẻ |
| Medium | MRN sequence race | Duplicate identifier khi đăng ký đồng thời |
| Medium | FHIR pagination chưa đầy đủ | Bỏ sót dữ liệu |
| Medium | Timezone mismatch | Kiểm tra ngày/giờ sai |
| Medium | Prompt/tool schema drift | Workflow thất bại hoặc dữ liệu sai |

## 10.4 Lộ trình đề xuất

### P0 — An toàn và quyền truy cập
- Thêm identity verification thực sự, principal-bound authorization.
- Chặn PHI tools nếu chưa xác thực.
- Loại bỏ PHI khỏi logs và debug prints.
- Thay wildcard CORS, kiểm tra session isolation.
- Sàng lọc cấp cứu trong medical advice.

### P1 — Tính toàn vẹn nghiệp vụ
- Reservation atomicity/idempotency cho Appointment.
- MRN sequence an toàn và unique.
- Sửa timezone/date validation.
- Đồng bộ prompt, agent state và tool contracts.
- Xử lý pagination và FHIR OperationOutcome.

### P2 — Production readiness
- Persistent session store.
- Config validation, pinned dependencies, health checks.
- Metrics/tracing/alerts và runbooks.
- Test suite FHIR sandbox và agent evaluation.

### P3 — Mở rộng chức năng có kiểm soát
- Hủy/đổi lịch.
- Notification service.
- Insurance/claims/diagnosis chỉ sau khi xác định nguồn dữ liệu, quyền truy cập và clinical governance.
- Dashboard nhân viên y tế với RBAC/audit.

## 10.5 Quyết định cần xác nhận

- Quốc gia/khu vực pháp lý và chính sách PHI áp dụng.
- Cơ chế xác thực bệnh nhân được chấp thuận (OTP, portal, identity provider).
- FHIR version, profiles, terminology và identifier systems chính thức.
- Chính sách appointment slot, duration, queue number và conflict.
- Thời gian lưu chat/session/log.
- Yêu cầu SLA, RPO/RTO và chi phí vận hành.
- Corpus Vertex AI Search được phép chứa loại dữ liệu nào.
