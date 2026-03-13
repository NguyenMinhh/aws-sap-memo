## Topic Deep Dive - Cross-Domain Scenarios (SAP-C02)

## Mục tiêu
- Luyện câu dài tích hợp security + migration + operations + cost.
- Học theo pattern “constraint -> eliminate -> best-fit”.

---

## Cách dùng file này hiệu quả
- Mỗi scenario trả lời trong 60 giây theo 4 bước:
  1) Constraint chính,
  2) Loại đáp án nhiễu,
  3) Chọn best-fit,
  4) Nêu trap.

---

## 20 scenario liên-domain (chi tiết vừa đủ)

1. Org 200 accounts, cấm tắt CloudTrail/Config -> Organizations + SCP explicit deny.
2. Third-party scanner đa account -> cross-account role + ExternalId + least privilege.
3. API toàn cầu, APAC chậm + DDoS -> CloudFront + WAF + Shield + Route53 phù hợp.
4. Near-zero downtime nhưng budget hạn chế -> cân nhắc Warm Standby/Pilot Light.
5. KMS cross-account decrypt fail -> kiểm tra key policy/grants + SCP/boundary.
6. 300 VM, deadline cứng, khó sửa app -> migration factory + rehost/replatform.
7. DB cutover chỉ cho 30 phút downtime -> rehearsal + go/no-go + rollback.
8. S3 public policy tái diễn -> Config + auto-remediation + preventive controls.
9. Cost tăng sau migration, SLA giữ nguyên -> CUR + rightsize + autoscaling tuning.
10. EU residency nghiêm ngặt -> region constraints + replication guardrails.
11. Ops team mỏng, cần MTTR thấp -> EventBridge + SSM Automation + runbook.
12. Dev cần self-service IAM không leo quyền -> permission boundary + SCP.
13. Audit yêu cầu evidence incident -> immutable logs + incident timeline.
14. Monolith critical cần modernize ít rủi ro -> strangler theo domain.
15. Đề yêu cầu immediate + long-term -> chọn đáp án 2 pha.
16. Event integration cross-account phức tạp -> EventBridge policies + centralized governance.
17. User chậm nhưng CPU đẹp -> trace dependency + DB bottleneck analysis.
18. Least operational overhead cho web global -> managed edge/security stack.
19. DR cho workload non-critical, cost ưu tiên -> backup/restore hoặc pilot light.
20. 2 đáp án đều đúng kỹ thuật, 1 cái không scale org-wide -> chọn centralized/delegated model.

---

## 5 scenario ưu tiên luyện mỗi ngày
- #3 (global edge/security)
- #5 (KMS cross-account)
- #7 (migration cutover)
- #11 (incident ops automation)
- #15 (immediate vs long-term)
