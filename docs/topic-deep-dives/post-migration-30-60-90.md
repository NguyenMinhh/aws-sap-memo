## Topic Deep Dive - Post-Migration 30/60/90 Checklist (SAP-C02)

## Mục tiêu
- Có checklist rõ sau cutover để không “migrate xong là bỏ”.
- Trả lời tốt câu hỏi: bước tiếp theo nào hợp lý nhất sau migration?

---

## 30 ngày - Stabilize (ưu tiên reliability/security)
- Chốt baseline SLO/SLI sau cutover.
- Bật đủ metrics/logs/traces/alarms.
- Xác nhận backup, encryption, IAM least privilege.
- Chạy incident drill cơ bản (runbook + on-call).

## 60 ngày - Optimize (ưu tiên cost/performance)
- Rightsize compute/storage/database theo usage thật.
- Tinh chỉnh autoscaling và DB parameters.
- Dùng CUR/Cost Explorer tìm waste + áp Savings Plans/RI nếu phù hợp.
- Giảm noisy alerts, tối ưu alert ownership.

## 90 ngày - Modernize (ưu tiên long-term value)
- Chọn candidate replatform/refactor theo ROI.
- Tăng automation cho compliance/remediation.
- Chuẩn hóa policy-as-code và governance workflow.
- Ghi lessons learned để dùng cho wave kế tiếp.

---

## Decision nhanh theo đề
- Hỏi **“immediately after migration”** -> chọn nhóm 30 ngày.
- Hỏi **“reduce cost without SLA impact”** -> chọn nhóm 60 ngày.
- Hỏi **“long-term agility/innovation”** -> chọn nhóm 90 ngày.

---

## Exam traps
- Nhảy vào refactor sâu ngay sau cutover khi chưa ổn định.
- Tối ưu cost quá sớm làm ảnh hưởng reliability.
- Không re-baseline KPI sau migration nên tối ưu sai hướng.
