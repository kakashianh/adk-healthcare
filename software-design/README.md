# ADK Healthcare — Software Design

Bộ tài liệu phân rã thiết kế phần mềm theo các góc nhìn độc lập. File [Overview-plan.md](Overview-plan.md) chỉ chứa metadata và mục lục.

## Cách đọc

- Bắt đầu từ Overview-plan.md để nắm metadata.
- Đọc 01–03 để hiểu phạm vi, kiến trúc và luồng bệnh nhân.
- Đọc 04–07 để hiểu agent, tích hợp, dữ liệu và giao diện.
- Đọc 08–10 để đánh giá bảo mật, triển khai, chất lượng và kế hoạch cải tiến.

## Quy ước diễn giải

- **Hiện trạng**: xác nhận trực tiếp từ mã nguồn repository tại thời điểm 2026-11-25.
- **Suy luận**: diễn giải từ cấu trúc và luồng gọi.
- **Đề xuất**: thay đổi cần thực hiện, chưa phải chức năng hiện hữu.
- **Cần xác minh**: phụ thuộc cấu hình Google Cloud, FHIR data hoặc runtime.

## Nguồn mã nguồn chính

| Thành phần | Đường dẫn |
|---|---|
| FastAPI entrypoint | main.py |
| Root agent | healthcare-patient-id/agent.py |
| Prompt điều phối | healthcare-patient-id/prompts.py |
| FHIR/business tools | healthcare-patient-id/tools.py |
| General search | healthcare-patient-id/sub_agents/general_search/agent.py |
| Medical advice | healthcare-patient-id/sub_agents/medical_advice/agent.py |
| Registration | healthcare-patient-id/sub_agents/new_patient_registration/agent.py |
| Appointment | healthcare-patient-id/sub_agents/create_appointment/ và check_upcoming_appointments/ |
| Web UI | templates/chatbot.html, templates/index.html |
| Build/deploy | Dockerfile, cloudbuild.yaml, requirements.txt |

## Giới hạn không được giả định

Insurance benefit, claim status, diagnosis retrieval, BPJS eligibility, OTP, patient login, appointment cancellation/rescheduling, payment, notification delivery và clinician-facing workflows không được xem là chức năng đang hoạt động. Một số mục chỉ xuất hiện trong prompt cũ hoặc code đã comment.
