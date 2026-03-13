## Topic Deep Dive 5 - Observability & Continuous Optimization (Advanced SAP Memo)

## 0) Viết tắt thường gặp

- **SLI (Service Level Indicator):** chỉ số đo chất lượng dịch vụ (latency, error rate, availability).
- **SLO (Service Level Objective):** mục tiêu cần đạt của SLI.
- **MTTR (Mean Time To Recovery):** thời gian trung bình khôi phục sau sự cố.
- **MTTD (Mean Time To Detect):** thời gian trung bình phát hiện sự cố.
- **RCA (Root Cause Analysis):** phân tích nguyên nhân gốc.
- **p95/p99 latency:** độ trễ ở nhóm user bị ảnh hưởng nặng nhất.

---

## 1) Mục tiêu ôn thi SAP-C02 cho topic này

- Biết chọn dịch vụ quan sát hệ thống phù hợp câu hỏi.
- Biết đọc keyword để chốt đáp án tối ưu performance/cost/reliability.
- Biết phân biệt giải pháp "chữa cháy" và giải pháp "đóng vòng cải tiến".

**Keywords hay gặp:** observability, bottleneck, high latency, intermittent errors, noisy alerts, right-sizing, auto-remediation.

---

## 2) Keyword -> đáp án nhanh (kèm sample)

- **high p99 latency, CPU not high** -> CloudWatch + X-Ray + dependency tracing *(sample: API chậm ngẫu nhiên do DB query/third-party timeout)*
- **cost spike without clear reason** -> CUR + Cost Explorer + anomaly detection *(sample: data transfer tăng đột biến sau release)*
- **too many repeated incidents** -> SSM runbook automation + EventBridge triggers *(sample: service crash lặp mỗi tuần)*
- **config drift causes outages** -> AWS Config rules + auto-remediation *(sample: security group bị mở public lại sau mỗi deploy)*
- **slow triage during incidents** -> centralized logs + trace correlation + runbook SOP *(sample: mất 2 tiếng mới xác định service lỗi)*

---

## 3) Framework thực chiến (đề dài rất hay hỏi)

### A. Detect (phát hiện nhanh)
- Thu thập metrics/logs/traces đủ depth.
- Tạo alerts theo SLO thay vì chỉ ngưỡng hạ tầng.
- Giảm noise bằng alert tuning (severity + ownership).

### B. Diagnose (chẩn đoán đúng)
- Correlate logs + traces + metrics theo request path.
- Ưu tiên xem p95/p99 và dependency map.
- Phân biệt symptom (CPU cao) và root cause (DB lock/query plan).

### C. Improve (cải tiến có kiểm chứng)
- Tối ưu đúng bottleneck: cache/index/pool/concurrency/autoscaling.
- Dùng canary/blue-green cho thay đổi rủi ro.
- So sánh before/after bằng cùng tải kiểm thử.

### D. Prevent recurrence (ngăn lỗi lặp)
- Viết runbook + automation remediation.
- Enforce IaC + config conformance.
- Theo dõi action items sau postmortem đến khi đóng.

**Memo 1 dòng:** *Không đo được thì không tối ưu được; không tự động hóa thì lỗi sẽ quay lại.*

---

## 4) Services cần nhớ (khi nào chọn)

- **CloudWatch (Metrics/Logs/Alarms):** nền quan sát chính cho AWS workloads.
- **AWS X-Ray:** truy vết request xuyên service, tìm bottleneck dependency.
- **CloudTrail:** audit hành động API phục vụ RCA/security.
- **AWS Config:** phát hiện và sửa drift theo rule.
- **Systems Manager (Automation/Runbook):** giảm manual toil.
- **Compute Optimizer + Cost Explorer + CUR:** tối ưu chi phí dựa số liệu thực.
- **Trusted Advisor:** khuyến nghị best-practice nhanh.

---

## 5) Câu thực tế hay gặp + đáp án định hướng

1) **User than chậm nhưng dashboard CPU/RAM đẹp.**
- **Đáp án hướng:** trace end-to-end bằng X-Ray + kiểm tra DB/query và external dependencies.
- **Ghi nhớ:** hạ tầng khỏe chưa chắc request path khỏe.

