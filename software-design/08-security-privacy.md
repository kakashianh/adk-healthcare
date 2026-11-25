# 08. Bảo mật và quyền riêng tư

## 8.1 Dữ liệu nhạy cảm

Patient name, date of birth, MRN, phone, email, identity document, address, medical symptoms, appointment history và chat content có thể là PHI/PII. Cần phân loại và xử lý theo luật áp dụng tại quốc gia triển khai.

## 8.2 Hiện trạng quan sát được và vấn đề

| Hạng mục | Hiện trạng | Rủi ro |
|---|---|---|
| Authentication | Frontend tạo USER_ID client-side | ID giả mạo được |
| Patient lookup | Email hoặc phone query | Không chứng minh quyền sở hữu |
| Authorization | Prompt hướng dẫn xác minh | Prompt không phải security boundary |
| CORS | Allow all origins | Mở rộng bề mặt tấn công |
| Logging | callback_logging log prompt/response | Có thể lộ PHI |
| Debug | tools.py print input và FHIR payload | Lộ dữ liệu cá nhân |
| Secrets | Environment variables | Cần đảm bảo không commit .env |
| IAM | ADC cloud-platform scope | Phạm vi quyền có thể rộng |
| Session | SQLite local | Rủi ro lưu trữ và cô lập session |
| Frontend | API URL cố định | Khó cấu hình và rotate endpoint |

## 8.3 Security controls bắt buộc trước production

1. **Identity proofing**: OTP hoặc identity provider; không dựa riêng email/phone nhập vào.
2. **Authorization**: backend xác nhận principal được truy cập Patient resource cụ thể; mọi tool PHI phải nhận context đã xác thực.
3. **Least privilege**: service account chỉ có quyền cần thiết trên FHIR store, Vertex AI và logging.
4. **Secrets**: Secret Manager; không commit .env, token hoặc credential.
5. **Transport/storage**: HTTPS/TLS; encryption at rest; quản lý key theo policy.
6. **Audit**: log ai truy cập resource nào, mục đích, thời điểm, kết quả; tránh ghi nội dung PHI.
7. **Redaction**: mask MRN, phone, email, DOB và token trong logs/traces.
8. **Session isolation**: user không thể đọc session của người khác; TTL và logout.
9. **Abuse protection**: rate limit, bot controls, input validation và alert.
10. **Data governance**: retention, consent, deletion, breach response và data residency.

## 8.4 Clinical safety

- Nêu rõ AI không thay thế bác sĩ và không chẩn đoán xác định.
- Phát hiện dấu hiệu cấp cứu, hướng dẫn gọi dịch vụ cấp cứu ngay.
- Không đưa khuyến nghị thuốc/liều nguy cơ cao nếu không có nguồn và quy trình phù hợp.
- Không suy diễn kết quả xét nghiệm hoặc diagnosis không có trong FHIR.
- Luôn phân biệt dữ liệu hệ thống với nội dung do mô hình suy luận.
- Có escalation path tới nhân viên y tế.

## 8.5 Threat scenarios

| Threat | Tác động | Giảm thiểu |
|---|---|---|
| Patient enumeration | Phát hiện hồ sơ qua email/phone | Response không tiết lộ quá mức, rate limit, verified identity |
| IDOR/session hijack | Xem lịch người khác | Principal-bound authorization |
| Prompt injection | Agent gọi tool ngoài ý muốn | Tool policy, allowlist, output validation |
| PHI leakage in logs | Lộ dữ liệu y tế | Redaction, access control, retention |
| Double booking | Đặt trùng lịch | Atomic reservation/transaction |
| Credential compromise | Truy cập FHIR trái phép | Least privilege, rotation, audit |
| Malicious HTML/model output | XSS | Text rendering, sanitize/escape |
