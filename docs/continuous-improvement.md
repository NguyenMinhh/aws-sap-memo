## AWS SAP-C02 - Domain 3: Continuous Improvement for Existing Solutions (Advanced Memo + Keyword)

## 0) Viết tắt thường gặp (đọc là hiểu)

- **SLO (Service Level Objective):** mục tiêu chất lượng dịch vụ.
- **SLI (Service Level Indicator):** chỉ số đo SLO.
- **p95/p99:** percentile latency (đuôi phân phối, phản ánh user experience thực).
- **CUR (Cost and Usage Report):** báo cáo cost/usage chi tiết.
- **IaC (Infrastructure as Code):** quản trị hạ tầng bằng code.
- **SSM (AWS Systems Manager):** automation/runbook vận hành.

---

## 1) Scope nhanh (đề hay hỏi)

- Tối ưu reliability, performance, security, cost cho hệ đang chạy
- Tăng observability + operational maturity
- Giảm toil bằng automation và policy-driven ops

**Keywords:** right-sizing, bottleneck, p95/p99 latency, error budget, operational excellence, auto-remediation.

---

## 2) Keyword -> đáp án nhanh

- **high cost, low utilization** -> Compute Optimizer + RI/Savings Plans + rightsizing *(sample: EC2 chạy 15% CPU nhiều tháng)*
- **latency spikes hard to trace** -> CloudWatch + X-Ray + trace correlation + structured logs *(sample: p99 tăng giờ cao điểm nhưng CPU bình thường)*
- **frequent manual fixes** -> SSM Automation + EventBridge + runbook-as-code *(sample: mỗi tuần ops restart service thủ công)*
- **configuration drift** -> AWS Config rules + auto-remediation *(sample: SG bị mở 0.0.0.0/0 lặp lại)*
- **security visibility gaps** -> Security Hub + GuardDuty + Inspector + Detective *(sample: thiếu góc nhìn lỗ hổng runtime + findings phân tán)*
- **database bottleneck** -> query/index tuning + caching + read replicas/partitioning strategy *(sample: query chậm làm tăng API latency)*

---

## 3) Framework cải tiến thực chiến (giải thích dễ nhớ)

### A. Reliability loop (Giữ hệ ổn định)
- **Định nghĩa SLI/SLO:** ví dụ availability 99.9%, error rate < 0.1%.
- **Theo dõi error budget:** nếu “xài” budget quá nhanh thì dừng rollout tính năng, ưu tiên fix stability.
- **Prioritize theo impact:** sửa lỗi ảnh hưởng nhiều user/doanh thu trước.
- **Memo nhớ nhanh:** *Không có SLO thì không biết khi nào hệ “đang fail”.*

### B. Performance loop (Làm hệ nhanh thật, không cảm tính)
- **Baseline:** chụp số hiện tại (p50/p95/p99, throughput, DB query time).
- **Profile:** tìm bottleneck thật (app code, DB, network, external dependency).
- **Optimize:** xử lý đúng cổ chai (index, cache, connection pool, async pattern).
- **Load test + compare:** test lại và so before/after để xác nhận hiệu quả.
- **Memo nhớ nhanh:** *Đừng nhìn average, ưu tiên p95/p99 vì đó là trải nghiệm user tệ nhất.*

### C. Cost loop (Giảm chi phí nhưng không phá SLA)
- **Detect waste:** instance nhàn rỗi, storage dư, data transfer bất thường.
- **Rightsize:** chọn lại size phù hợp usage thật, tránh overprovision.
- **Purchase optimization:** Savings Plans/Reserved Instances cho tải ổn định.
- **Anomaly detection:** cảnh báo khi cost tăng bất thường theo ngày/tuần.
- **Memo nhớ nhanh:** *Cost tối ưu bền vững = rightsize + model mua cam kết đúng + autoscaling chuẩn.*

### D. Security loop (Giảm rủi ro lặp lại)
- **Detect:** thu findings từ Security Hub/GuardDuty/Inspector.
- **Triage:** phân mức nghiêm trọng và SLA xử lý (Critical/High/Medium).
- **Remediate:** auto-remediation cho lỗi lặp (ví dụ đóng SG open public).
- **Prevent recurrence:** policy-as-code + guardrail để lỗi không quay lại.
- **Memo nhớ nhanh:** *Fix 1 lần bằng tay chỉ chữa cháy; policy + automation mới là chữa gốc.*

### Mini flow 1 dòng dễ thuộc
- **Measure -> Find bottleneck -> Fix -> Automate -> Verify -> Standardize**.
- **Tiếng Việt:** **Đo số liệu -> tìm cổ chai -> sửa đúng chỗ -> tự động hóa -> kiểm chứng kết quả -> chuẩn hóa vận hành**.

---

## 4) Services cần nhớ

- CloudWatch metrics/logs/alarms, X-Ray, CloudTrail *(quan sát & điều tra root cause)*
- Config, Systems Manager, Automation documents *(governance + auto-remediation)*
- Compute Optimizer, Cost Explorer, CUR, Trusted Advisor *(tối ưu chi phí)*
- Security Hub, GuardDuty, Inspector, Detective *(security posture tập trung)*
- Auto Scaling, ECS/EKS/Lambda tuning controls *(tối ưu hiệu năng vận hành)*

---

## 5) Câu khó hay gặp + đáp án định hướng

