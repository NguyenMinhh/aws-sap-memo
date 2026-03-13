## Topic Deep Dive 3 - Multi-Account Landing Zone Architecture (Advanced Memo + Keyword)

## 0) Viết tắt thường gặp

- **OU (Organizational Unit):** nhóm account theo môi trường/chức năng.
- **SCP (Service Control Policy):** guardrail cấp tổ chức, không cấp quyền trực tiếp.
- **CT (Control Tower):** dịch vụ dựng landing zone + account factory + guardrails.
- **AFT (Account Factory for Terraform):** tự động cấp account bằng Terraform.
- **RAM (Resource Access Manager):** chia sẻ tài nguyên cross-account.
- **Log Archive account:** account lưu audit log tập trung, hạn chế chỉnh sửa.

---

## 1) Scope ôn thi SAP-C02 cho topic này

- Thiết kế multi-account governance ở quy mô enterprise.
- Tổ chức security/audit tập trung nhưng workload team vẫn tự chủ.
- Thiết kế network/shared services chuẩn để scale bền vững.

**Keywords hay gặp:** landing zone, account factory, governance baseline, guardrails, delegated admin, centralized logging.

---

## 2) Keyword -> đáp án nhanh (exam mapping + sample)

- **many accounts, mandatory governance** -> Organizations + OU strategy + SCP baseline *(sample: 300 account phải cùng chuẩn bảo mật)*
- **rapid account provisioning with standards** -> Control Tower (hoặc AFT) *(sample: mỗi tháng tạo thêm 20 account mới)*
- **centralized audit and immutable logs** -> Org CloudTrail + Log Archive account *(sample: team audit yêu cầu bằng chứng toàn tổ chức)*
- **security team needs central visibility** -> Security Hub/GuardDuty delegated admin *(sample: SOC monitor toàn bộ account)*
- **shared services across accounts** -> Shared Services VPC + RAM/PrivateLink *(sample: DNS/AD/CI dùng chung)*
- **prevent risky IAM usage org-wide** -> SCP deny IAM user/access key patterns *(sample: bắt buộc SSO + AssumeRole)*

---

## 3) Kiến trúc chuẩn hay ra đề (đọc đề là thấy kiến trúc)

### A. Core landing zone accounts
- Management account
- Security account
- Log Archive account
- Shared Services account
- Workload accounts (Prod/NonProd/Sandbox)
- **Hình dung khi đọc đề:** mỗi account có vai trò rõ như một phòng ban; security/logging tách riêng để kiểm soát.
- **Dấu hiệu nhận diện:** `separate security/log archive accounts`, `enterprise multi-account setup`.

### B. OU segmentation model
- Prod OU (guardrail chặt)
- NonProd OU (linh hoạt hơn)
- Sandbox OU (thử nghiệm có giới hạn)
- **Hình dung khi đọc đề:** cùng một tổ chức nhưng mỗi khu có “nội quy” khác nhau.
- **Dấu hiệu nhận diện:** `different controls per environment`, `strict policy for production`.

### C. Centralized security/audit
- Org CloudTrail + Config aggregator.
- Delegated admin cho Security Hub/GuardDuty.
- Quy trình incident response xuyên account.
- **Hình dung khi đọc đề:** SOC ngồi 1 nơi vẫn nhìn được toàn bộ hoạt động của tất cả account.
- **Dấu hiệu nhận diện:** `central visibility`, `organization-wide evidence`, `compliance reporting`.

### D. Network + shared services
- TGW hub-and-spoke cho multi-account VPC.
- Shared DNS/AD/artifact services.
- Egress/inspection tập trung khi yêu cầu compliance.
- **Hình dung khi đọc đề:** các account là “quận”, TGW là “vành đai giao thông trung tâm”, shared services là hạ tầng dùng chung.
- **Dấu hiệu nhận diện:** `many VPC/accounts routing`, `shared DNS/AD`, `central egress inspection`.

---

## 4) Services cần nhớ (khi nào chọn)

- **AWS Organizations + SCP:** governance bắt buộc toàn org.
- **AWS Control Tower:** landing zone nhanh, guardrail mặc định.
- **AWS Config (Org Aggregator):** compliance visibility đa account.
- **CloudTrail Org Trail:** audit log tập trung, phục vụ forensic.
- **Security Hub + GuardDuty delegated admin:** security posture tập trung.
- **TGW + RAM + PrivateLink:** kết nối/chia sẻ dịch vụ cross-account.

