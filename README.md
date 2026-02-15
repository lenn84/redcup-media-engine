# 📸 RedCup Media Engine
**A stateless media delivery system decoupling application logic from storage.**

## 🏗 Architecture
- **Storage:** MinIO (S3-compatible) cluster for binary assets (Images/Video).
- **Caching:** Redis L2 cache for API responses, reducing DB hits by ~95% on hot content.
- **Server:** Gunicorn with multi-threaded workers handling concurrent upload streams.

## 🔧 Integration
- **Internal Wiring:** Services communicate exclusively over the `redcup_net` bridge (No NAT hairpinning).
- **Auto-Provisioning:** Startup scripts automatically initialize buckets and policies via `boto3` checks.
