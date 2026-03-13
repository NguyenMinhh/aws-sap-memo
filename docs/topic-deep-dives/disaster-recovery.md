## AWS SAP - Disaster Recovery Cheat Sheet (Memo + Keyword)

## 0) Viết tắt thường gặp (đọc là hiểu)

- **RTO (Recovery Time Objective):** thời gian tối đa chấp nhận gián đoạn.
- **RPO (Recovery Point Objective):** mức mất dữ liệu tối đa chấp nhận.
- **DR (Disaster Recovery):** khôi phục khi thảm họa cấp vùng/hệ thống.
- **HA (High Availability):** sẵn sàng cao (thường nội vùng/AZ).
- **PITR (Point-In-Time Recovery):** khôi phục về thời điểm bất kỳ trong khoảng retention.
- **CRR (Cross-Region Replication):** sao chép dữ liệu liên vùng (S3).

---

## 1) Chốt nhanh theo RTO/RPO/Cost

| Strategy | RTO | RPO | Cost | Khi nào dùng |
|---|---|---|---|---|
| Backup & Restore | Giờ -> ngày | Giờ | Thấp nhất | Workload không critical, chấp nhận downtime dài |
| Pilot Light | ~10s phút -> giờ | Phút -> giờ | Thấp | Muốn tiết kiệm, giữ lõi hệ thống chạy sẵn |
| Warm Standby | Vài phút | Phút | Trung bình | Cần failover nhanh, đã có môi trường DR chạy thu nhỏ |
| Multi-site Active-Active | Giây -> gần 0 | Gần 0 | Cao nhất | Mission-critical, global users, gần như không downtime |

**Rule nhớ nhanh:**
- RTO càng thấp + RPO càng thấp -> kiến trúc càng đắt và phức tạp.
- Đề bài nói **near-zero downtime** + **near-zero data loss** -> nghiêng mạnh về **Active-Active**.

---

## 2) Keyword -> Strategy (đáp án nhanh)

### Backup & Restore
- **keywords:** cheapest, non-critical, archive, cold site, restore later, tolerate hours/day downtime
- **nhận diện:** backup lên S3/Glacier, khi sự cố mới dựng lại infra và restore data

### Pilot Light
- **keywords:** cost optimized, minimal running infrastructure, core services always on, scale up on DR
- **nhận diện:** chỉ giữ DB/replication + thành phần lõi, app tier bật khi failover

### Warm Standby
- **keywords:** minutes recovery, fast recovery, scaled-down fully functional environment
- **nhận diện:** DR site đã chạy sẵn full stack nhưng nhỏ, failover bằng scale-out

### Active-Active
- **keywords:** no downtime, no data loss, global users, multi-region active traffic, highest availability
- **nhận diện:** cả 2 region cùng phục vụ traffic, đồng bộ dữ liệu chặt chẽ

---

## 3) AWS services hay đi kèm trong đề SAP

- **AWS Elastic Disaster Recovery (DRS):** block-level replication, failover/failback, giảm RTO cho server-based workloads.
- **AWS Backup:** policy-based backup, cross-account/cross-region backup copy, Vault Lock (compliance).
- **Amazon S3:** versioning, CRR/SRR, Object Lock (WORM), lifecycle -> Glacier classes.
- **Route 53:** health check + failover/latency/geolocation/weighted routing.
- **RDS/Aurora:** Multi-AZ (HA nội vùng), cross-region read replica/global database (DR liên vùng).
- **EBS snapshots / AMI:** nền tảng cho Backup & Restore/Pilot Light.
- **CloudFormation/Terraform:** IaC để dựng lại nhanh và nhất quán.

---

## 4) Keyword theo yêu cầu nghiệp vụ

- **Compliance / immutable backup / ransomware:** S3 Object Lock, Backup Vault Lock, cross-account backup.
- **Regional disaster:** multi-region design, data replication cross-region, Route 53 failover.
- **Data residency:** tách vùng dữ liệu theo khu vực, chọn region phù hợp pháp lý.
- **Global low latency:** active-active + latency-based routing + data replication strategy.
- **Cost-sensitive DR:** Backup & Restore hoặc Pilot Light.

---

## 5) SAP traps (rất hay gài)

- **HA != DR:** Multi-AZ chủ yếu cho HA, không thay thế DR liên vùng.
- **Pilot Light != Warm Standby:** Warm Standby nhanh hơn vì full environment đã chạy sẵn.
- **No data loss** thường yêu cầu replication rất chặt (có thể sync), trade-off latency/cost.
- **Backup có rồi chưa đủ:** phải có runbook, drill test, đo RTO/RPO thực tế.
- **Single region** không đáp ứng yêu cầu regional disaster.
- **Async replication != RPO 0:** cross-region thường async, vẫn có khả năng mất dữ liệu nhỏ.