1) **CPU ổn nhưng latency p99 cao thất thường.**
- **Đáp án hướng:** trace theo request path + DB/network dependency analysis + cache/tuning.
  **Ghi nhớ:** p99 cao nhưng CPU ổn -> nghẽn thường ở dependency, không phải compute.

2) **Cost tăng sau khi scale, cần giảm mà không giảm SLA.**
- **Đáp án hướng:** rightsize + mixed purchase options + autoscaling policy tuning.
  **Ghi nhớ:** giảm cost bền vững = rightsize + mua cam kết đúng mức.

3) **Drift liên tục do manual changes ngoài IaC.**
- **Đáp án hướng:** enforce IaC pipeline + Config rules + auto-remediation.
  **Ghi nhớ:** drift tái diễn -> giải pháp phải policy + automation, không chỉ nhắc nhở.

4) **Incident lặp lại cùng nguyên nhân.**
- **Đáp án hướng:** runbook automation + postmortem action tracking + alarm quality improvement.
  **Ghi nhớ:** incident lặp lại = thiếu cơ chế đóng vòng học hỏi.

5) **Security findings quá nhiều, team xử lý chậm.**
- **Đáp án hướng:** centralized triage + severity routing + automated containment.
  **Ghi nhớ:** findings nhiều -> cần ưu tiên theo severity và auto-contain ca nguy cấp.

---

## 6) Checklist trả lời câu SAP (siêu nhanh + có đáp án)

1) **Mục tiêu optimize chính là gì: cost, performance, reliability hay security?**
- **Keyword:** high cloud bill, underutilized resources -> **Đáp án:** rightsize + Savings Plans/RI + autoscaling policy tuning.
- **Keyword:** p99 latency spikes, timeout, slow API -> **Đáp án:** tracing + bottleneck analysis + targeted tuning.
- **Keyword:** recurring incidents / unstable service -> **Đáp án:** SLO/error budget + reliability fixes + runbook automation.
- **Keyword:** too many findings / compliance pressure -> **Đáp án:** centralized security triage + automated remediation.

2) **Đã có metric/chứng cứ kỹ thuật trước khi sửa chưa?**
- **Keyword:** no baseline, cảm giác chậm -> **Đáp án:** thiết lập baseline (p50/p95/p99, error rate, cost/day) trước optimize.
- **Keyword:** cannot prove improvement -> **Đáp án:** đo before/after bằng cùng workload test.

3) **Giải pháp có tự động hóa và rollback không?**
- **Keyword:** repetitive manual operations -> **Đáp án:** SSM Automation + EventBridge + runbook-as-code.
- **Keyword:** risky change in production -> **Đáp án:** canary/blue-green + rollback criteria rõ ràng.

4) **Giải pháp có làm xấu đi security/compliance không?**
- **Keyword:** cut cost by removing controls -> **Đáp án:** loại bỏ phương án đó; giữ security baseline bắt buộc.
- **Keyword:** config drift recurring -> **Đáp án:** IaC enforcement + Config conformance + auto-remediation.

5) **Có cơ chế ngăn lỗi lặp lại không?**
- **Keyword:** same incident repeats -> **Đáp án:** postmortem actions + policy-as-code + automation closure.
- **Keyword:** alerts noisy, team ignored -> **Đáp án:** tune actionable alerts theo severity + ownership rõ ràng.

> Chốt đáp án: phải có đủ **metric rõ ràng + cải tiến đo được + tự động hóa + không phá SLA/security**.

---

## 7) Trap keywords & anti-patterns

- **Sai:** Chỉ tối ưu cost mà bỏ reliability.
  **Nên nhớ:** giảm chi phí nhưng làm tăng downtime là trade-off xấu trong đề SAP.
  **Ví dụ:** tắt Multi-AZ để tiết kiệm -> outage 1 AZ là gián đoạn dịch vụ.

- **Sai:** Tối ưu khi chưa có baseline/telemetry.
  **Nên nhớ:** không đo trước thì không chứng minh được hiệu quả sau.
  **Ví dụ:** nói “đã nhanh hơn” nhưng không có p95/p99 before/after.

- **Sai:** Tạo quá nhiều alarm không actionable.
  **Nên nhớ:** alert phải có owner + hành động rõ.
  **Ví dụ:** on-call nhận 500 cảnh báo/ngày rồi bỏ qua cả cảnh báo critical.

- **Sai:** Rollout optimize lớn mà không có rollback rehearsal.
  **Nên nhớ:** thay đổi càng lớn càng cần canary/rollback criteria.
  **Ví dụ:** đổi autoscaling policy sai làm latency tăng đột biến, không quay đầu kịp.

- **Sai:** Không gắn owner/SLO cho service.
  **Nên nhớ:** không owner = không ai chịu trách nhiệm khi SLO fail.
  **Ví dụ:** incident lặp lại nhiều sprint vì action items không ai đóng.

---

## 8) Quick answer templates (1 dòng)

- **"Need cost optimization safely"** -> `Rightsize + Savings Plans + keep HA boundaries`.
- **"Need faster incident response"** -> `Better telemetry + SSM auto-remediation`.
- **"Need stop config drift"** -> `IaC enforcement + Config conformance + remediation`.
- **"Need improve app latency"** -> `Trace bottlenecks + optimize p95/p99 path`.
