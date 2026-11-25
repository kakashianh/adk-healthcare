# 05. Thiết kế tích hợp

## 5.1 Google Cloud Healthcare API — FHIR

Base URL được ghép từ:
- GOOGLE_CLOUD_PROJECT
- FHIR_LOCATION
- FHIR_DATASET_ID
- FHIR_STORE_ID

Resource endpoint dạng:
projects/{project}/locations/{location}/datasets/{dataset}/fhirStores/{store}/fhir/{resource}

Authentication hiện tại dùng google.auth.default với cloud-platform scope; credential được refresh trước request.

## 5.2 FHIR resources

| Resource | Mục đích | Đọc/ghi hiện tại |
|---|---|---|
| Patient | Hồ sơ bệnh nhân | Search, create |
| Practitioner | Thông tin bác sĩ | Search/read via include |
| PractitionerRole | Chuyên khoa và lịch thực hành | Search |
| Appointment | Lịch hẹn | Search, create |

Các FHIR search parameter quan sát được: Patient?email, Patient?telecom, Patient?identifier, Patient?given, Patient?birthdate, Practitioner?name:contains, PractitionerRole?practitioner, PractitionerRole?specialty:text, Appointment?actor, Appointment?status, Appointment?date.

## 5.3 FHIR request adapter hiện tại

make_fhir_request hỗ trợ GET và POST, đặt Authorization Bearer token và Content-Type application/fhir+json. Request không khai báo timeout; exception RequestException được log và trả None.

**Đề xuất adapter production:**
- Dùng requests.Session hoặc async HTTP client có connection pooling.
- Bắt buộc timeout connect/read.
- Retry chỉ cho lỗi transient và method an toàn; POST cần idempotency.
- Chuẩn hóa FHIR OperationOutcome thành mã lỗi domain.
- Thêm pagination (_count, Bundle.link next), correlation ID và audit event.
- Không log payload PHI hoặc access token.
- Kiểm tra response resourceType/schema trước khi xử lý.

## 5.4 Vertex AI Search

GeneralSearchAgent khởi tạo VertexAiSearchTool với Data Store resource path gồm GOOGLE_CLOUD_PROJECT và DATASTORE_ID, location global. Dùng để trả lời thông tin bệnh viện, dịch vụ, bác sĩ và poli.

Cần xác minh:
- Data Store ID có đúng project/region/collection.
- Nguồn dữ liệu được cập nhật và có owner.
- Nội dung index có chứa PHI hay không (khuyến nghị không đưa PHI vào corpus tìm kiếm công khai).
- Có trích dẫn/nguồn trả về để người dùng kiểm chứng.

## 5.5 Google GenAI / Vertex AI model

Tên model lấy từ MODEL, MODEL_LITE, MODEL_PRO. Root agent và các agent nghiệp vụ chủ yếu dùng model_pro; workflow dùng model_lite hoặc model_name cho tác vụ đơn giản. GOOGLE_GENAI_USE_VERTEXAI và GOOGLE_CLOUD_LOCATION được set từ environment trong agent.py.

Cần cấu hình rõ project, location, model allowlist, quota, timeout và fallback; không đưa secret vào source control.

## 5.6 Session và artifact

main.py cấu hình SQLite session URI sqlite:///./sessions.db và ARTIFACT_URI từ environment. Với Cloud Run nhiều instance, SQLite local không phải kho session bền vững dùng chung. Cần chọn managed database/session service và artifact storage bền vững.

## 5.7 Tích hợp chưa hiện thực

Không thấy adapter đang hoạt động cho insurance/claims, SMS/email notifications, payment gateway, HIS/EMR khác hoặc external identity provider. Các tích hợp này cần API contract, security review và sandbox test riêng.
