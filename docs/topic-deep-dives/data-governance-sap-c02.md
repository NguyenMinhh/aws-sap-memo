## Topic Deep Dive - Data Governance for SAP-C02

## 0) Mục tiêu ôn thi
- Chọn đúng mô hình governance dữ liệu trong multi-account.
- Trả lời chắc các câu compliance + encryption + residency.

---

## 1) Checklist 6 điểm khi làm đề
1. Data classification rõ chưa?
2. Encrypt at rest/in transit đủ chưa?
3. KMS key ownership/rotation/audit rõ chưa?
4. Access model least privilege + ABAC chưa?
5. Data residency có vi phạm legal constraints không?
6. Evidence/audit logs có đầy đủ không?

---

## 2) KMS thực chiến (hay gài)
- IAM Allow không đủ nếu key policy chưa cho.
- Tách key theo env và data class (Prod/NonProd, PCI/non-PCI).
- Dùng central governance account nếu đề yêu cầu separation of duties.

---

## 3) Pattern chọn nhanh
| Bài toán | Hướng chọn |
|---|---|
| Cross-account data lake | S3 bucket policy + role + KMS policy/grant |
| Private data service access | PrivateLink |
| Regulatory immutable retention | Object Lock / Vault Lock |
| Prevent public exposure | Block public + Config + auto-remediation |

---

## 4) 4 mini case
1. **EU data không ra ngoài EU** -> region allowlist + replication guardrail.
2. **Decrypt fail cross-account** -> key policy/grant trước IAM.
3. **Audit yêu cầu evidence immutable** -> centralized immutable logs + retention policy.
4. **Team dynamic, quyền thay đổi liên tục** -> ABAC/tag governance.

---

## 5) Exam traps
- “SSE-S3 là đủ mọi compliance” -> sai trong đề cần key control.
- “Replication nhiều region luôn tốt hơn” -> sai nếu residency nghiêm ngặt.
- “Data lake mở nhanh trước, governance sau” -> thường sai với SAP-C02.

---

## 6) Quick answer templates
- `Cần key control + audit mạnh` -> KMS governance theo classification.
- `Cross-account secure data access` -> role + resource policy + KMS policy đồng bộ.
- `Residency strict` -> region constraints + replication control + compliance logging.
