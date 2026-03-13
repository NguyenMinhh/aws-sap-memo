## Topic Deep Dive - Exam Playbook for SAP-C02 (Constraint-Based)

## 0) Mục tiêu
- Làm nhanh câu dài, giảm chọn nhầm đáp án “đúng kỹ thuật nhưng sai đề”.

---

## 1) Playbook 5 bước
1. **Parse constraints**: compliance, RTO/RPO, cost, timeline, ops overhead.
2. **Loại đáp án fail hard constraints** ngay.
3. **So trade-off**: risk vs cost vs complexity.
4. **Chọn best-fit theo bối cảnh** (không phải “best tuyệt đối”).
5. **Sanity check**: có đáp ứng cả immediate need + maintainability không?

---

## 2) Priority khi đề mâu thuẫn
1. Compliance/legal/security baseline.
2. Business continuity (SLA, RTO/RPO).
3. Operational feasibility (least ops overhead).
4. Cost optimization.

---

## 3) Từ khóa gài bẫy + cách đọc
- **most cost-effective**: không phải rẻ nhất, phải đạt yêu cầu kỹ thuật.
- **least operational overhead**: ưu tiên managed service.
- **immediately**: phải có pha xử lý ngắn hạn.
- **without application changes**: tránh đáp án refactor lớn.
- **organization-wide**: cần centralized/delegated model.

---

## 4) Mẫu reasoning (để tự loại đáp án)
- “A sai vì fail compliance.”
- “B đúng vì đạt SLA + overhead thấp nhất.”
- “C đúng kỹ thuật nhưng vượt timeline.”
- “D thiếu khả năng scale org-wide.”

---

## 5) 6 bẫy kinh điển SAP-C02
1. Nhầm HA = DR.
2. Nhầm async replication = RPO 0.
3. Nhầm IAM Allow là đủ cross-account.
4. Over-engineer multi-region khi đề không cần.
5. Chọn self-managed dù đề yêu cầu least ops.
6. Chọn chiến lược dài hạn dù đề yêu cầu immediate mitigation.

---

## 6) 5 câu tự kiểm trước khi chốt đáp án
- Có thỏa toàn bộ hard constraints chưa?
- Có bỏ sót từ khóa “least ops/immediately/org-wide” không?
- Có phương án nào đơn giản hơn nhưng vẫn đạt yêu cầu không?
- Có layer policy nào chưa xét (SCP/boundary/resource/KMS)?
- Đáp án có hợp bối cảnh business không?
