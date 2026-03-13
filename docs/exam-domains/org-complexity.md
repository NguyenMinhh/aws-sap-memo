## AWS SAP-C02 - Domain 1: Design Solutions for Organizational Complexity (Advanced Memo + Keyword)

## 0) Viết tắt thường gặp (đọc là hiểu)

- **SCP (Service Control Policy):** policy guardrail cấp Organizations.
- **IAM (Identity and Access Management):** quản lý danh tính/quyền truy cập.
- **OU (Organizational Unit):** nhóm account trong Organizations.
- **TGW (Transit Gateway):** hub kết nối network có transitive routing.
- **DX (Direct Connect):** kết nối private line on-prem -> AWS.
- **SSO / IAM Identity Center:** đăng nhập tập trung đa account.

---

## 1) Scope nhanh (đề thi hỏi gì)

- Multi-account governance ở quy mô lớn
- Kết nối network phức tạp: hybrid, multi-region, multi-VPC, multi-account
- Security/compliance tập trung + delegated admin
- Shared services model, centralized logging/inspection
- M&A / nhiều business unit / data residency theo quốc gia

**Keywords siêu hay gặp:** organizations, landing zone, governance, guardrails, delegated admin, centralized security, transitive routing, private connectivity, inspection VPC, egress control.

---

## 2) Keyword -> đáp án nhanh (exam mapping)

- **many accounts, prevent policy drift, mandatory baseline** -> AWS Organizations + SCP + Control Tower *(sample: công ty 200+ accounts cần guardrail bắt buộc)*
- **team cần tự chủ nhưng vẫn bị giới hạn bảo mật** -> OU strategy + SCP deny-by-default cho hành vi rủi ro *(sample: dev tự deploy nhưng cấm tắt CloudTrail/KMS)*
- **centralized security findings toàn tổ chức** -> Security Hub delegated admin + organization aggregation *(sample: SOC team xem findings tập trung từ mọi account)*
- **centralized config compliance** -> AWS Config aggregator + conformance packs *(sample: kiểm tra chuẩn CIS/tagging cho toàn org)*
- **network hub-and-spoke, transitive routing** -> Transit Gateway + route table segmentation *(sample: 100+ VPC tách prod/dev/shared)*
- **on-prem + nhiều account/VPC private access** -> DX Gateway + TGW (hoặc VPN to TGW) *(sample: datacenter truy cập private vào nhiều VPC)*
- **private service sharing cross-account** -> PrivateLink / VPC Lattice / RAM *(sample: account A gọi service nội bộ account B không qua internet)*
- **central DNS hybrid** -> Route 53 Resolver inbound/outbound endpoints + forwarding rules *(sample: on-prem resolve private DNS của AWS)*
- **không dùng long-term key cross-account** -> IAM Role AssumeRole + IAM Identity Center *(sample: user login SSO rồi assume role sang prod account)*

---

## 3) Kiến trúc chuẩn hay ra đề (đọc đề là hình dung được ngay)

### A. Multi-account Landing Zone
- Account tách theo: `Security`, `Log Archive`, `Shared Services`, `Workloads (Prod/NonProd)`.
- Apply guardrails bằng SCP theo OU (prod OU chặt hơn sandbox OU).
- Audit trail tập trung về Log Archive account.
- **Hình dung khi đọc đề:** công ty lớn nhiều team, mỗi team 1 account, nhưng security/audit muốn kiểm soát tập trung.
- **Dấu hiệu nhận diện:** `many accounts`, `governance baseline`, `mandatory guardrails`, `audit toàn tổ chức`.

### B. Centralized Egress + Inspection
- Workload VPC route qua TGW đến Egress/Inspection VPC.
- Dùng AWS Network Firewall / third-party firewall.
- NAT/egress policy tập trung + giảm public exposure.
- **Hình dung khi đọc đề:** toàn bộ traffic ra internet phải đi qua “cửa kiểm soát chung” để lọc/ghi log.
- **Dấu hiệu nhận diện:** `inspection`, `egress control`, `compliance requires centralized filtering`.

