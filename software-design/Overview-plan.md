# Software Design — Metadata

| Thuộc tính | Giá trị |
|---|---|
| Sản phẩm | ADK Healthcare — Virtual Clinical Assistant |
| Repository | kakashianh/adk-healthcare |
| Branch phân tích | main |
| Ngày lập tài liệu | 2026-11-25 |
| Trạng thái | Baseline từ mã nguồn; đề xuất được đánh dấu riêng |
| Backend | Python 3.13, FastAPI, Google ADK, Uvicorn |
| LLM | Google ADK; MODEL, MODEL_LITE, MODEL_PRO từ environment |
| Search | Vertex AI Search |
| Clinical data | Google Cloud Healthcare API — FHIR store |
| Session | ADK session service, SQLite sessions.db |
| Frontend | HTML, Tailwind CDN, JavaScript, browser speech APIs |
| Deployment | Docker, Cloud Build, Artifact Registry, Cloud Run |
| Timezone nghiệp vụ | WIB (UTC+7) |
| Chức năng quan sát được | Tra cứu thông tin, tư vấn sức khỏe phổ thông, đăng ký bệnh nhân, tra cứu lịch bác sĩ, tạo/kiểm tra lịch hẹn |
| FHIR resources quan sát được | Patient, Practitioner, PractitionerRole, Appointment |
| Chủ sở hữu repository | kakashianh; fork từ ahmadluay9/adk-healthcare |

## Tài liệu thiết kế

1. [README — chỉ mục](README.md)
2. [01 — Phạm vi và yêu cầu](01-scope-requirements.md)
3. [02 — Kiến trúc hệ thống](02-system-architecture.md)
4. [03 — Quy trình nghiệp vụ](03-business-workflows.md)
5. [04 — Thiết kế AI agents](04-agent-design.md)
6. [05 — Tích hợp FHIR và Google Cloud](05-integration-design.md)
7. [06 — Mô hình dữ liệu](06-data-model.md)
8. [07 — Giao diện và API](07-interface-api-design.md)
9. [08 — Bảo mật và quyền riêng tư](08-security-privacy.md)
10. [09 — Triển khai và vận hành](09-deployment-operations.md)
11. [10 — Kiểm thử, rủi ro và lộ trình](10-testing-risks-roadmap.md)

> Quy ước: hiện trạng phải được xác nhận từ source code. Nội dung chưa triển khai được ghi là đề xuất hoặc cần xác minh, không được hiểu là tính năng đang hoạt động.
