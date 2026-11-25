# 06. Mô hình dữ liệu

## 6.1 Tổng quan

FHIR là mô hình dữ liệu lâm sàng trung tâm. Ứng dụng hiện thao tác trực tiếp với JSON resource, không thấy lớp ORM/domain model riêng.

~~~mermaid
erDiagram
    PATIENT ||--o{ APPOINTMENT_PARTICIPANT : attends
    PRACTITIONER ||--o{ PRACTITIONER_ROLE : practices
    PRACTITIONER ||--o{ APPOINTMENT_PARTICIPANT : attends
    PRACTITIONER_ROLE }o--|| PRACTITIONER : references
    APPOINTMENT ||--|{ APPOINTMENT_PARTICIPANT : has
~~~

## 6.2 Patient

Các trường được ghi/đọc:
- identifier: MRN và loại/số giấy tờ.
- name: given (tên và có thể tên tengah), family (họ).
- gender, birthDate.
- telecom: phone và email.
- address: text, use=home; nơi lahir disimpan lewat extension.
- extension tùy biến: agama, pendidikan, pekerjaan, golongan_darah, kewarganegaraan.
- maritalStatus.text.

Một số URL extension hiện dùng example.info; cần thay bằng canonical StructureDefinition URL của tổ chức và công bố profile FHIR.

## 6.3 Practitioner

Dùng name gồm prefix, given, family, suffix để hiển thị tên bác sĩ. Tìm kiếm dùng name:contains. Code hiện lấy kết quả đầu tiên khi tìm theo tên; thiết kế mục tiêu phải yêu cầu disambiguation nếu nhiều kết quả.

## 6.4 PractitionerRole

Liên kết practitioner với specialty và availableTime:
- daysOfWeek
- availableStartTime
- availableEndTime

Tool dùng specialty.text để lọc poli và availableTime để tính lịch. Cần làm rõ hỗ trợ timezone, effectivePeriod, notAvailable, location và nhiều role cho cùng một bác sĩ.

## 6.5 Appointment

Appointment được tạo với:
- status = booked
- start/end dạng ISO timestamp
- participant Patient và Practitioner, mỗi participant status accepted
- duration hiện tính cố định 30 phút.

Chưa thấy lưu rõ serviceType/specialty/location trong Appointment payload. Poli được truy vấn gián tiếp qua PractitionerRole, điều này có thể gây sai lệch khi bác sĩ làm nhiều poli.

## 6.6 MRN

get_next_mrn đọc Patient mới cập nhật nhất, parse numeric suffix và tăng 1. Đây là cách sinh số tuần tự không an toàn khi concurrent registration. Đề xuất:
- Sinh MRN bằng sequence/transaction ở datastore có unique constraint.
- Đặt identifier.system ổn định và unique.
- Retry khi conflict; không dựa vào sort của Patient collection.

## 6.7 Data lifecycle và retention

| Data | Mục đích | Đề xuất |
|---|---|---|
| Patient PHI | Quản lý hồ sơ | Retention theo luật/chính sách y tế |
| Appointment | Điều phối khám | Lưu theo retention clinical record |
| Chat/session | Hội thoại và ngữ cảnh | TTL rõ ràng, xóa theo chính sách |
| Logs/traces | Vận hành | Không ghi PHI mặc định; retention ngắn |
| Search corpus | Thông tin bệnh viện | Versioning và review định kỳ |

## 6.8 Data quality và validation

- Chuẩn hóa email lowercase và điện thoại E.164.
- Validate ngày sinh, gender code, loại giấy tờ và mã quốc tịch.
- Kiểm tra duplicate theo định danh có tính uniqueness phù hợp.
- Chuẩn hóa specialty/poli bằng terminology catalog.
- Validate FHIR profile trước POST.
- Pagination cho Bundle thay vì chỉ dùng total và entry đầu tiên.
