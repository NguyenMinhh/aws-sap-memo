## Topic Deep Dive 6 - Migration Factory (Wave-Based)

### Mục tiêu
- Di chuyển khối lượng lớn workload với rủi ro thấp, downtime kiểm soát.

### Core Components
- Discovery (dependency mapping)
- Wave planning (group theo dependency/criticality)
- AWS Application Migration Service (MGN)
- AWS Database Migration Service (DMS) + Schema Conversion Tool (SCT)
- Rehearsal cutover + rollback criteria

### Reference Pattern (nhớ nhanh)
1. Inventory app + dependency map.
2. Chia wave theo risk và business impact.
3. Chạy rehearsal trước production cutover.
4. Định nghĩa go/no-go + rollback rõ ràng.
5. Hypercare sau cutover rồi mới tối ưu tiếp.

### Keywords
- wave migration, cutover, rollback plan, dependency map, low downtime

### Trap
- Cutover one-shot không rehearsal cho hệ critical -> rủi ro rất cao.
