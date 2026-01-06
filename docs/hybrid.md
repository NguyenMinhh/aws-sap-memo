```
On-prem cần kết nối AWS?
        |
        |  [KEYWORDS]
        |  hybrid, on-premises, data center, private connectivity
        |
        +-- Cần nhanh / tạm thời / chi phí thấp?
        |        |
        |        |  [KEYWORDS]
        |        |  quick setup, temporary, low cost,
        |        |  pilot, proof of concept
        |        |
        |     Site-to-Site VPN
        |        |
        |        |  [NOTE]
        |        |  - Đi qua Internet
        |        |  - AWS default có 2 tunnels (HA)
        |        |  - Có thể dùng làm backup cho DX
        |        |
        |     Kết nối tới bao nhiêu VPC?
        |        |
        |        +-- 1 VPC
        |        |        |
        |        |        |  [KEYWORDS]
        |        |        |  single VPC, simple architecture
        |        |        |
        |        |     VGW (Virtual Private Gateway)
        |        |        |
        |        |        |  [NOTE]
        |        |        |  - Gắn 1–1 với VPC
        |        |        |  - Không scale nhiều VPC
        |        |        |
        |        |      VPC
        |        |
        |        +-- Nhiều VPC / multi-account
        |                 |
        |                 |  [KEYWORDS]
        |                 |  multiple VPCs, shared services,
        |                 |  centralized routing
        |                 |
        |              Transit Gateway (TGW)
        |                 |
        |                 |  [NOTE]
        |                 |  - Hub-and-spoke
        |                 |  - Central routing / inspection
        |                 |
        |              Multiple VPCs
        |
        +-- Lâu dài / traffic lớn / low latency / compliance?
                 |
                 |  [KEYWORDS]
                 |  long-term, high throughput,
                 |  predictable performance,
                 |  compliance, no internet
                 |
              Direct Connect (DX)
                 |
                 |  [NOTE]
                 |  - Dedicated private line
                 |  - Stable bandwidth, low latency
                 |  - Thường đi kèm VPN backup
                 |
        (BẮT BUỘC phải có Virtual Interface – VIF)
                 |
                 |  [KEYWORDS]
                 |  private VIF, transit VIF, BGP
                 |
        Chỉ kết nối 1 Region hay nhiều Region?
                 |
        +-------------------------+---------------------------+
        |                         |                           |
     1 VPC                  Nhiều VPC                    Nhiều VPC
     1 Region               1 Region                    Nhiều Region
        |                         |                           |
        |  [KEYWORDS]             |  [KEYWORDS]               |  [KEYWORDS]
        |  simple hybrid          |  multi VPC                |  multi-region
        |                         |  single region             |  scale, reuse DX
        |
   Private VIF              Private VIF                  Transit VIF
        |                         |                           |
        |  [NOTE]                 |  [NOTE]                   |  [NOTE]
        |  - BGP routing          |  - Central routing        |  - Cross-region
        |                         |                           |
       VGW                Transit Gateway (TGW)        Direct Connect Gateway
        |                         |                           |
        |  [NOTE]                 |  [NOTE]                   |  [NOTE]
        |  - 1 VPC only           |  - Hub network            |  - Global DX hub
        |                         |                           |
       VPC                Multiple VPCs               Transit Gateway (TGW)
                                                              |
                                                              |  [NOTE]
                                                              |  - Hub-and-spoke
                                                              |
                                                        Multiple VPCs
                                                    (multi-account / multi-region)