---

## 6) Checklist trả lời câu SAP (siêu nhanh + có đáp án)

1) **Bài toán ưu tiên gì: RTO, RPO, cost, compliance?**
- **Keyword:** cheapest, non-critical, tolerate downtime -> **Backup & Restore**
- **Keyword:** cost optimized, minimal running core -> **Pilot Light**
- **Keyword:** minutes recovery, fast recovery -> **Warm Standby**
- **Keyword:** near-zero RTO/RPO, mission-critical -> **Active-Active**

2) **Mất AZ hay mất region?**
- **Keyword:** AZ failure, datacenter failure -> **Multi-AZ HA** (chưa chắc là DR)
- **Keyword:** region outage, regional disaster -> **Cross-Region DR** (Warm Standby / Active-Active)

3) **Cần active traffic 2 nơi hay chỉ standby?**
- **Keyword:** global users, active-active, low latency toàn cầu -> **Active-Active + Route 53 latency/failover**
- **Keyword:** standby environment, scale up when failover -> **Warm Standby / Pilot Light**

4) **Có yêu cầu no data loss thực sự không?**
- **Keyword:** zero data loss, RPO = 0 -> ưu tiên **đồng bộ dữ liệu chặt + Active-Active** (chấp nhận cost/độ phức tạp cao)
- **Keyword:** small acceptable data loss -> có thể dùng **Warm Standby / Pilot Light** tùy RTO

5) **Có cần tự động failover + test định kỳ + runbook không?**
- **Keyword:** automated failover, health checks -> **Route 53 failover, orchestration runbook**
- **Keyword:** compliance, audit, DR drill -> **bắt buộc có DR test định kỳ + runbook + bằng chứng đo RTO/RPO**

> Chốt đáp án: phải khớp đồng thời **RTO/RPO + scope thảm họa (AZ hay Region) + ngân sách + compliance**.

---

## 7) Câu trả lời mẫu 1 dòng (đi thi/đi phỏng vấn)

- **Nếu đề yêu cầu rẻ nhất, chấp nhận downtime dài:** chọn **Backup & Restore** (S3/Glacier + IaC restore).
- **Nếu đề yêu cầu tiết kiệm nhưng khôi phục nhanh hơn backup thuần:** chọn **Pilot Light**.
- **Nếu đề yêu cầu recovery trong vài phút:** chọn **Warm Standby**.
- **Nếu đề yêu cầu near-zero downtime + global traffic:** chọn **Multi-Region Active-Active**.
- **Nếu đề nhấn mạnh compliance/ransomware:** thêm **immutable backup (Object Lock/Vault Lock) + cross-account**.

## 8) Mini keyword dictionary (VN -> EN)

- **mất vùng / thảm họa vùng** -> regional outage / regional disaster
- **không mất dữ liệu** -> no data loss / RPO 0
- **khôi phục trong vài phút** -> minutes recovery / low RTO
- **chi phí tối ưu** -> cost optimized
- **kiểm thử DR định kỳ** -> DR drill / periodic failover testing

---

## 9) Bổ sung quan trọng (hay ra đề nhưng dễ bỏ sót)

- **RDS Multi-AZ = HA, không phải DR cross-region** (muốn DR liên vùng cần cross-region replica / global DB).
- **Read Replica không tự động failover như Multi-AZ** (đề hay gài giữa HA vs scale đọc vs DR).
- **DR cho stateful app**: phải nêu rõ chiến lược replication data trước, compute chỉ là phần dễ dựng lại.
- **DR đa tài khoản (multi-account)** thường được ưu tiên cho isolation/security/compliance.
- **Runbook + game day** là bằng chứng vận hành thật; không test định kỳ thì RTO/RPO trên giấy dễ sai.

## 10) Anti-pattern cần né trong đáp án

- **Sai:** Chỉ nói “bật Multi-AZ” khi đề hỏi **regional disaster**.
  **Vì sao sai:** Multi-AZ chủ yếu là HA nội vùng, region outage vẫn có thể ngừng dịch vụ.
  **Nên chọn:** Cross-Region DR (Warm Standby hoặc Active-Active tùy RTO/RPO).
  **Ví dụ nhanh:** Đề ghi “entire region unavailable” -> chọn Multi-AZ là trượt ý chính.

- **Sai:** Chỉ nói “backup hằng ngày” nhưng đề yêu cầu **minutes RPO**.
  **Vì sao sai:** Backup daily thường mất dữ liệu đến nhiều giờ/1 ngày.
  **Nên chọn:** Replication liên tục (CDC/read replica/global DB pattern).
  **Ví dụ nhanh:** RPO 5 phút -> backup 1 lần/ngày chắc chắn không đạt.

