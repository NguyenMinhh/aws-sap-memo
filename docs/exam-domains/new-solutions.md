## AWS SAP-C02 - Domain 2: Design for New Solutions (Advanced Memo + Keyword)

## 0) Viết tắt thường gặp (đọc là hiểu)

- **API (Application Programming Interface):** cổng giao tiếp ứng dụng.
- **ACID (Atomicity, Consistency, Isolation, Durability):** tính chất giao dịch DB quan hệ.
- **DLQ (Dead-Letter Queue):** queue chứa message xử lý lỗi.
- **CQRS (Command Query Responsibility Segregation):** tách luồng ghi/đọc.
- **MSK (Managed Streaming for Apache Kafka):** Kafka managed trên AWS.
- **DAX (DynamoDB Accelerator):** cache in-memory cho DynamoDB.

---

## 1) Scope nhanh (đề hay hỏi)

- Thiết kế kiến trúc mới: resilient, scalable, secure, cost-efficient
- Chọn compute/data/integration phù hợp workload
- Thiết kế global architecture, caching, asynchronous patterns
- Security-by-design ngay từ đầu

**Keywords:** greenfield, event-driven, microservices, low-latency global, stateless, decoupled, idempotent.

---

## 2) Keyword -> đáp án nhanh

- **burst traffic, decouple components** -> SQS/SNS/EventBridge + autoscaling consumers *(sample: flash sale tăng x10 traffic)*
- **orchestration workflow, retries, human approval** -> Step Functions *(sample: duyệt hoàn tiền nhiều bước có manual approval)*
- **API global low latency** -> CloudFront + Route 53/Global Accelerator + regional backends *(sample: user US/EU truy cập API với latency thấp)*
- **strict relational consistency** -> Aurora/RDS (Multi-AZ + read replicas) *(sample: hệ thanh toán cần ACID transaction)*
- **massive key-value, unpredictable scale** -> DynamoDB (+ DAX nếu read-heavy low latency) *(sample: session/profile app scale lớn khó đoán)*
- **search analytics log data** -> OpenSearch + S3 data lake pattern *(sample: tìm kiếm log ứng dụng theo thời gian thực)*
- **real-time ingestion stream** -> Kinesis/MSK + Lambda/ECS consumers *(sample: clickstream/events liên tục)*
- **serverless rapid delivery** -> API Gateway + Lambda + DynamoDB/EventBridge *(sample: MVP cần ra nhanh, ít ops)*

---

## 3) Kiến trúc mẫu hay ra đề (dễ học, dễ hình dung)

### A. Event-driven microservices
- API/service publish event qua EventBridge/SNS.
- Consumer tách riêng, retry + DLQ.
- Idempotency key để tránh double processing.
- **Hình dung khi đọc đề:** request vào nhanh, xử lý nặng chạy phía sau theo hàng đợi/sự kiện.
- **Dấu hiệu nhận diện:** `burst`, `decouple`, `asynchronous`, `no data loss in queue`.

### B. CQRS/read-optimized
- Write vào transactional DB.
- Read model qua cache/search/index store.
- Tối ưu read latency mà không phá write consistency.
- **Hình dung khi đọc đề:** 1 bên chuyên ghi chuẩn ACID, 1 bên chuyên đọc cực nhanh cho dashboard/search.
- **Dấu hiệu nhận diện:** `read-heavy`, `reporting slows OLTP`, `optimize reads`.

### C. Multi-region active-active API
- Route 53 latency/health check routing.
- Stateless compute ở nhiều region.
- Data replication phù hợp RPO/RTO target.
- **Hình dung khi đọc đề:** user ở đâu vào region gần đó, region lỗi thì traffic tự chuyển.
- **Dấu hiệu nhận diện:** `global users`, `low latency worldwide`, `region failover`.

### D. Data lake + analytics foundation
- Raw/curated zone trên S3.
- Event/ETL bằng Glue/Lambda/EMR.
- Governance + encryption + lifecycle.
- **Hình dung khi đọc đề:** dữ liệu đổ về hồ chung, rồi xử lý thành lớp sạch cho BI/ML.
- **Dấu hiệu nhận diện:** `analytics at scale`, `multiple data sources`, `cost-effective storage`.

---

## 4) Services cần nhớ (và khi nào chọn)

- **ALB/NLB/API Gateway:** front door theo giao thức và use case.
- **Lambda/ECS/EKS/EC2:** chọn theo ops model, startup latency, control level.
- **SQS/SNS/EventBridge:** queue/pub-sub/event bus.
- **Aurora/RDS vs DynamoDB:** relational transactions vs massive scale key-value.
- **ElastiCache:** giảm read latency, offload DB.
- **CloudFront/Global Accelerator:** edge acceleration + global routing.
- **WAF/Shield/KMS/Secrets Manager:** security baseline.

---