### C. Hybrid Enterprise Connectivity
- On-prem vào AWS qua DX/VPN, phân phối tới nhiều VPC bằng TGW.
- Route domain tách rõ (prod/dev/shared) tránh route leak.
- **Hình dung khi đọc đề:** datacenter/branch kết nối 1 lần vào hub AWS rồi đi đến nhiều môi trường.
- **Dấu hiệu nhận diện:** `on-prem + many VPC`, `transitive routing`, `segmentation`.

### D. Shared Services VPC
- AD, DNS outbound resolver, artifact repo, CI/CD services đặt tập trung.
- Chia sẻ cho account khác qua RAM/PrivateLink.
- **Hình dung khi đọc đề:** có 1 “khu tiện ích chung” để mọi account dùng dịch vụ nền tảng.
- **Dấu hiệu nhận diện:** `shared DNS/AD`, `central CI/CD tooling`, `cross-account private service access`.

---

## 4) Services cần nhớ (và lúc nào chọn)

- **AWS Organizations + SCP:** governance cấp tổ chức.
- **AWS Control Tower:** dựng landing zone nhanh + account factory + guardrails.
- **Transit Gateway:** kết nối transitive scale lớn, tách route table theo domain.
- **AWS RAM:** share TGW/subnet/service giữa account.
- **PrivateLink / VPC Endpoints:** private access, tránh internet.
- **CloudTrail org trail:** audit tập trung, forensic.
- **AWS Config org aggregator:** compliance trạng thái tài nguyên đa account.
- **Security Hub / GuardDuty delegated admin:** security findings tập trung.
- **KMS multi-account model:** tách key ownership, key policy chặt.

---

## 5) Câu khó hay gặp + đáp án định hướng

1) **Công ty có 300 account, yêu cầu cấm xóa CloudTrail/KMS key ở mọi account, team app vẫn tự deploy được.**
- **Đáp án hướng:** Organizations + SCP explicit deny hành vi nguy hiểm; delegated operations qua role.
  **Ghi nhớ:** “Bắt buộc toàn org” -> ưu tiên SCP thay vì IAM policy cục bộ.

2) **Cần kết nối 100+ VPC cross-account và on-prem, có yêu cầu route transitive + segmentation prod/dev.**
- **Đáp án hướng:** TGW + nhiều TGW route tables + propagation control; hybrid attach DX/VPN.
  **Ghi nhớ:** thấy từ “transitive + segmentation” là nghĩ TGW route tables.

3) **Security team muốn xem findings tập trung nhưng workload team giữ account riêng.**
- **Đáp án hướng:** Security Hub delegated admin + Org integration; GuardDuty org-level enable.
  **Ghi nhớ:** team security tập trung -> delegated admin là keyword vàng.

4) **Workload private cần gọi service nội bộ account khác, không đi internet, không mở CIDR rộng.**
- **Đáp án hướng:** PrivateLink (interface endpoint) hoặc VPC Lattice service network.
  **Ghi nhớ:** “private cross-account service” -> PrivateLink trước, peering sau.

5) **M&A: 2 tổ chức AWS cần governance thống nhất nhưng rollout dần, giảm rủi ro.**
- **Đáp án hướng:** chuẩn hóa OU/SCP baseline trước, migrate account theo wave, dùng shared logging/security account.
  **Ghi nhớ:** M&A làm theo wave + baseline trước để tránh compliance drift.

---

## 6) Keyword bẫy (trap keywords)

- **“quick and simple VPC connectivity”** với ít VPC -> có thể peering (không phải TGW mọi lúc).
- **“transitive routing required”** -> peering sai, phải TGW.
- **“must enforce for all accounts”** -> policy cục bộ không đủ, cần Organizations/SCP.
- **“private access to service in another account”** -> ưu tiên PrivateLink.
- **“centralized inspection/egress control”** -> centralized egress VPC + TGW.
- **“data residency”** -> tách account/OU + region strategy + policy guardrail.

---

## 7) Sai lầm phổ biến (SAP traps)

- Dùng VPC peering mesh cho topology lớn -> khó vận hành, không transitive.
- Nhầm IAM permission boundary thay cho SCP ở cấp tổ chức.
- Bật CloudTrail từng account riêng lẻ nhưng không có org-level log integrity tập trung.
- Không phân tách route table TGW theo trust zone -> route leak giữa môi trường.
- Quên DNS design hybrid (Resolver endpoints/rules) làm private name resolution lỗi.