- **Sai:** Chọn Active-Active dù đề nhấn mạnh **cost-sensitive** mà không cần near-zero.
  **Vì sao sai:** Active-Active đắt và vận hành phức tạp hơn mức yêu cầu.
  **Nên chọn:** Warm Standby hoặc Pilot Light nếu RTO/RPO cho phép.
  **Ví dụ nhanh:** Đề yêu cầu “recover in minutes, optimize cost” -> Warm Standby thường hợp lý hơn.

- **Sai:** Không nhắc **DNS failover / traffic steering** khi đề yêu cầu tự động chuyển vùng.
  **Vì sao sai:** Có DR site mà không có traffic steering thì failover không tự diễn ra.
  **Nên chọn:** Route 53 failover/latency routing + health checks.
  **Ví dụ nhanh:** Region A down -> thiếu DNS failover thì người dùng vẫn đổ về endpoint hỏng.

---


## 11) Công thức chốt đáp án 10 giây

- **Region-level + minutes RTO + cost vừa** -> **Warm Standby**.
- **Region-level + gần 0 downtime/RPO** -> **Active-Active**.
- **Cost thấp nhất + chấp nhận downtime dài** -> **Backup & Restore**.
- **Cost tối ưu + muốn nhanh hơn restore thuần** -> **Pilot Light**.

---

## 12) DR theo từng service (RDS, DynamoDB, EC2, Lambda...)

### RDS / Aurora
- **HA nội vùng:** Multi-AZ.
- **DR liên vùng:** cross-region read replica / Aurora Global Database.
- **Keyword:** relational DB, transaction consistency, failover in minutes, cross-region database.
- **Trap:** chỉ Multi-AZ thì chưa giải quyết regional disaster.

### DynamoDB
- **DR liên vùng:** Global Tables (multi-region active-active), PITR backup.
- **Keyword:** serverless key-value, global write/read, low-latency global users, near-zero RPO.
- **Trap:** quên bật PITR/backup policy dù đã có global table.

### EC2 / VM-based workloads
- **DR nhanh:** AWS Elastic Disaster Recovery (DRS).
- **DR tiết kiệm:** AMI + EBS snapshot + IaC để rebuild.
- **Keyword:** lift-and-shift, block-level replication, failback required.

### Lambda / Serverless API
- **DR pattern:** deploy multi-region (active-active hoặc active-standby), dùng Route 53/API Gateway failover.
- **State cần DR:** dữ liệu phía sau (DynamoDB, S3, RDS) mới là trọng tâm.
- **Keyword:** stateless compute, rapid redeploy, regional failover for API.

### S3
- **DR/compliance:** versioning + CRR + Object Lock + lifecycle Glacier.
- **Keyword:** immutable backup, ransomware protection, long-term retention.

### EKS/ECS
- **DR:** multi-AZ cho HA; DR liên vùng bằng image registry replication + IaC + data replication.
- **Keyword:** container platform, rapid re-provision, region evacuation.

### ElastiCache / Redis
- **HA:** Multi-AZ with auto-failover.
- **DR liên vùng:** Global Datastore (Redis) hoặc backup/restore cross-region.
- **Trap:** cache-only không đủ, phải có hệ dữ liệu nguồn DR-ready.

### SQS/SNS/Event-driven
- **DR:** replicate topology bằng IaC; dữ liệu durable nên cần design idempotency/replay.
- **Keyword:** asynchronous decoupling, replay after failover.

---

## 13) Nên gộp hay tách section riêng?

- **Hiện tại nên gộp trong 1 file** (như section 12) vì bạn đang ôn theo memo nhanh, cần tra cứu tập trung.
- **Nên tách file riêng** khi mỗi service cần playbook chi tiết (runbook từng bước, lệnh, kiến trúc mẫu, cost estimate).
- Gợi ý ngưỡng tách: khi section service > 150-200 dòng hoặc bạn bắt đầu thêm sơ đồ/lab notes.


### Checklist có keyword + đáp án + memo (tóm tắt nhanh)

- **Keyword:** cheapest, tolerate long downtime -> **Đáp án:** Backup & Restore.
- **Keyword:** minimal running core + cost optimized -> **Đáp án:** Pilot Light.
- **Keyword:** recovery in minutes -> **Đáp án:** Warm Standby.
- **Keyword:** near-zero downtime + global users -> **Đáp án:** Active-Active multi-region.
- **Memo chốt:** Đọc đề theo thứ tự: scope thảm họa (AZ/Region) -> RTO/RPO -> ngân sách/compliance.
