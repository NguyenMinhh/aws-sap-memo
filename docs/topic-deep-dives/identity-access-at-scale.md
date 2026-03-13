## Topic Deep Dive - Identity & Access at Scale (SAP-C02)

## 0) Mục tiêu ôn thi
- Thiết kế truy cập đa account an toàn, scale theo OU/team.
- Chọn đúng giữa SCP, permission boundary, identity policy, resource policy.
- Xử lý nhanh các câu AccessDenied cross-account (đặc biệt S3/KMS).
- Tránh bẫy quyền khi có nhiều lớp kiểm soát chồng nhau.

---

## 1) Khung tư duy 30 giây khi đọc đề IAM
1. **Scope đề bài là gì?** (1 account, multi-account, toàn org).
2. **Loại principal là gì?** (workforce user, workload role, third-party).
3. **Ràng buộc cứng là gì?** (compliance, least privilege, no IAM users, region restrictions).
4. **Yêu cầu vận hành gì?** (least overhead, centralized governance, self-service cho dev).

**Mnemonic:** `S-P-C-O` = `Scope - Principal - Constraints - Operations`.

---

## 2) IAM policy evaluation logic (phần quan trọng nhất)

### Quy tắc chấm điểm
- `Explicit Deny` luôn thắng.
- `Allow` chỉ hợp lệ khi **không có deny** và được cho phép bởi các lớp liên quan.

### Thứ tự kiểm tra thực chiến (để debug AccessDenied)
1. SCP có deny action/resource/condition không?
2. Permission boundary có chặn không?
3. Session policy (nếu AssumeRole có truyền) có thu hẹp quá mức không?
4. Identity policy của principal có allow đúng action/resource không?
5. Resource policy có cho principal truy cập không?
6. Với KMS: key policy/grant có cho phép không?

**Exam tip:** Câu có cụm `despite IAM allow` gần như luôn là bẫy policy layering.

---

## 3) Khi nào chọn cơ chế nào (decision table)

| Mục tiêu | Dùng chính | Bổ sung thường đi kèm |
|---|---|---|
| Guardrail toàn tổ chức | SCP | Control Tower guardrails, rollout theo OU |
| Dev tự tạo role nhưng không leo quyền | Permission Boundary | SCP deny privilege escalation |
| Quản lý workforce access tập trung | IAM Identity Center | Permission sets theo persona + OU |
| Cross-account data access | Role trust + resource policy | KMS key policy/grants |
| Third-party truy cập an toàn | AssumeRole + ExternalId | CloudTrail + session tagging |

---

## 4) IAM Identity Center (mở rộng cho SAP-C02)

### Pattern triển khai chuẩn
1. Dùng IdP ngoài (hoặc directory nội bộ) làm source of truth.
2. Tạo permission sets theo persona: Platform, Security, AppOps, ReadOnly.
3. Tách environment boundary: Prod/NonProd permission sets riêng.
4. Gán quyền ở OU/account scope, không gán lẻ thủ công.
5. Bật MFA, giới hạn session duration theo risk profile.

### Điều đề thi hay gài
- “Developer needs quick access to production for troubleshooting.”
  - Hướng đúng: JIT/break-glass + audit + approval; không cấp rộng mặc định.
- “Organization-wide access standardization.”
  - Hướng đúng: Identity Center + permission set governance.

### Sai lầm phổ biến
- Dùng 1 permission set cho mọi môi trường.
- Gán direct permissions trong account khiến drift.
- Quên lifecycle offboarding từ IdP.

---

## 5) SCP nâng cao (rất hay trúng SAP)

### SCP dùng tốt khi
- Cần cấm thao tác nguy hiểm toàn org (disable CloudTrail/Config, create access keys, leave approved regions).
- Cần enforce guardrail compliance ở mọi account.

### SCP anti-pattern
- Attach deny quá rộng ở Root OU ngay từ đầu.
- Không có exception role/break-glass path.
- Không rollout theo Sandbox -> NonProd -> Prod.

