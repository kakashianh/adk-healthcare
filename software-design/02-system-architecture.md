# 02. Kiến trúc hệ thống

## 2.1 Kiến trúc logic

~~~mermaid
flowchart TD
    U[Patient / Visitor] --> UI[Browser Chat UI]
    UI --> API[FastAPI + ADK]
    API --> S[ADK Session SQLite]
    API --> R[Root Agent]
    R --> GS[Search Agent]
    R --> MA[Medical Advice Agent]
    R --> REG[Registration Agent]
    R --> CA[Create Appointment Agent]
    R --> CH[Check Appointment Agent]
    GS --> V[Vertex AI Search]
    MA --> FHIR[FHIR Tools]
    REG --> FHIR
    CA --> FHIR
    CH --> FHIR
    FHIR --> HC[Google Healthcare API]
    API --> WEB[Static templates]
~~~

## 2.2 Các lớp kiến trúc

1. **Presentation** — chatbot.html là giao diện chat; index.html là landing page.
2. **Application/API** — main.py khởi tạo get_fast_api_app, cấu hình SQLite session, CORS và artifact service; mount static files.
3. **Agent orchestration** — root_agent điều phối sub-agents theo promp_instruction_v4.
4. **Domain tools** — tools.py chứa logic Patient, Practitioner, lịch và Appointment.
5. **External services** — Google GenAI/Vertex AI, Vertex AI Search, Healthcare API FHIR và ADC credentials.
6. **Delivery** — Docker, Cloud Build, Artifact Registry và Cloud Run.

## 2.3 Agent inventory

| Agent | Trách nhiệm |
|---|---|
| RootAgent | Chào hỏi, phân loại bệnh nhân mới/cũ, định tuyến |
| SearchAgent | Thông tin chung, danh sách/lịch bác sĩ |
| GeneralSearchTool | AgentTool bọc VertexAiSearchTool |
| MedicalAdviceAgent | Thu thập triệu chứng và tư vấn phổ thông |
| MedicalAdviceSearchAgent | Tìm chuyên khoa/bác sĩ |
| RegistrationAgent | Thu thập thông tin và tạo Patient |
| CreateAppointmentRootAgent | Điều phối lookup bệnh nhân và đặt lịch |
| CreateAppointmentAgent | Tìm lịch và tạo Appointment |
| CheckAppointmentRootAgent | Điều phối lookup và kiểm tra lịch |
| CheckAppointmentAgent | Truy vấn lịch gần nhất |

## 2.4 Luồng dữ liệu

- Browser gửi hội thoại tới ADK FastAPI app.
- ADK duy trì session qua SQLite; frontend tự sinh USER_ID và SESSION_ID.
- Root agent gọi tools hoặc delegate tới sub-agent.
- FHIR tools lấy access token bằng Application Default Credentials và gọi FHIR REST API.
- SearchAgent truy vấn Vertex AI Search Data Store qua DATASTORE_ID.
- Agent tổng hợp kết quả trả về UI.

## 2.5 Trust boundaries

| Boundary | Dữ liệu | Kiểm soát cần có |
|---|---|---|
| Browser → API | Chat, user/session IDs | HTTPS, auth, rate limit, origin allowlist |
| Agent → Tool | Tham số bệnh nhân/lịch | Schema validation, authorization |
| Backend → FHIR | PHI | IAM least privilege, TLS, audit |
| Backend → Search | Query và corpus | Data Store access control |
| Runtime → Logs | Prompt, output, lỗi | Redaction, retention, restricted access |

## 2.6 Kiến trúc mục tiêu đề xuất

Tách tools.py thành patient, practitioner, appointment services và FHIR adapter; bổ sung auth/identity, config, observability. Mọi truy cập PHI phải được backend policy kiểm tra, không phụ thuộc prompt. Thay SQLite local bằng session store bền vững phù hợp multi-instance Cloud Run.

## 2.7 Rủi ro hiện trạng

- CORS đang cho phép mọi origin.
- SQLite nằm trong filesystem container, không bảo đảm persistence giữa instance/revision.
- tools.py ghép domain, HTTP, auth và response formatting.
- Đặt lịch và tính queue number chưa có transaction/locking.
- Prompt nói về xác minh nhưng chưa thấy enforcement độc lập ở backend.
