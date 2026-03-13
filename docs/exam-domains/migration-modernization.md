## AWS SAP-C02 - Domain 4: Accelerate Workload Migration and Modernization (Advanced Memo + Keyword)

## 0) Viết tắt thường gặp (đọc là hiểu)

- **7Rs:** Relocate, Rehost, Replatform, Refactor, Repurchase, Retain, Retire.
- **MGN (AWS Application Migration Service):** migrate server/VM theo kiểu rehost.
- **DMS (Database Migration Service):** migrate/replicate database.
- **SCT (Schema Conversion Tool):** convert schema/code khi đổi engine DB.
- **CDC (Change Data Capture):** đồng bộ thay đổi liên tục gần real-time.
- **RTO/RPO:** mục tiêu thời gian khôi phục / mất dữ liệu cho phép.

---

## 1) Scope nhanh (đề hay hỏi)

- Migration strategy (7Rs) theo business/risk/downtime
- Chọn tool đúng cho server/data/app
- Cutover planning, rollback planning, wave execution
- Modernization theo từng bước, không big-bang

**Keywords:** rehost, replatform, refactor, wave migration, near-zero downtime, dependency mapping.

---

## 2) Keyword -> đáp án nhanh

- **quickest move, minimal code change** -> Rehost + AWS MGN *(sample: lift-and-shift datacenter VM sang AWS nhanh)*
- **DB move with minimal downtime** -> AWS DMS (+ SCT nếu đổi engine/schema) *(sample: Oracle -> Aurora PostgreSQL với downtime thấp)*
- **petabyte/offline transport** -> Snow Family *(sample: dữ liệu rất lớn, đường truyền không đủ)*
- **continuous file transfer** -> DataSync/Transfer Family *(sample: đồng bộ file NAS/SFTP định kỳ)*
- **discover dependencies first** -> Migration Hub + App Discovery Service *(sample: chưa rõ app nào phụ thuộc DB/service nào)*
- **incremental modernization** -> strangler pattern + API facade + event decomposition *(sample: tách dần monolith theo từng bounded context)*

---

## 3) 7Rs memo (nhớ nhanh)

- **Relocate:** chuyển platform (VD VMware Cloud) ít đổi app.
- **Rehost:** lift-and-shift nhanh nhất.
- **Replatform:** đổi platform vừa phải, ít sửa code.
- **Refactor/Re-architect:** đổi sâu để cloud-native.
- **Repurchase:** thay bằng SaaS.
- **Retain:** giữ lại tạm thời.
- **Retire:** loại bỏ workload không còn giá trị.

---

## 4) Migration factory pattern

### A. Assess
- Inventory apps + dependencies + business criticality.

### B. Mobilize
- Landing zone, security baseline, network connectivity.

### C. Migrate & Modernize
- Theo wave, rehearsal cutover, rollback plan.

### D. Operate & Optimize
- Hypercare + post-migration tuning/cost/security.

---

## 5) Services cần nhớ

- MGN, DMS, SCT, Migration Hub
- App Discovery Service, DataSync, Snowball
- Step Functions/CodePipeline cho automation wave
- CloudEndure concepts (nền tảng lịch sử của MGN/DRS)

---

## 6) Câu khó hay gặp + đáp án định hướng

1) **Ứng dụng 3-tier phụ thuộc nhau, downtime chỉ 15 phút.**
- **Đáp án hướng:** dependency mapping + wave grouping + rehearsal + rollback.
  **Ghi nhớ:** downtime chặt -> rehearsal trước, cutover sau.

2) **DB Oracle sang Aurora PostgreSQL, downtime thấp.**
- **Đáp án hướng:** SCT convert schema + DMS CDC cutover.
  **Ghi nhớ:** khác engine DB thì nghĩ ngay SCT + DMS.

3) **Data center băng thông yếu, dữ liệu rất lớn.**
- **Đáp án hướng:** Snowball seeding + sync delta online trước cutover.
  **Ghi nhớ:** dữ liệu lớn + băng thông yếu = offline seed + online delta.

4) **Business muốn move nhanh rồi mới tối ưu.**
- **Đáp án hướng:** rehost trước, modernization theo phase sau.
  **Ghi nhớ:** business cần tốc độ -> migrate first, optimize later.

5) **Team lo rủi ro cutover production.**
- **Đáp án hướng:** blue/green hoặc parallel run + runbook + go/no-go criteria.
  **Ghi nhớ:** hệ critical luôn cần rollback criteria rõ ràng.

---

## 7) Trap keywords & anti-patterns

- **Sai:** Nhầm tool server migration (MGN) với DB migration (DMS).
  **Nên nhớ:** MGN cho server/VM, DMS cho database replication/migration.
  **Ví dụ:** đề hỏi Oracle -> Aurora mà chọn MGN là lệch hoàn toàn.

- **Sai:** Không dependency mapping trước khi chia wave.
  **Nên nhớ:** migration sai thứ tự dễ gây outage dây chuyền.
  **Ví dụ:** chuyển app trước DB phụ thuộc -> app lên cloud nhưng không chạy được.

- **Sai:** Chỉ plan cutover mà thiếu rollback criteria.
  **Nên nhớ:** cutover plan luôn phải đi kèm go/no-go + rollback rõ ràng.
  **Ví dụ:** sau cutover latency xấu nhưng không có đường quay lại.

- **Sai:** Refactor quá sớm khi business cần migrate nhanh.
  **Nên nhớ:** yêu cầu tốc độ cao thường nên rehost/replatform trước.
  **Ví dụ:** rewrite 9 tháng trong khi deadline rời data center là 3 tháng.

- **Sai:** Không validate security/compliance target state sau migrate.
  **Nên nhớ:** migrate xong phải đạt chuẩn bảo mật như (hoặc tốt hơn) trước đó.
  **Ví dụ:** workload chạy được nhưng log/audit encryption thiếu -> không go-live được.

---

## 8) Checklist chốt đáp án

1. Asset chính là server, database hay file/data lake?
2. Downtime/RPO/RTO constraints là gì?
3. Có đổi engine/platform không?
4. Dữ liệu lớn đến mức cần offline transfer không?
5. Có migration waves + rehearsal + rollback plan không?

> Đáp án tốt: **đúng tool cho đúng loại workload + giảm rủi ro cutover + lộ trình modernize thực tế**.

---


### Checklist có keyword + đáp án + memo

- **Keyword:** fastest move, minimal change -> **Đáp án:** Rehost với MGN.
- **Keyword:** heterogeneous DB migration + low downtime -> **Đáp án:** SCT + DMS CDC.
- **Keyword:** huge data + low bandwidth -> **Đáp án:** Snowball seed + delta sync online.
- **Keyword:** complex dependencies + tight cutover -> **Đáp án:** wave migration + rehearsal + rollback criteria.
- **Memo chốt:** Dùng đúng tool theo asset (server vs DB vs data transfer) quan trọng hơn “tool mới nhất”.

## 9) Quick answer templates (1 dòng)

- **"Move fast, low change"** -> `Rehost with MGN`.
- **"DB migrate low downtime"** -> `DMS CDC (+ SCT if heterogeneous)`.
- **"Huge data, limited bandwidth"** -> `Snowball + delta sync`.
- **"Modernize safely"** -> `Phased strangler approach, not big-bang rewrite`.
