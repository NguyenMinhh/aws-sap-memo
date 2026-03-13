## Topic Deep Dive - Security Response & Incident Operations (SAP-C02)

## 0) Mục tiêu ôn thi
- Chọn đúng kiến trúc detect/respond trong môi trường multi-account.
- Tăng tốc trả lời câu “reduce MTTR nhưng vẫn compliant”.

---

## 1) Flow chuẩn 6 bước (rất hay ra đề)
1. **Detect** (GuardDuty/Security Hub/CloudTrail/Config).
2. **Triage** (severity + owner + SLA).
3. **Contain** (cô lập nhanh, giảm blast radius).
4. **Eradicate** (xóa nguyên nhân gốc).
5. **Recover** (khôi phục an toàn).
6. **Prevent recurrence** (guardrail + automation).

**Mnemonic:** `D-T-C-E-R-P`.

---

## 2) Kiến trúc mẫu cho enterprise
- Security account làm delegated admin cho GuardDuty/Security Hub.
- Organization CloudTrail + Config aggregator để tập trung evidence.
- EventBridge route findings theo severity.
- SSM Automation cho các pattern lặp (public S3, SG mở rộng).

---

## 3) Decision table

| Tình huống đề | Chọn hướng |
|---|---|
| Findings quá nhiều, team xử lý chậm | Security Hub tập trung + severity routing |
| Incident lặp lại cùng misconfig | Config rules + auto-remediation |
| Suspected compromised EC2 | isolate network + snapshot + rotate credentials |
| Cần bằng chứng audit đầy đủ | immutable logs + timeline + ticket artifacts |

---

## 4) Auto-remediation: khi nào auto, khi nào manual
### Auto khi
- Pattern đã biết, blast radius thấp, rollback rõ.
### Manual khi
- Có thể gây downtime lớn hoặc đụng dữ liệu nhạy cảm.

**Rule làm bài:** đề có “least response time + repeat incident” => nghiêng về automation có guardrail.

---

## 5) 4 mini case
1. **Public S3 lặp lại nhiều lần** -> auto-remediate + preventive controls.
2. **Credential leak prod** -> revoke/rotate ngay + điều tra CloudTrail.
3. **Unknown critical compromise** -> containment tối thiểu + escalated manual investigation.
4. **Audit yêu cầu RCA traceable** -> tập trung logs, incident timeline, action evidence.

---

## 6) Exam traps
- Chỉ tăng alarm mà không có triage workflow.
- Tắt service toàn bộ để contain dù business continuity yêu cầu cao.
- Chỉ detective controls, không có preventive loop.

---

## 7) Quick answer templates
- `Org-wide visibility` -> Security Hub + GuardDuty delegated admin + org trail/config.
- `Giảm MTTR incident lặp` -> EventBridge + SSM Automation + runbook chuẩn hóa.
- `Audit-ready response` -> immutable centralized logging + evidence chain.
