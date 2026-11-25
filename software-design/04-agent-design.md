# 04. Thiết kế AI Agents

## 4.1 Nguyên tắc

Google ADK được dùng để khai báo Agent/LlmAgent, SequentialAgent và AgentTool. Root agent là điểm điều phối; mỗi sub-agent có instruction, model, tools và output_key riêng. Model names lấy từ environment qua tools.py.

## 4.2 Root agent

RootAgent trong healthcare-patient-id/agent.py:
- Model: model_pro.
- Instruction: promp_instruction_v4.
- Sub-agents: RegistrationAgent, MedicalAdviceAgent, CheckAppointmentRootAgent, CreateAppointmentRootAgent, SearchAgent.
- Tools: lấy giờ/ngày, tìm lịch bác sĩ, liệt kê bác sĩ.

Prompt hiện tại hỏi bệnh nhân mới/cũ, công bố dịch vụ, rồi phân luồng. Có hướng dẫn giới hạn tư vấn ngoài ngữ cảnh y tế.

## 4.3 Agent catalog chi tiết

| Agent | Model | Tools/sub-agents | Output key |
|---|---|---|---|
| SearchAgent | model_pro | GeneralSearchTool, date, schedule, doctors | — |
| GeneralSearchTool | model_pro | VertexAiSearchTool | search_results |
| MedicalAdviceAgent | model_pro | MedicalAdviceSearchAgent | medical_advice_results |
| MedicalAdviceSearchAgent | model_pro | schedule, date, doctors | search_results |
| RegistrationAgent | model_pro | registrasi_pasien_baru, time | registration_result |
| CreateAppointmentRootAgent | model_pro | verification workflow, registration, appointment agent; schedule/doctors | — |
| CreateAppointmentAgent | model_pro | create appointment, date, schedule, doctors | appointment_confirmation |
| CheckAppointmentRootAgent | model_pro | verification workflow, registration, check agent | — |
| CheckAppointmentAgent | model_pro | periksa_janji_temu | upcoming_appointment |

## 4.4 Sequential verification workflows

CreateAppointment và CheckAppointment định nghĩa pipeline:
1. Agent thông báo sedang mencari data pasien.
2. Agent lookup theo email/nomor telepon bằng dapatkan_data_pasien.
3. Agent menyajikan patient_info (given/family/MRN).
4. Greeting agent menggunakan waktu lokal dan menanyakan tindakan selanjutnya.

Pipeline didefinisikan sebagai SequentialAgent. Namun, urutan agent tidak otomatis menjamin status autentikasi yang aman; hasil lookup harus diikat ke principal yang terautentikasi.

## 4.5 Tool contracts (ringkasan)

| Tool | Input utama | Output |
|---|---|---|
| dapatkan_waktu_sekarang | none | WIB time string |
| dapatkan_tanggal_hari_ini | none | tanggal Bahasa Indonesia |
| dapatkan_data_pasien | email atau nomor_telepon | nama, tanggal lahir, MRN |
| registrasi_pasien_baru | identitas, demografi, kontak | status dan MRN |
| daftar_semua_dokter | none | daftar dokter dan poli |
| cari_jadwal_dokter | nama_dokter dan/atau nama_poli | jadwal rutin dan tanggal praktik |
| periksa_janji_temu | nama/MRN/tanggal lahir | appointment terdekat |
| buat_janji_temu | dokter, poli, ISO datetime, identitas pasien | appointment confirmation |

## 4.6 Prompt engineering

Prompt sebaiknya:
- Memisahkan system policy, task instruction dan response style.
- Menetapkan output schema untuk setiap tool.
- Menentukan kapan harus meminta klarifikasi dan kapan harus berhenti.
- Tidak meminta LLM mengarang jadwal, ketersediaan atau status transaksi.
- Memperlakukan tool output sebagai data, bukan instruksi.
- Menggunakan Bahasa Indonesia/Inggris berdasarkan pilihan bahasa yang disimpan secara eksplisit.

## 4.7 Temuan konsistensi

- Prompts lama menyebut greeting_agent, verification workflow, insurance, claim, diagnosis dan BPJS agents yang tidak seluruhnya tersedia pada root agent aktif.
- Registration prompts không thu thập rõ toàn bộ tham số bắt buộc của registrasi_pasien_baru.
- Appointment check agent phải map patient_info sang signature periksa_janji_temu; kiểm thử contract cần xác nhận mapping.
- Một số import/biến model có thể thừa; cần static analysis và smoke test khi khởi động.
- Nên đưa quy tắc ngày giờ, xác minh và giới hạn lịch vào service layer, không chỉ prompt.

## 4.8 Guardrails mục tiêu

1. Safety triage trước medical advice.
2. PHI tool access chỉ sau backend authorization.
3. Validate tham số và output của tools.
4. Confirm user intent trước side effect như tạo Appointment.
5. Không lặp POST khi kết quả không chắc chắn; dùng idempotency key.
6. Redact dữ liệu nhạy cảm trong log và trace.
