## Topic Deep Dive - Identity & Access at Scale (SAP-C02)

## 0) Mục tiêu ôn thi
- Thiết kế truy cập đa account an toàn, scale theo OU/team.
- Chọn đúng giữa SCP, permission boundary, identity policy, resource policy.
- Xử lý nhanh các câu AccessDenied cross-account (đặc biệt S3/KMS).

---

## 1) Công thức chọn đáp án siêu nhanh
- **Guardrail toàn tổ chức** -> SCP.
- **Giới hạn trần quyền role do dev tự tạo** -> Permission Boundary.
- **Quản lý user tập trung nhiều account** -> IAM Identity Center + Permission Sets.
- **Truy cập tài nguyên cross-account** -> Trust policy + permission policy + resource policy.
- **Third-party SaaS** -> AssumeRole + ExternalId + least privilege + audit.

**Mnemonic:** `S-B-I-R` = `SCP - Boundary - Identity - Resource` (4 lớp phải nghĩ khi làm bài IAM).

---

## 2) IAM Identity Center (cụ thể theo enterprise)

### Khi nào nên chọn
- Có nhiều OU/account và user thay đổi liên tục.
- Muốn bỏ IAM user static, đồng bộ với IdP (Okta/Azure AD/AD).

### Mẫu triển khai dễ nhớ
1. Chia persona: `Platform`, `Security`, `Developer`, `ReadOnly`.
2. Tạo permission set theo persona + tách Prod/NonProd.
3. Gán theo OU (không gán lẻ từng account).
4. Bật MFA + session duration theo mức rủi ro.

### Sai lầm hay gặp
- Một permission set dùng chung mọi môi trường.
- Cho developer quyền rộng ở Prod vì “đỡ mất thời gian”.

---

## 3) SCP vs Boundary vs Policy (bảng thi)

| Cơ chế | Dùng khi nào | Không dùng để |
|---|---|---|
| SCP | Đặt guardrail cho OU/account | Cấp quyền trực tiếp |
| Permission Boundary | Chặn principal tự leo quyền | Quản trị toàn org |
| Identity policy | Cấp quyền cho role/user/group | Vượt qua SCP/Boundary |
| Resource policy | Cho phép principal truy cập resource | Thay thế toàn bộ IAM model |

**Quy tắc chấm điểm:** explicit deny luôn thắng.

---

## 4) AccessDenied triage checklist (dùng trong đề)
1. Có SCP deny không?
2. Boundary có chặn không?
3. Trust policy role đích đúng principal chưa?
4. Permission policy role đích đã allow action/resource chưa?
5. Resource policy (S3/KMS/SQS) đã mở đúng principal chưa?
6. Condition (region/tag/encryption context) có làm fail không?

### KMS note cực hay gài
- IAM Allow không đủ.
- Cần key policy/grant tương ứng.

---

## 5) 4 mini case hay gặp
1. **Dev tự tạo role quá rộng** -> Boundary + SCP deny privilege escalation.
2. **S3 cross-account AccessDenied** -> kiểm tra bucket policy + IAM caller + SCP.
3. **KMS decrypt fail dù admin role** -> kiểm tra key policy trước IAM.
4. **Yêu cầu emergency access** -> break-glass role có MFA + approval + full audit.

---

## 6) Exam traps (1 dòng dễ nhớ)
- “SCP cấp quyền” -> Sai.
- “IAM Allow là đủ” -> Sai.
- “Cross-account chỉ cần IAM phía gọi” -> Sai.
- “KMS giống S3 IAM-only” -> Sai.

---

## 7) Quick answer templates
- `Guardrail toàn org` -> Organizations + SCP.
- `Self-service IAM an toàn` -> Permission Boundary + SCP.
- `Truy cập tập trung enterprise` -> IAM Identity Center + permission sets theo OU.
- `Third-party an toàn` -> AssumeRole + ExternalId + audit trail.