---

## 8) Checklist chốt đáp án (siêu nhanh)

1. Bài toán ở **scope account đơn lẻ** hay **toàn tổ chức**?
2. Có cần **mandatory enforcement** không? -> SCP.
3. Network có cần **transitive + segmentation** không? -> TGW + route-table strategy.
4. Có yêu cầu **private-only connectivity** không? -> PrivateLink/VPC endpoints.
5. Security/compliance có cần **aggregated view** không? -> Security Hub/Config/CloudTrail org-wide.

> Đáp án đúng thường là: **Organizations baseline + TGW segmentation + centralized security/audit + private connectivity đúng ngữ cảnh**.

---


### Checklist có keyword + đáp án + memo

- **Keyword:** enforce all accounts, mandatory guardrails -> **Đáp án:** Organizations + SCP (+ Control Tower).
- **Keyword:** transitive routing, many VPCs/accounts -> **Đáp án:** TGW + route table segmentation.
- **Keyword:** private cross-account service access -> **Đáp án:** PrivateLink (không mở CIDR rộng).
- **Keyword:** centralized security/audit -> **Đáp án:** Org CloudTrail + Config aggregator + Security Hub delegated admin.
- **Memo chốt:** Scope càng lớn (toàn org) -> ưu tiên giải pháp org-level, không chọn giải pháp account-local.

## 9) Quick answer templates (1 dòng)

- **“Enforce guardrails across all accounts”** -> `AWS Organizations + SCP (+ Control Tower)`.
- **“Scale network across many VPC/accounts with transitive routing”** -> `Transit Gateway + segmented route tables`.
- **“Private cross-account service access”** -> `AWS PrivateLink`.
- **“Centralized org security posture”** -> `Security Hub + GuardDuty delegated admin`.
- **“Hybrid enterprise network with governance”** -> `DX/VPN -> TGW + centralized inspection + org guardrails`.

---

## 10) SCP nâng cao (rất hay xuất hiện trong đề)

### A. Khi nào dùng SCP
- Dùng để đặt **giới hạn tối đa** cho tài khoản/OU trong Organizations.
- SCP **không cấp quyền**, chỉ giới hạn quyền mà IAM có thể cấp.
- Áp dụng cho member accounts (management account có hành vi đặc thù, không dùng SCP để tự khóa nhầm).

### B. Pattern SCP thường gặp
- **Deny rời region cho phép** (data residency/compliance).
- **Deny tắt CloudTrail/Config/GuardDuty/Security Hub**.
- **Deny xóa KMS keys, deny tắt encryption controls**.
- **Deny tạo IAM user/access key**, bắt buộc dùng IAM Identity Center + AssumeRole.
- **Deny public S3 policy / public access changes**.

### C. Trap SCP (làm rõ để tránh hiểu mơ hồ)
- **Sai:** Attach 1 SCP deny quá chặt ở Root OU (áp cho toàn bộ account).
  **Hậu quả:** chặn luôn các hành vi vận hành hợp lệ như deploy pipeline, xoay vòng key, patch incident.
  **Nên làm:** rollout theo tầng (Sandbox -> NonProd -> Prod), test policy trước rồi mới attach rộng.
  **Ví dụ:** deny `iam:*` ở root khiến pipeline không thể `PassRole` để deploy.

- **Sai:** Không carve-out `break-glass role` cho tình huống khẩn cấp.
  **Hậu quả:** khi incident xảy ra, team không có đường thao tác khôi phục nhanh.
  **Nên làm:** tạo role khẩn cấp có điều kiện chặt (MFA, approval, audit log đầy đủ, thời hạn ngắn).
  **Ví dụ:** region gặp sự cố nhưng role vận hành bị SCP deny toàn bộ action sửa route/failover.