### Từ khóa đề thi
- `organization-wide governance`, `prevent disabling security services`, `enforce regional restrictions`.

---

## 6) Permission Boundary + Session Policy

### Permission Boundary
- Dùng khi cho team ứng dụng tự tạo role nhưng không vượt “trần quyền”.
- Thường kết hợp SCP để chặn escalation pattern (attach admin policies, create high-privilege roles).

### Session Policy
- Dùng để thu hẹp thêm quyền tạm thời khi AssumeRole.
- Hữu ích trong mô hình brokered access / just-in-time access.

**Exam trap:** nhầm boundary là công cụ governance toàn org (sai; boundary chỉ áp principal cụ thể).

---

## 7) Cross-account access chuẩn (S3/KMS/SNS/SQS)

### Checklist chung
1. Trust policy role đích cho đúng principal account nguồn.
2. Permission policy role đích allow đúng action/resource.
3. Resource policy mở đúng principal (nếu service yêu cầu).
4. KMS key policy/grant tương thích với caller.

### Điểm gài theo dịch vụ
- **S3:** bucket policy + IAM caller đều quan trọng.
- **KMS:** IAM allow chưa đủ, key policy/grants quyết định.
- **SNS->SQS cross-account:** cả topic policy và queue policy phải đúng.

---

## 8) ABAC / tag-based access (khi nào ưu tiên)
- Dùng tốt khi team/workload thay đổi nhanh và số lượng role lớn.
- Chuẩn hóa tag keys (`Project`, `Env`, `DataClass`, `Owner`).
- Kết hợp tag policy + SCP deny thiếu tag bắt buộc.

**Exam tip:** Nếu đề nhấn mạnh scale authorization linh hoạt, ABAC thường hợp hơn RBAC thuần.

---

## 9) Break-glass & privileged access governance
- Break-glass role riêng, giới hạn dùng trong incident.
- Điều kiện chặt: MFA, approval, time-bound session.
- Bắt buộc audit trail và alert khi assume role.
- Post-incident review để tránh lạm dụng.

---

## 10) AccessDenied runbook rút gọn (đem vào phòng thi)
- B1: Xác định principal thực tế đang gọi API.
- B2: Kiểm tra deny từ SCP/Boundary/Session.
- B3: Kiểm tra identity policy và resource policy.
- B4: Nếu có KMS -> kiểm tra key policy/grants.
- B5: So condition keys (region, sourceVpce, tags, encryption context).

---

## 11) 8 mini case sát đề
1. **Dev tự tạo role quá rộng** -> Boundary + SCP deny escalation.
2. **S3 cross-account vẫn AccessDenied** -> kiểm tra bucket policy + role caller + SCP.
3. **KMS decrypt fail dù admin role** -> key policy/grant trước IAM.
4. **No IAM users policy toàn org** -> Identity Center + SCP deny create access keys/users.
5. **Third-party SaaS scanner** -> AssumeRole + ExternalId + read-only scoped permissions.
6. **Need org-wide least privilege consistency** -> permission set catalog + OU assignment.
7. **Incident cần quyền tạm thời Prod** -> break-glass audited workflow.
8. **ABAC adoption for dynamic teams** -> tag schema + policy conditions + tag governance.

---

## 12) Exam traps (dễ mất điểm)
- “SCP cấp quyền.” -> Sai.
- “IAM Allow là đủ trong mọi trường hợp.” -> Sai.
- “Cross-account chỉ cần identity policy caller.” -> Sai.
- “Boundary thay thế SCP.” -> Sai.
- “KMS xử lý như S3 IAM-only.” -> Sai.

---

## 13) Quick answer templates
- `Guardrail toàn org` -> Organizations + SCP (rollout theo OU).
- `Self-service IAM an toàn` -> Permission Boundary + SCP deny escalation.
- `Workforce access tập trung` -> IAM Identity Center + permission sets theo persona/OU.
- `Cross-account data access` -> AssumeRole + resource policy + KMS policy alignment.
- `Third-party an toàn` -> AssumeRole + ExternalId + least privilege + audit trail.
