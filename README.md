# 📸 RedCup Media Engine
**A stateless media delivery system decoupling application logic from storage.**

## 🏗 Architecture
- **Storage:** MinIO (S3-compatible) cluster for binary assets (Images/Video).
- **Caching:** Redis L2 cache for API responses, reducing DB hits by ~95% on hot content.
- **Server:** Gunicorn with multi-threaded workers handling concurrent upload streams.

## 🔧 Integration
- **Internal Wiring:** Services communicate exclusively over the `redcup_net` bridge (No NAT hairpinning).
- **Auto-Provisioning:** Startup scripts automatically initialize buckets and policies via `boto3` checks.

```mermaid
graph LR
    User[User] -->|POST /upload| Web[Flask Web (Gunicorn)]
    Web -->|1. Save Temp| Disk[Local Volume]
    Web -->|2. Enqueue Job| Redis[Redis Queue]
    
    subgraph Background_Processing
        Worker[RQ Worker] -->|3. Dequeue| Redis
        Worker -->|4. Process & Upload| MinIO[MinIO Object Store]
        Worker -->|5. Update DB| DB[(SQL Database)]
    end
```
## ⚡ Performance Engineering

**Concurrency Fix:**
- Migrated from single-threaded Flask to Gunicorn with 4 workers, 2 threads each '(-w 4 --threads 2)', handling 8 concurrent streams while background workers process IO-heavy tasks.

**Media Tunnel Strategy:**
The application streams bytes directly from MinIO to clients instead of exposing MinIO or redirecting users.
- **Mechanism:** 's3.get_object' streams bytes to the response wrapper.
- **Optimization:** 'Cache-Control: public, max-age=31536000' headers allow downstream CDNs and browsers to cache assets for 1 year, eliminating repeated server load.
## 🏗️ Component Breakdown


| Component   | Tech Stack       | Role                                                                          |
| ----------- | ---------------- | ----------------------------------------------------------------------------- |
| API Gateway | Flask + Gunicorn | Handles routing, authentication, and metadata retrieval                       |
| Task Broker | Redis            | Manages upload queue and caches API JSON responses (L2 cache)                 |
| Worker Unit | Python RQ        | Stateless consumer for S3 uploads and image resizing                          |
| Storage     | MinIO (S3)       | "Cold" object storage, accessed via internal Docker DNS (`http://minio:9000`) |
