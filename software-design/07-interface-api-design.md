# 07. Thiết kế giao diện và API

## 7.1 Frontend

templates/chatbot.html cung cấp:
- Khung chat, message bubbles, ô nhập và nút gửi.
- Nút tạo session mới.
- Toggle Text-to-Speech qua SpeechSynthesis.
- Speech recognition qua SpeechRecognition/webkitSpeechRecognition, locale id-ID.
- Trạng thái online/offline.
- Kết nối tới Cloud Run URL được hard-code trong JavaScript.
- APP_NAME, USER_ID và SESSION_ID được khai báo ở client.

templates/index.html là landing page có phần giới thiệu, dịch vụ, bác sĩ, liên hệ và form yêu cầu lịch hẹn. Form hiển thị trong HTML không chứng minh backend xử lý form.

## 7.2 Backend

main.py khởi tạo FastAPI app bằng google.adk.cli.fast_api.get_fast_api_app:
- agents_dir = ROOT_DIR.
- session_service_uri = sqlite:///./sessions.db.
- allow_origins = ["*"].
- artifact_service_uri lấy từ ARTIFACT_URI.
- ADK web UI tắt (web=False).
- GET / redirect tới /index.html.
- StaticFiles mount tại root path.

Các endpoint hội thoại chi tiết do ADK FastAPI app cung cấp; cần kiểm tra OpenAPI thực tế của phiên bản google-adk đã cài trước khi viết client contract cứng.

## 7.3 API contract mục tiêu

Không thay thế endpoint ADK đang dùng; đề xuất một lớp BFF/versioned API để ổn định giao diện:

| Method | Path mục tiêu | Mô tả |
|---|---|---|
| POST | /api/v1/chat/sessions | Tạo session sau auth |
| POST | /api/v1/chat/sessions/{id}/messages | Gửi message |
| GET | /api/v1/chat/sessions/{id} | Lấy trạng thái/lịch sử được phép |
| GET | /api/v1/health/live | Liveness |
| GET | /api/v1/health/ready | Readiness với dependency checks |

Các endpoint trên là đề xuất, chưa được xác nhận tồn tại trong source.

## 7.4 UI states

- Initial / connecting.
- Ready / online.
- Sending / waiting response.
- Error / retry available.
- Empty input / disabled send.
- Voice recording / speech unsupported.
- New session.

Cần đảm bảo trạng thái lỗi không làm mất message người dùng và không gửi lại mutation nếu request trước có thể đã thành công.

## 7.5 API/UI security

- Không hard-code production API URL trong source; dùng build-time/runtime config.
- Không tin USER_ID do client tự sinh làm danh tính đã xác thực.
- Dùng secure session cookie hoặc bearer token có TTL ngắn.
- CSRF protection nếu dùng cookie; strict CORS allowlist.
- Rate limit, input length limit và abuse protection.
- Escape/render output an toàn; tránh innerHTML với nội dung model chưa sanitize.
- Voice API cần thông báo rõ browser có thể gửi audio/transcript qua dịch vụ của trình duyệt.
