# 01. Phạm vi và yêu cầu

## 1.1 Mục tiêu

ADK Healthcare là trợ lý hội thoại hướng đến bệnh nhân của bệnh viện giả định RS Sehat Selalu. LLM hiểu ý định và delegate tới agent chuyên biệt; các thao tác bệnh nhân/lịch hẹn được thực hiện qua Python tools gọi Google Cloud Healthcare API theo FHIR.

Mục tiêu quan sát được:
- Hỗ trợ hỏi thông tin bệnh viện và bác sĩ.
- Hướng dẫn đăng ký hồ sơ bệnh nhân.
- Tra cứu lịch bác sĩ.
- Tạo và kiểm tra lịch hẹn.
- Cung cấp thông tin sức khỏe phổ thông cho tình huống không khẩn cấp.

## 1.2 Actors

| Actor | Mục tiêu | Giới hạn thiết kế |
|---|---|---|
| Bệnh nhân mới | Tìm dịch vụ, tạo hồ sơ | Không truy cập hồ sơ người khác |
| Bệnh nhân hiện hữu | Tra cứu/đặt lịch | Chỉ truy cập dữ liệu cá nhân sau xác minh mạnh |
| Khách | Tra cứu thông tin công khai | Không truy cập PHI |
| Vận hành | Cấu hình và giám sát | IAM theo least privilege |

Hiện tại chưa thấy lớp phân quyền actor riêng trong source code.

## 1.3 Functional requirements hiện có

| ID | Chức năng | Hiện trạng |
|---|---|---|
| FR-01 | Điều phối hội thoại | Root ADK agent |
| FR-02 | Tra cứu thông tin bệnh viện | SearchAgent + Vertex AI Search |
| FR-03 | Liệt kê bác sĩ/chuyên khoa | PractitionerRole + Practitioner |
| FR-04 | Tra cứu lịch trong 30 ngày | cari_jadwal_dokter |
| FR-05 | Tư vấn sức khỏe không khẩn cấp | MedicalAdviceAgent |
| FR-06 | Đăng ký bệnh nhân | Tạo FHIR Patient |
| FR-07 | Tìm hồ sơ theo email/điện thoại | dapatkan_data_pasien |
| FR-08 | Tạo lịch hẹn | Tạo FHIR Appointment |
| FR-09 | Kiểm tra lịch sắp tới | Truy vấn Appointment |
| FR-10 | Chat và voice UI | chatbot.html |

## 1.4 Ngoài phạm vi hiện tại

Không xem là đã triển khai: OTP/MFA, đăng nhập tài khoản, quản trị nhân viên y tế, xác nhận lịch bởi nhân viên, hủy/đổi lịch, thanh toán, gửi SMS/email, quản lý bảo hiểm/claim, truy xuất diagnosis, tích hợp HIS ngoài FHIR store.

## 1.5 Quy tắc nghiệp vụ hiện có

- Timezone được hard-code WIB (UTC+7).
- Lịch thực hành được sinh trong khoảng 30 ngày.
- Appointment có thời lượng cố định 30 phút.
- Prompt giới hạn đặt lịch trong 30 ngày tới.
- Prompt yêu cầu tên poli chuẩn dạng “Poli …” và tên họ bác sĩ khi tìm lịch.
- Patient lưu MRN, định danh, liên hệ, địa chỉ, giới tính, ngày sinh và extension tùy biến.

## 1.6 NFR mục tiêu

| Nhóm | Yêu cầu |
|---|---|
| An toàn lâm sàng | Sàng lọc dấu hiệu cấp cứu; không thay thế bác sĩ |
| Bảo mật | Xác thực/ủy quyền trước PHI; log tối thiểu |
| Toàn vẹn | Chống double-booking, retry an toàn |
| Khả dụng | Timeout, retry có giới hạn, health checks |
| Hiệu năng | Giới hạn token và kết quả FHIR |
| Bảo trì | Tách domain, adapter, prompt và config |
| Kiểm thử | Unit, contract, integration, safety, E2E |
| Tuân thủ | Xác định luật và chính sách theo vùng triển khai |

NFR là mục tiêu thiết kế; chưa thấy SLO, benchmark hoặc bằng chứng tuân thủ trong repository.
