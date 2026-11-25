# 09. Triển khai và vận hành

## 9.1 Container

Dockerfile dùng python:3.13-slim, cài requirements.txt, copy source vào /app và chạy Uvicorn trên 0.0.0.0:8080. PORT được đặt 8080. requirements.txt hiện khai báo google-adk, uvicorn, fastapi, python-dotenv, requests, aiohttp.

**Đề xuất:** pin dependency versions, lockfile, non-root user, image scanning, SBOM và build reproducibility.

## 9.2 Cloud Build pipeline

cloudbuild.yaml gồm:
1. Docker build image tag theo COMMIT_SHA.
2. Push image lên Artifact Registry region _REGION.
3. Deploy Cloud Run service _SERVICE_NAME.
4. Mặc định region asia-southeast2 và service healthcare-patient-service.
5. Logging Cloud Logging only.

Deploy hiện dùng --allow-unauthenticated; phù hợp chatbot public chỉ khi API và PHI operations có auth/authorization riêng. Không thể xem public ingress là bằng chứng API bảo vệ dữ liệu.

## 9.3 Environment configuration

Biến môi trường quan sát được:
- GOOGLE_CLOUD_PROJECT
- GOOGLE_CLOUD_LOCATION
- GOOGLE_GENAI_USE_VERTEXAI
- MODEL, MODEL_LITE, MODEL_PRO
- FHIR_LOCATION, FHIR_DATASET_ID, FHIR_STORE_ID
- DATASTORE_ID
- ARTIFACT_URI
- PORT

Nên có config schema kiểm tra biến bắt buộc khi startup, phân biệt secret/non-secret và fail-fast nếu thiếu.

## 9.4 Runtime dependencies

| Dependency | Vai trò | Kiểm tra vận hành |
|---|---|---|
| Cloud Run | HTTP runtime | instance, concurrency, CPU/memory, scaling |
| Healthcare API | FHIR | auth, latency, quota, error rate |
| Vertex AI | LLM | quota, latency, model availability |
| Vertex AI Search | Search corpus | index health, freshness |
| Session store | Conversation state | persistence, backup, TTL |
| Cloud Logging | Logs | redaction, retention, alert |

## 9.5 Health checks và observability đề xuất

- Liveness: process/ASGI server còn hoạt động.
- Readiness: config hợp lệ và dependencies cần thiết sẵn sàng.
- Metrics: request count/latency, LLM latency/token, tool error rate, FHIR status, appointment success/conflict, session failures.
- Tracing: correlation ID xuyên API → agent → tool → FHIR.
- Alert: error budget burn, FHIR 5xx/403, model quota, elevated latency, suspicious patient lookup.
- Dashboard không hiển thị PHI.

## 9.6 Backup và recovery

- Backup/restore session store và artifact store theo RPO/RTO đã thống nhất.
- FHIR backup/retention theo Healthcare API capabilities và policy.
- Kiểm thử restore định kỳ.
- Giữ image tags bất biến theo commit SHA và có rollback revision.
- Runbook cho FHIR outage, LLM outage, deploy failure và suspected data breach.

## 9.7 Deployment checklist

- [ ] Service account least privilege.
- [ ] Secret Manager references đã cấu hình.
- [ ] FHIR store, dataset, region đúng.
- [ ] Vertex AI model và Data Store đúng project/region.
- [ ] CORS allowlist.
- [ ] Authentication và authorization đã bật.
- [ ] Persistent session/artifact store.
- [ ] Logs đã redacted.
- [ ] Health checks, alerts và rollback đã kiểm thử.