- **Sai:** Nhầm SCP với Permission Boundary rồi dùng sai chỗ.
  **Hậu quả:** nghĩ đã khóa toàn org nhưng thực tế chỉ giới hạn 1 nhóm principal (hoặc ngược lại).
  **Nên làm:** nhớ quy tắc: SCP = guardrail cấp Organizations; Boundary = trần quyền cho từng IAM principal.
  **Ví dụ:** dùng boundary để "enforce toàn org" -> account khác vẫn tạo quyền vượt chuẩn nếu không có SCP.

---

## 11) IAM Role & Policy - checklist ra quyết định

### A. Role design (cross-account)
- Account A cần truy cập B -> tạo Role ở B + trust policy cho principal từ A.
- Quyền thực thi nằm ở **permission policy của role B**.
- Dùng **external ID** cho third-party để chống confused deputy.

### B. Policy types cần nhớ
- **Identity-based policy:** gắn vào user/role/group.
- **Resource-based policy:** gắn vào tài nguyên (S3 bucket, KMS key, SQS, SNS...).
- **SCP:** guardrail cấp org.
- **Permission boundary:** trần quyền tối đa cho IAM principal.
- **Session policy:** thu hẹp thêm quyền khi AssumeRole.

### C. Evaluation logic (keyword hay gài)
- `Explicit deny` luôn thắng.
- Muốn `Allow` thì phải được cho phép bởi **mọi lớp liên quan** (SCP, boundary, identity/resource policy).
- Thiếu một lớp allow cần thiết => implicit deny.

---

## 12) Permission Boundary, ABAC, Tag-based control

### A. Permission Boundary
- Dùng khi cho team app tự tạo role nhưng không vượt guardrail.
- Pattern: "developer self-service IAM role creation but limited permissions".

### B. ABAC (Attribute-Based Access Control)
- Dùng tag (`Project`, `Env`, `CostCenter`) để scale authorization ở nhiều account.
- Pattern đề: "dynamic teams, nhiều workload thay đổi nhanh" -> ABAC phù hợp hơn RBAC thuần.

### C. Tag policy + SCP phối hợp
- Tag policies để chuẩn hóa key/value.
- SCP để deny action nếu thiếu tag bắt buộc (ví dụ `Owner`, `DataClass`).

---

## 13) Resource policy điểm khó (S3/KMS/SNS/SQS)

- **S3 cross-account:** cần bucket policy cho principal account khác + IAM principal phù hợp.
- **KMS cross-account:** cần key policy mở đúng principal + IAM policy bên caller.
- **SNS->SQS cross-account:** cả topic policy và queue policy phải đúng.
- **Trap:** chỉ sửa IAM policy mà quên resource policy => vẫn AccessDenied.

---

## 14) Câu khó thực tế (SCP/IAM/Permission)

1) **Dev team tạo role quá rộng gây rủi ro, nhưng vẫn cần self-service nhanh.**
- **Hướng đáp án:** permission boundary bắt buộc + SCP deny privilege escalation actions.

2) **Security yêu cầu mọi account không được tắt CloudTrail/Config.**
- **Hướng đáp án:** org-level services + SCP explicit deny `StopLogging/DeleteTrail/StopConfigurationRecorder`.

3) **Third-party SaaS cần vào account để scan, nhưng phải an toàn.**
- **Hướng đáp án:** cross-account role + external ID + least privilege + CloudTrail monitoring.

4) **App cross-account bị AccessDenied dù IAM đã Allow.**
- **Hướng đáp án:** kiểm tra SCP, permission boundary, resource policy (đặc biệt KMS/S3).

5) **Muốn buộc workload chỉ deploy ở approved regions.**
- **Hướng đáp án:** SCP deny theo `aws:RequestedRegion` + exception role rõ ràng.

---

## 15) Quick keywords riêng cho IAM/SCP

- **"enforce across all accounts"** -> SCP/Organizations.
- **"self-service but controlled"** -> permission boundary.
- **"cross-account secure access"** -> AssumeRole + trust policy + external ID.
- **"access denied even with allow"** -> check explicit deny / SCP / resource policy / KMS key policy.
- **"attribute-driven authorization"** -> ABAC (principal/resource tags).

> Rule chốt: đề càng nhấn mạnh "toàn tổ chức + bắt buộc + không lệch chuẩn" thì càng nghiêng về **Organizations + SCP + delegated security services**.
