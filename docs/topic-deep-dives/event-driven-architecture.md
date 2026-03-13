## Topic Deep Dive 4 - Event-Driven Architecture for SAP-C02 (Advanced Memo + Keyword)

## 0) Viết tắt thường gặp

- **EDA (Event-Driven Architecture):** kiến trúc hướng sự kiện.
- **SQS (Simple Queue Service):** hàng đợi message để decouple và buffer traffic.
- **SNS (Simple Notification Service):** pub/sub fan-out notifications.
- **DLQ (Dead-Letter Queue):** queue chứa message xử lý lỗi.
- **FIFO (First In, First Out):** bảo toàn thứ tự + deduplication.
- **Idempotency:** xử lý lặp nhiều lần nhưng kết quả nghiệp vụ không bị nhân đôi.

---

## 1) Scope ôn thi SAP-C02 cho topic này

- Giải bài toán burst traffic, decoupling, retry, fault isolation.
- Chọn đúng service cho pattern queue/pub-sub/event-bus.
- Thiết kế xử lý lỗi và đảm bảo không mất message quan trọng.

**Keywords hay gặp:** asynchronous, decoupling, fan-out, retry, backpressure, eventual consistency, replay.

---

## 2) Keyword -> đáp án nhanh (exam mapping + sample)

- **burst traffic, absorb spike** -> SQS buffer + consumer autoscaling *(sample: flash sale tăng request x10)*
- **one event to many subscribers** -> SNS hoặc EventBridge fan-out *(sample: order created -> billing + email + analytics)*
- **cross-account SaaS/event integration** -> EventBridge bus + rule-based routing *(sample: tích hợp event giữa nhiều account)*
- **strict ordering, no duplicate processing** -> SQS FIFO + deduplication + idempotency *(sample: xử lý thanh toán theo thứ tự)*
- **failed messages must be inspected/replayed** -> DLQ + replay workflow *(sample: message lỗi do dependency downtime)*
- **long-running orchestration with retries** -> Step Functions (không chaining Lambda tay) *(sample: workflow duyệt hoàn tiền nhiều bước)*

> Trả lời câu hỏi bạn nêu: **đúng**, khi gặp keyword `high cost, low utilization` thì thường nghiêng về **Compute Optimizer + RI/Savings Plans + rightsizing**; nhưng vẫn phải kiểm tra ràng buộc SLA/HA trước khi chốt đáp án.

---

## 3) Kiến trúc chuẩn hay ra đề (đọc đề là hình dung flow)

### A. Queue-based decoupling
- API nhận request nhanh, đẩy vào SQS.
- Worker scale theo queue depth.
- Tăng resiliency khi downstream chậm.
- **Hình dung khi đọc đề:** quầy nhận đơn phía trước luôn mở, bếp xử lý phía sau theo công suất.
- **Dấu hiệu nhận diện:** `spike traffic`, `buffer`, `decouple web tier and worker tier`.

### B. Event fan-out
- Producer publish 1 lần vào SNS/EventBridge.
- Nhiều consumer nhận độc lập.
- Tránh gọi sync chain dài.
- **Hình dung khi đọc đề:** 1 thông báo “đơn hàng mới” phát tới nhiều đội cùng lúc.
- **Dấu hiệu nhận diện:** `one event many targets`, `billing + notification + analytics`.

### C. Event choreography + orchestration mix
- Choreography cho event đơn giản, loosely coupled.
- Orchestration (Step Functions) cho flow nhiều bước, cần trạng thái/retry rõ.
- **Hình dung khi đọc đề:** việc nhỏ để các service tự phản ứng; việc phức tạp có “nhạc trưởng” điều phối.
- **Dấu hiệu nhận diện:** `multi-step workflow`, `conditional retry`, `human approval`.

### D. Outbox pattern (tránh mất event)
- Ghi DB transaction + outbox record.
- Process outbox để publish event.
- Giảm rủi ro "DB commit nhưng event thất lạc".
- **Hình dung khi đọc đề:** luôn ghi “sổ gửi tin” cùng lúc với transaction rồi mới gửi dần an toàn.
- **Dấu hiệu nhận diện:** `transactional consistency with event publishing`, `avoid lost events`.

---

## 4) Services cần nhớ (khi nào chọn)

