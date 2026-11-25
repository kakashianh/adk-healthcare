# 03. Quy trình nghiệp vụ

## 3.1 Tiếp nhận hội thoại

1. Browser tạo USER_ID và SESSION_ID khi tải trang.
2. Người dùng gửi tin nhắn tới ADK API.
3. RootAgent lấy giờ WIB, chọn lời chào.
4. Prompt hỏi bệnh nhân mới hay cũ.
5. Root phân loại ý định và delegate.

Đây là chỉ dẫn prompt, chưa phải state machine được backend cưỡng chế.

## 3.2 Đăng ký bệnh nhân mới

~~~mermaid
sequenceDiagram
    actor P as Patient
    participant A as RegistrationAgent
    participant T as Registration Tool
    participant F as FHIR Store
    P->>A: Yêu cầu đăng ký
    A->>P: Thu thập thông tin
    A->>P: Xác nhận dữ liệu
    A->>T: Gửi dữ liệu chuẩn hóa
    T->>F: GET Patient theo phone/email
    F-->>T: Kết quả kiểm tra trùng
    T->>F: GET Patient mới nhất để sinh MRN
    T->>F: POST Patient
    F-->>T: Patient id
    T-->>A: Kết quả và MRN
    A-->>P: Thông báo
~~~

Prompt thu thập tên, ngày sinh, giấy tờ, tôn giáo, giới tính, điện thoại, email và nhiều trường nhân khẩu học. Chữ ký registrasi_pasien_baru yêu cầu nhiều trường hơn prompt mô tả; cần đồng bộ schema và hội thoại.

## 3.3 Tra cứu hồ sơ bệnh nhân

- Agent yêu cầu email hoặc điện thoại.
- Prompt hướng dẫn lowercase email và chuẩn hóa điện thoại quốc tế.
- dapatkan_data_pasien gọi verifikasi_pasien.
- FHIR query Patient theo email hoặc telecom.
- Nếu đúng một kết quả, trả tên, ngày sinh, MRN; nếu không có hoặc nhiều kết quả thì báo lỗi.

**Khoảng trống:** lookup bằng email/điện thoại chưa chứng minh quyền sở hữu kênh liên hệ. Không coi đây là xác thực đủ mạnh để mở PHI.

## 3.4 Tra cứu bác sĩ/lịch

1. Người dùng nhập tên bác sĩ hoặc poli.
2. Agent chuẩn hóa tên poli.
3. cari_jadwal_dokter truy vấn Practitioner hoặc PractitionerRole.
4. Đọc availableTime gồm daysOfWeek, availableStartTime, availableEndTime.
5. Sinh ngày thực hành trong 30 ngày.
6. Trả lịch định kỳ và các ngày cụ thể.

Tìm theo tên bác sĩ hiện lấy Practitioner đầu tiên; cần xử lý trường hợp trùng tên.

## 3.5 Tạo lịch hẹn

~~~mermaid
sequenceDiagram
    actor P as Patient
    participant R as CreateAppointmentRoot
    participant V as Patient Lookup
    participant C as CreateAppointmentAgent
    participant F as FHIR Store
    P->>R: Muốn đặt lịch
    R->>P: Email hoặc điện thoại
    R->>V: Tìm hồ sơ
    V->>F: GET Patient
    F-->>V: Patient data
    V-->>R: patient_info
    R->>C: Patient info + yêu cầu
    C->>F: Tìm Practitioner/Role
    C->>P: Xác nhận bác sĩ, poli, giờ
    P-->>C: Xác nhận
    C->>F: Kiểm tra lịch và POST Appointment
    F-->>C: Appointment id
    C-->>P: Xác nhận + queue number
~~~

Appointment kéo dài 30 phút. Input thời gian là ISO-8601; tool kiểm tra ngày/giờ trong availableTime; prompt giới hạn 30 ngày. Cần sửa kiểm tra ngày quá khứ: code đổi start_time sang UTC trước khi so sánh với ngày WIB.

## 3.6 Kiểm tra lịch sắp tới

- User cung cấp email/điện thoại.
- Workflow lấy patient_info.
- periksa_janji_temu hiện gọi verifikasi_pasien theo tên/MRN/ngày sinh; agent phải map patient_info đúng tham số.
- FHIR query Appointment theo patient, booked và date >= hôm nay.
- Tool lấy một lịch gần nhất, _include Patient/Practitioner, tra PractitionerRole và tính queue number.
- Agent trình bày bác sĩ, poli, ngày giờ và số thứ tự.

## 3.7 Tư vấn sức khỏe

MedicalAdviceAgent hỏi từng câu về thời gian, tần suất, triệu chứng đi kèm, bệnh nền, dị ứng, thuốc và xử trí trước đó. Khi đủ dữ liệu, agent đưa thông tin phổ thông và dùng MedicalAdviceSearchAgent để tìm chuyên khoa.

**Yêu cầu an toàn bổ sung:** sàng lọc dấu hiệu cấp cứu trước tư vấn thường quy. Khi có dấu hiệu nguy hiểm, dừng workflow thông thường và hướng dẫn liên hệ cấp cứu tại địa phương; không để LLM trì hoãn trợ giúp.

## 3.8 Trạng thái lỗi cần chuẩn hóa

- Patient không tồn tại hoặc nhiều hồ sơ khớp.
- FHIR timeout, 401/403, 429, 5xx.
- Practitioner/PractitionerRole không tồn tại.
- Lịch không khả dụng hoặc bị đặt đồng thời.
- POST Appointment thành công nhưng response bị mất.
- Input ngày giờ sai hoặc thiếu timezone.
- Dữ liệu đăng ký thiếu, sai định dạng hoặc trùng định danh.
