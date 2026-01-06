## VPC Peering (Not for Hybrid - only trong nội bộ AWS)

・VPC-to-VPC connection (không phải hybrid networking)

・KHÔNG transitive → dùng Transit Gateway thay thế

【KEYWORDS】

・NOT transitive, NOT scalable, peer-to-peer only

## Client VPN
　・VPN cho người dùng cá nhân (user / laptop) kết nối an toàn vào AWS
        
**【KEYWORDS】**

　・User / laptop / remote access <p>
         
　・Remote user, laptop, individual access, secure access, client-based

## Site-to-Site VPN
　・1 (hoặc nhiều) on-prem ↔ 1 VPC

　・Kết nối on-prem vào AWS, KHÔNG phải để các site nói chuyện với nhau
 
**【KEYWORDS】**

　・temporary, low cost, quick setup
 
　・single site, hybrid connectivity

        VPN CloudHub (đây một tính năng của Site-to-Site VPN, không phải service riêng biệt)
        ・NHIỀU on-prem sites ↔ AWS (làm HUB)
        ・Các chi nhánh nói chuyện với nhau THÔNG QUA AWS
        ・AWS vừa là trung tâm trung chuyển (hub) VÀ có thể là đích đến (VPC)
        ・Sites CÓ THỂ: (1) nói chuyện với nhau, (2) truy cập VPC resources
        【KEYWORDS】
        　　・multiple on-prem sites, branch offices
        　　・site-to-site via AWS, hub-and-spoke