---

## 5) Câu thực tế hay gặp + đáp án định hướng

1) **Công ty tăng từ 20 lên 200 account trong 1 năm.**
- **Đáp án hướng:** Control Tower/AFT + account baseline automation.
- **Ghi nhớ:** scale account nhanh mà không automation = drift chắc chắn xảy ra.

2) **Dev team cần tự deploy nhưng security cấm tắt CloudTrail/KMS.**
- **Đáp án hướng:** SCP explicit deny hành vi rủi ro + IAM role delegation.
- **Ghi nhớ:** tự chủ workload vẫn phải trong guardrail org-level.

3) **Audit yêu cầu truy vết sự kiện toàn tổ chức trong 7 năm.**
- **Đáp án hướng:** org trail về Log Archive account + lifecycle/retention policy.
- **Ghi nhớ:** log phải tập trung, immutable và có retention rõ.

4) **M&A: hợp nhất governance 2 tổ chức AWS khác nhau.**
- **Đáp án hướng:** chuẩn OU/SCP baseline trước, migration account theo wave.
- **Ghi nhớ:** governance first, migration second.

5) **Shared DNS/AD cho nhiều account nhưng không muốn mở CIDR rộng.**
- **Đáp án hướng:** Shared Services VPC + PrivateLink/Resolver strategy.
- **Ghi nhớ:** private service sharing ưu tiên endpoint-based.

---

## 6) Checklist trả lời câu SAP (format giống DR)

1) **Bài toán ở scope account đơn lẻ hay toàn tổ chức?**
- **Keyword:** enforce across all accounts -> Organizations + SCP.
- **Keyword:** local app permission only -> IAM policy/role có thể đủ.

2) **Có cần tạo account mới nhanh và chuẩn hóa không?**
- **Keyword:** rapid account vending -> Control Tower/AFT.
- **Keyword:** manual account setup causes drift -> automation bắt buộc.

3) **Có yêu cầu audit/security tập trung không?**
- **Keyword:** centralized evidence/compliance -> Org CloudTrail + Config aggregator + delegated admin.
- **Keyword:** SOC visibility gap -> Security Hub/GuardDuty org integration.

4) **Có nhu cầu shared services/network segmentation không?**
- **Keyword:** many VPC/accounts transitive routing -> TGW + route-table segmentation.
- **Keyword:** private service sharing -> RAM/PrivateLink.

5) **Giải pháp có giảm rủi ro drift lâu dài không?**
- **Keyword:** recurring policy violations -> SCP + config conformance + auto-remediation.
- **Keyword:** inconsistent provisioning -> account factory + baseline pipeline.

> Chốt đáp án: phải có đủ **governance baseline + centralized security/audit + scalable account/network model**.

---

## 7) Trap keywords (bẫy thường gặp)

- **Sai:** Chỉ dùng IAM policy, bỏ SCP cho yêu cầu org-wide.
  **Nên nhớ:** IAM policy cục bộ không enforce toàn tổ chức như SCP.
  **Ví dụ:** account mới tạo ra không thừa hưởng guardrail nếu không có SCP đúng OU.

- **Sai:** Dựng nhiều account nhưng không có logging/audit tập trung.
  **Nên nhớ:** không có org trail/aggregator thì audit điều tra rất khó.
  **Ví dụ:** security incident xảy ra nhưng log phân tán, thiếu bằng chứng chuỗi sự kiện.

- **Sai:** Chưa phân OU rõ nhưng đã mở quyền rộng cho prod.
  **Nên nhớ:** prod và sandbox phải khác mức guardrail.
  **Ví dụ:** sandbox user có quyền gần như prod dẫn đến rủi ro nhầm lẫn cao.

- **Sai:** Chia sẻ service bằng mở CIDR rộng thay vì endpoint/private pattern.
  **Nên nhớ:** mở mạng rộng làm tăng attack surface và khó compliance.
  **Ví dụ:** cho phép toàn dải VPC truy cập service thay vì PrivateLink theo endpoint.

- **Sai:** Không có account lifecycle automation.
  **Nên nhớ:** tạo account thủ công lâu dài chắc chắn drift.
  **Ví dụ:** mỗi account thiếu 1 guardrail khác nhau, vận hành và audit bị phân mảnh.