- **SQS Standard:** throughput cao, at-least-once, thứ tự best-effort.
- **SQS FIFO:** cần thứ tự + deduplication.
- **SNS:** fan-out nhanh pub/sub.
- **EventBridge:** event bus linh hoạt, rule routing, tích hợp SaaS/AWS services.
- **Lambda/ECS workers:** consumer xử lý message.
- **Step Functions:** orchestration retry/timeout/human approval.

---

## 5) Câu thực tế hay gặp + đáp án định hướng

1) **Checkout bị timeout giờ cao điểm, DB chưa kịp xử lý.**
- **Đáp án hướng:** thêm SQS buffer + worker autoscaling + DLQ.
- **Ghi nhớ:** API cần trả lời nhanh, xử lý nặng chuyển async.

2) **Một event tạo order phải kích hoạt 5 hệ thống khác nhau.**
- **Đáp án hướng:** SNS/EventBridge fan-out thay vì gọi tuần tự.
- **Ghi nhớ:** fan-out = mở rộng dễ, giảm coupling.

3) **Message bị xử lý lặp làm trừ tiền 2 lần.**
- **Đáp án hướng:** FIFO (nếu cần thứ tự) + idempotency key ở consumer.
- **Ghi nhớ:** at-least-once delivery bắt buộc thiết kế idempotent.

4) **Workflow nhiều bước có retry khác nhau theo từng trạng thái.**
- **Đáp án hướng:** Step Functions state machine.
- **Ghi nhớ:** workflow càng phức tạp càng không nên "if/else" thủ công trong Lambda.

5) **Sự cố downstream làm backlog tăng liên tục.**
- **Đáp án hướng:** backpressure control + scaling policy + DLQ/retry strategy.
- **Ghi nhớ:** queue giải được spike, nhưng vẫn cần capacity planning.

---

## 6) Checklist trả lời câu SAP (format giống DR)

1) **Bài toán cần decouple hay vẫn cần sync real-time?**
- **Keyword:** tolerate async, eventual consistency -> chọn EDA (SQS/SNS/EventBridge).
- **Keyword:** strict immediate response with transaction lock-step -> cân nhắc sync + strong consistency.

2) **Có cần fan-out đến nhiều consumer không?**
- **Keyword:** one-to-many integration -> SNS/EventBridge.
- **Keyword:** single work queue processing -> SQS.

3) **Yêu cầu ordering/deduplication có bắt buộc không?**
- **Keyword:** strict order/no duplicate -> SQS FIFO + idempotency.
- **Keyword:** max throughput, order not strict -> SQS Standard.

4) **Xử lý lỗi và replay đã có chưa?**
- **Keyword:** failed messages must be retained -> DLQ + replay runbook.
- **Keyword:** repeated downstream failure -> retry policy + circuit breaker pattern.

5) **Flow có nhiều bước/state phức tạp không?**
- **Keyword:** branching/retry/human approval -> Step Functions.
- **Keyword:** simple event consumer -> Lambda/ECS worker đủ dùng.

> Chốt đáp án: phải có **decoupling đúng chỗ + error handling (DLQ/retry) + idempotency + scale strategy**.

---

## 7) Trap keywords (bẫy thường gặp)

- **Sai:** Có queue nhưng không DLQ/replay strategy.
  **Nên nhớ:** lỗi production sẽ không có đường điều tra/phục hồi message.
  **Ví dụ:** downstream down 30 phút, message lỗi thất lạc hoàn toàn.

- **Sai:** Dùng EDA nhưng vẫn gọi sync chain dài trong consumer.
  **Nên nhớ:** async bên ngoài nhưng sync chain bên trong vẫn tạo bottleneck.
  **Ví dụ:** worker nhận message rồi gọi 5 API tuần tự -> backlog vẫn tăng.

- **Sai:** Quên idempotency với at-least-once delivery.
  **Nên nhớ:** duplicate processing là bình thường trong distributed systems.
  **Ví dụ:** nhận lại event thanh toán gây cập nhật đơn hàng 2 lần.

- **Sai:** Chọn FIFO cho mọi workload dù không cần ordering.
  **Nên nhớ:** FIFO có trade-off throughput, không phải mặc định tốt hơn.
  **Ví dụ:** telemetry events số lượng lớn thường phù hợp SQS Standard hơn.

- **Sai:** Không monitor queue depth/age.
  **Nên nhớ:** backlog tăng âm thầm rồi bùng nổ thành outage.
  **Ví dụ:** queue age tăng 3 giờ nhưng team không biết để scale consumer kịp.