## Hybrid Network Decision Tree
```
On-prem / User cần kết nối AWS?
        |
        |  [KEYWORDS]
        |  hybrid, on-premises, data center,
        |  private connectivity, remote access
        |
        +-- Ai kết nối?
        |        |
        |        +-- Người dùng cá nhân (User / Laptop)?
        |        |        |
        |        |        |  [KEYWORDS]
        |        |        |  remote user, laptop, WFH,
        |        |        |  individual access, secure access,
        |        |        |  client-based, OpenVPN-based
        |        |        |
        |        |     Client VPN
        |        |        |
        |        |        |  [NOTE]
        |        |        |  - OpenVPN-based
        |        |        |  - Username/password or cert auth
        |        |        |  - Attach to VPC (subnet)
        |        |        |  - Users get private IP
        |        |        |
        |        |     Kết nối tới VPC/subnet
        |        |
        |        +-- Site / Data center / Branch?
        |                 |
        |                 |  [KEYWORDS]
        |                 |  on-prem site, data center,
        |                 |  branch office, corporate network
        |                 |
        |              (Xem tiếp nhánh dưới...)
        |
        +-- Cần nhanh / tạm thời / chi phí thấp?
        |        |
        |        |  [KEYWORDS]
        |        |  quick setup, temporary, low cost,
        |        |  pilot, proof of concept,
        |        |  internet-based, IPsec
        |        |
        |     Site-to-Site VPN
        |        |
        |        |  [SPECS]
        |        |  - Internet-based (IPsec)
        |        |  - 2 tunnels (HA)
        |        |  - Up to 1.25 Gbps / tunnel
        |        |  - ECMP support (multi-tunnel)
        |        |  - Can be DX backup
        |        |  - BGP or Static routing
        |        |
        |        +-- Bao nhiêu on-prem sites?
        |                 |
        |                 +-- 1 site
        |                 |        |
        |                 |        |  [USE]
        |                 |        |  Simple hybrid on-prem ↔ AWS
        |                 |        |
        |                 |     Kết nối tới bao nhiêu VPC?
        |                 |        |
        |                 |        +-- 1 VPC
        |                 |        |        |
        |                 |        |        |  [KEYWORDS]
        |                 |        |        |  single VPC,
        |                 |        |        |  simple architecture
        |                 |        |        |
        |                 |        |     VGW (Virtual Private Gateway)
        |                 |        |        |
        |                 |        |        |  [NOTE]
        |                 |        |        |  - 1-1 với VPC
        |                 |        |        |  - BGP or Static
        |                 |        |        |
        |                 |        |       VPC
        |                 |        |
        |                 |        +-- Nhiều VPC / multi-account
        |                 |                 |
        |                 |                 |  [KEYWORDS]
        |                 |                 |  multiple VPCs,
        |                 |                 |  shared services,
        |                 |                 |  centralized routing,
        |                 |                 |  hub-and-spoke
        |                 |                 |
        |                 |              Transit Gateway (TGW)
        |                 |                 |
        |                 |                 |  [NOTE]
        |                 |                 |  - Hub network
        |                 |                 |  - Transitive routing
        |                 |                 |  - Up to 5000 attachments
        |                 |                 |  - 50 Gbps / VPC
        |                 |                 |
        |                 |              Multiple VPCs
        |                 |
        |                 +-- 2+ sites
        |                          |
        |                          +-- Sites CHỈ cần vào AWS?
        |                          |        |
        |                          |        |  [KEYWORDS]
        |                          |        |  multiple sites to AWS,
        |                          |        |  centralized routing,
        |                          |        |  no branch-to-branch
        |                          |        |
        |                          |     Site-to-Site VPN
        |                          |        |
        |                          |        +-- Transit Gateway (TGW)
        |                          |               |
        |                          |               |  [NOTE]
        |                          |               |  - Each site: separate VPN
        |                          |               |  - All attach to same TGW
        |                          |               |
        |                          |            Multiple VPCs
        |                          |
        |                          +-- Sites cần nói chuyện với NHAU?
        |                                   |
        |                                   |  [KEYWORDS]
        |                                   |  branch-to-branch,
        |                                   |  hub-and-spoke via AWS,
        |                                   |  inter-branch communication
        |                                   |
        |                                VPN CloudHub
        |                                   |
        |                                   |  [ARCHITECTURE]
        |                                   |  - 1 VGW + multiple CGWs
        |                                   |  - BGP routing (required)
        |                                   |  - AWS làm routing hub
        |                                   |
        |                                   |  [NOTE]
        |                                   |  - Sites CÓ THỂ vào VPC
        |                                   |  - Sites CÓ THỂ route qua nhau
        |                                   |  - Traffic qua AWS
        |                                   |
        |                                1 VPC (via VGW)
        |
        +-- Lâu dài / traffic lớn / low latency / compliance?
                 |
                 |  [KEYWORDS]
                 |  long-term, high throughput (> 1.25 Gbps),
                 |  predictable performance, consistent latency,
                 |  compliance, no internet, SLA,
                 |  dedicated private line
                 |
              Direct Connect (DX)
                 |
                 |  [SPECS]
                 |  - Dedicated: 1 / 10 / 100 Gbps
                 |  - Hosted: 50 Mbps - 10 Gbps
                 |  - Low latency (~10ms typical)
                 |  - Stable bandwidth
                 |  - No internet routing
                 |  [NOTE]
                 |  - Dedicated private line
                 |  - Stable bandwidth, low latency
                 |  - Thường đi kèm VPN backup (HA design)
                 |
        (⚠️ BẮT BUỘC phải có Virtual Interface – VIF)
                 |
                 |  [VIF TYPES]
                 |  - Private VIF: Access VPC
                 |  - Transit VIF: Access via DX Gateway
                 |  - Public VIF: Access S3, public services
                 |  - BGP required
                 |
        Bao nhiêu VPC? Bao nhiêu Region?
                 |
        +-------------------------+---------------------------+
        |                         |                           |
     1 VPC                  Nhiều VPC                    Nhiều VPC
     1 Region               1 Region                    Nhiều Region
        |                         |                           |
        |  [KEYWORDS]             |  [KEYWORDS]               |  [KEYWORDS]
        |  simple hybrid          |  multi VPC                |  multi-region
        |                         |  single region            |  scale, reuse DX
        |                         |                           |
   Private VIF              Private VIF                  Transit VIF
        |                         |                           |
        |  [NOTE]                 |  [NOTE]                   |  [NOTE]
        |  - BGP routing          |  - Central routing        |  - Cross-region
        |                         |                           |
       VGW                Transit Gateway (TGW)        Direct Connect Gateway
        |                         |                           |
        |  [NOTE]                 |  [NOTE]                   |  [NOTE]
        |  - 1 VPC only           |  - Hub network            |  - Global DX hub
        |                         |                           |  [LIMITS]
        |                         |                           |  - With VGW: Max 10 VPCs, SAME REGION ONLY ⚠️
        |                         |                           |  - With TGW: Max 3 TGWs, CROSS-REGION OK ✅
        |                         |                           |
       VPC                Multiple VPCs               Transit Gateway (TGW)
                                                              |
                                                              |  [NOTE]
                                                              |  - Hub-and-spoke
                                                              |
                                                        Multiple VPCs
                                                    (multi-account / multi-region)
