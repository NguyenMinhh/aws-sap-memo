## Topic Deep Dive - Modernization Economics (SAP-C02)

## 0) Mục tiêu ôn thi
- Chọn rehost/replatform/refactor đúng theo deadline, risk, ROI.

---

## 1) Khung quyết định 3 trục
1. **Time-to-value**: cần nhanh -> rehost/replatform.
2. **Delivery risk**: hệ critical -> incremental + strangler.
3. **Long-term ROI**: cần giảm ops burden/technical debt -> replatform/refactor có chọn lọc.

---

## 2) Keyword -> chiến lược
- **tight timeline, minimal change** -> Rehost.
- **reduce ops overhead quickly** -> Replatform.
- **need new capabilities** -> Refactor.
- **high-risk monolith** -> Strangler phased modernization.

---

## 3) Bảng chọn nhanh
| Context | Hướng phù hợp | Không nên |
|---|---|---|
| Deadline cứng | Rehost trước | Refactor toàn hệ ngay |
| Team ops mỏng | Replatform managed services | Self-managed phức tạp |
| Mục tiêu innovation | Refactor theo domain | Chỉ rehost rồi dừng |

---

## 4) 4 mini case
1. **300 VM, 6 tháng, ít thay đổi code** -> Rehost + wave-based migration.
2. **Chi phí vận hành cao sau migrate** -> Replatform DB/cache/runtime.
3. **Monolith cản tốc độ ra feature** -> Strangler theo bounded context.
4. **Business yêu cầu value sớm + roadmap dài hạn** -> 2 pha: stabilize -> modernize.

---

## 5) Exam traps
- Refactor toàn bộ ở wave đầu.
- Chỉ nhìn infra cost ngắn hạn, bỏ qua productivity dài hạn.
- Rehost xong coi như hoàn tất modernization.

---

## 6) Quick answer templates
- `Fastest with controlled risk` -> Rehost now, modernize later.
- `Lower ops + reliability` -> Replatform managed services first.
- `Strategic transformation` -> Phased refactor with ROI gates.
