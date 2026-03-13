## Topic Deep Dive - Edge Security & Global Delivery (SAP-C02)

## 0) Mục tiêu ôn thi
- Chọn đúng entry-point toàn cầu theo latency, availability, security.

---

## 1) Chọn dịch vụ trong 10 giây
- **HTTP(S) web/API + cache + WAF** -> CloudFront.
- **TCP/UDP + anycast IP + failover nhanh** -> Global Accelerator.
- **DNS routing theo mục tiêu business** -> Route 53 policies.
- **DDoS nâng cao** -> Shield Advanced.

---

## 2) Route 53 policy map
| Mục tiêu | Policy |
|---|---|
| Giảm latency user toàn cầu | Latency-based |
| Active-passive DR | Failover |
| Ràng buộc địa lý/compliance | Geolocation/Geoproximity |
| Canary/traffic shift | Weighted |

---

## 3) Pattern hay trúng đề
- CloudFront + WAF + Shield cho web toàn cầu.
- Route 53 failover + health checks cho DR routing.
- Multi-origin + origin failover để tăng availability.

---

## 4) 4 mini case
1. **Global e-commerce bị bot/DDoS** -> CloudFront + WAF managed rules + Shield Advanced.
2. **Banking app cần route theo quốc gia** -> Route 53 geolocation + compliance controls.
3. **Gaming TCP service cần IP tĩnh toàn cầu** -> Global Accelerator.
4. **Yêu cầu overhead thấp** -> managed edge controls + centralized logs.

---

## 5) Exam traps
- Global không đồng nghĩa luôn active-active multi-region.
- Route 53 failover không giải quyết được data consistency.
- WAF không thay được security kiến trúc backend.

---

## 6) Quick answer templates
- `Global web latency + security` -> CloudFront + WAF (+ Shield Advanced nếu critical).
- `Geo-compliance routing` -> Route 53 geolocation.
- `TCP fast failover + global static IP` -> Global Accelerator.