2) **Chi phí tăng 30% sau 1 tháng, team không rõ nguyên nhân.**
- **Đáp án hướng:** phân tích CUR + Cost Explorer theo service/tag/account + bật anomaly alerts.
- **Ghi nhớ:** tối ưu cost phải bắt đầu từ visibility chuẩn.

3) **Sự cố giống nhau lặp lại nhiều lần.**
- **Đáp án hướng:** runbook automation + event-driven remediation + postmortem tracking.
- **Ghi nhớ:** xử lý tay nhanh nhưng không bền, automation mới bền.

4) **Alert quá nhiều, on-call bị mù tín hiệu.**
- **Đáp án hướng:** tuning alert theo SLO/severity, gắn owner rõ cho mỗi alarm.
- **Ghi nhớ:** ít alert nhưng actionable tốt hơn nhiều alert vô ích.

5) **Release mới thường làm latency dao động.**
- **Đáp án hướng:** canary deployment + rollback criteria + compare before/after metrics.
- **Ghi nhớ:** thay đổi có kiểm soát luôn an toàn hơn big-bang deploy.

---

## 6) Checklist trả lời câu SAP (format giống DR)

1) **Vấn đề chính là detect, diagnose hay optimize?**
- **Keyword:** cannot find issue quickly -> ưu tiên **observability baseline** (metrics/logs/traces).
- **Keyword:** know issue but slow fix -> ưu tiên **runbook + automation**.

2) **Câu hỏi nhấn mạnh metric nào? average hay tail latency?**
- **Keyword:** user complaint intermittent -> ưu tiên **p95/p99 + trace path**, không chỉ average.
- **Keyword:** random timeout -> kiểm tra **dependency timeout/retry policy**.

3) **Có yêu cầu giảm chi phí nhưng giữ SLA không?**
- **Keyword:** underutilized -> **rightsizing + Savings Plans/RI**.
- **Keyword:** cost spike unknown -> **CUR + anomaly detection**.

4) **Có yêu cầu ngăn lỗi lặp không?**
- **Keyword:** repeated incident/drift -> **Config + IaC enforcement + SSM auto-remediation**.
- **Keyword:** audit/compliance -> cần **CloudTrail/Config evidence**.

5) **Giải pháp có đo được hiệu quả không?**
- **Keyword:** no baseline/no proof -> thêm **before/after measurement + load test**.
- **Keyword:** risky change -> thêm **canary + rollback plan**.

> Chốt đáp án: ưu tiên phương án có đủ **visibility + root-cause path + automation + measurable improvement**.

---

## 7) Trap keywords (bẫy thường gặp)

- **Sai:** Thấy chậm là scale compute ngay khi chưa rõ bottleneck.
  **Nên nhớ:** scale sai chỗ chỉ tăng cost, không giải quyết root cause.
  **Ví dụ:** nguyên nhân do query lock nhưng tăng CPU app không giúp gì.

- **Sai:** Nhìn CPU average thấp rồi kết luận hệ ổn.
  **Nên nhớ:** user đau ở p95/p99, không phải average đẹp.
  **Ví dụ:** average 40ms nhưng p99 2s vào giờ cao điểm.

- **Sai:** Có dashboard nhưng thiếu correlation logs-traces-metrics.
  **Nên nhớ:** thiếu correlation thì RCA chậm và dễ đoán sai.
  **Ví dụ:** mất 2 giờ chỉ để xác định service nào trong chuỗi gây timeout.

- **Sai:** Tối ưu cost bằng cách bỏ HA/control security.
  **Nên nhớ:** đề SAP thường ưu tiên cân bằng cost + reliability + security.
  **Ví dụ:** bỏ encryption/logging để tiết kiệm vài % cost nhưng fail compliance.

- **Sai:** Không gắn ownership cho alert/runbook.
  **Nên nhớ:** cảnh báo không owner = không ai xử lý triệt để.
  **Ví dụ:** alarm critical nổ nhiều ngày mà không team nào nhận trách nhiệm.