## 5) Câu khó hay gặp + đáp án định hướng

1) **Checkout bị spike lớn theo giờ, cần không mất request và tách xử lý hậu kỳ.**
- **Đáp án hướng:** SQS buffer + autoscaling workers + DLQ + idempotency.
  **Ghi nhớ:** spike traffic -> queue làm lớp giảm sốc.

2) **Global users, API phải low latency và chịu được region failover.**
- **Đáp án hướng:** multi-region stateless + Route 53 latency/failover + replicated data.
  **Ghi nhớ:** global + failover => compute stateless trước, data replication theo sau.

3) **Hệ transaction cần ACID mạnh nhưng báo cáo nặng làm chậm OLTP.**
- **Đáp án hướng:** Aurora/RDS cho OLTP + read replica/analytics pipeline tách riêng.
  **Ghi nhớ:** OLTP và analytics không nên tranh tài nguyên cùng một DB chính.

4) **Workflow nhiều bước, cần retry có điều kiện + manual approval.**
- **Đáp án hướng:** Step Functions orchestration thay vì chaining Lambda tự xử lý tay.
  **Ghi nhớ:** nhiều bước + retry/timeout -> nghĩ Step Functions.

5) **Yêu cầu mở rộng nhanh nhưng team ops nhỏ.**
- **Đáp án hướng:** ưu tiên serverless managed services (Lambda, DynamoDB, EventBridge).
  **Ghi nhớ:** team nhỏ + cần ra nhanh = managed/serverless-first.

---

## 6) Trap keywords & anti-patterns

- **Sai:** Thấy từ “simple” rồi chọn monolith single-AZ cho workload cần scale/resilience.
  **Nên nhớ:** “simple” không đồng nghĩa “single point of failure”.
  **Ví dụ:** Flash sale app mà chỉ 1 DB instance/1 AZ -> nghẽn hoặc sập giờ cao điểm.

- **Sai:** Bài toán “real-time” nhưng chọn batch ETL thuần.
  **Nên nhớ:** Real-time thường cần stream/event-driven path.
  **Ví dụ:** Fraud detection cần phản hồi gần tức thì -> batch theo giờ là trễ nghiệp vụ.

- **Sai:** Hiểu “exactly once” theo nghĩa tuyệt đối mà bỏ idempotency.
  **Nên nhớ:** Nhiều hệ phân tán dùng at-least-once, nên app phải idempotent.
  **Ví dụ:** Thanh toán nhận lại cùng event 2 lần -> không có idempotency có thể trừ tiền 2 lần.

- **Sai:** Dùng synchronous chain quá dài giữa nhiều service.
  **Nên nhớ:** Chain dài tăng latency + failure cascade.
  **Ví dụ:** service C chậm làm A/B cùng timeout dù business logic đơn giản.

- **Sai:** Có queue nhưng không có DLQ/retry/backoff.
  **Nên nhớ:** Không có error path thì production incident rất khó xử.
  **Ví dụ:** downstream lỗi 10 phút -> message lỗi mất dấu, không replay được.

---

## 7) Security-by-design checklist

1. Encrypt at rest/in transit (KMS/TLS).
2. Secrets qua Secrets Manager/Parameter Store, không hardcode.
3. Least privilege IAM roles cho từng workload.
4. WAF/rate limiting cho public APIs.
5. Audit trail + logging tập trung ngay từ đầu.

---

## 8) Checklist chốt đáp án (siêu nhanh)

1. Workload sync hay async? có cần decouple?
2. Latency/throughput target là gì?
3. Consistency yêu cầu mức nào?
4. Failure domain AZ hay region?
5. Team vận hành mạnh hay cần managed/serverless?

> Đáp án tốt nhất thường là: **đúng workload pattern + đúng data model + đúng ops model**.

---


### Checklist có keyword + đáp án + memo

- **Keyword:** burst traffic, decouple, no data loss in queue -> **Đáp án:** SQS + autoscaling consumers + DLQ.
- **Keyword:** workflow nhiều bước, retry/approval -> **Đáp án:** Step Functions (orchestration).
- **Keyword:** global low latency API -> **Đáp án:** CloudFront/Global Accelerator + multi-region stateless compute.
- **Keyword:** strict transaction consistency -> **Đáp án:** Aurora/RDS (ACID), không chọn key-value thuần.
- **Memo chốt:** Chọn theo workload pattern trước (sync/async, consistency), rồi mới chọn service.

## 9) Quick answer templates (1 dòng)

- **"High burst + reliability"** -> `SQS + autoscaling consumers + DLQ`.
- **"Global low-latency API"** -> `CloudFront/GA + multi-region stateless + Route 53`.
- **"ACID transaction workload"** -> `Aurora/RDS Multi-AZ (+ read replicas)`.
- **"Fast team, low ops burden"** -> `Serverless-first architecture`.
