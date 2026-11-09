# BẢNG QUY HOẠCH ĐẤU NỐI CÁP MẠNG (MASTER PATCHING MATRIX)
**Môn học:** Quản trị mạng (CO3151)  
**Nhóm:** Nhóm trưởng Nguyễn Ánh Dương (MSSV: `2310596` $\rightarrow$ Hằng số $M = 96$)  
**Nền tảng mô phỏng:** GNS3 Server (Cisco IOSv L3, Cisco IOU L2/L3, QEMU Ubuntu 22.04 LTS, VPCS)

---

## 1. Danh mục Định danh Thiết bị (Standard Hostnames)

Theo quy tắc đề bài (`[Device-Type]-M`), toàn bộ thiết bị trong sơ đồ phải đặt `hostname` chính xác như sau để phục vụ chấm tự động và kịch bản kiểm tra NetDevOps ở BTL2:

| Phân lớp (Tier) | Nhãn trên Sơ đồ | Hostname Cấu hình (CLI) | Loại Image trên GNS3 | Người phụ trách chính |
| :--- | :--- | :--- | :--- | :--- |
| **WAN / Core** | WAN Border Router 1 | `WAN-R1-96` | Cisco IOSv L3 | Dương (TV1) |
| **WAN / Core** | WAN Border Router 2 | `WAN-R2-96` | Cisco IOSv L3 | Dương (TV1) |
| **Distribution** | Multilayer Switch 1 | `DSW1-96` | Cisco IOU L2 (Bật `ip routing`) | Nghĩa (TV3) |
| **Distribution** | Multilayer Switch 2 | `DSW2-96` | Cisco IOU L2 (Bật `ip routing`) | Nghĩa (TV3) |
| **Access (Users)** | Layer 2 Access Switch 1 | `ASW1-96` | Cisco IOU L2 | Huy (TV2) |
| **Access (Users)** | Layer 2 Access Switch 2 | `ASW2-96` | Cisco IOU L2 | Huy (TV2) |
| **Access (Servers)** | Layer 2 Access Switch 3 | `ASW3-96` | Cisco IOU L2 | Huy (TV2) |
| **Access (Servers)** | Layer 2 Access Switch 4 | `ASW4-96` | Cisco IOU L2 | Huy (TV2) |
| **End-User PCs** | VLAN 10 (Sales) | `PC1-SALES`, `PC2-SALES` | GNS3 VPCS | Kiên (TV4) |
| **End-User PCs** | VLAN 20 (Engineering) | `PC3-ENG`, `PC4-ENG` | GNS3 VPCS | Kiên (TV4) |
| **Server Farm** | NMS Zabbix Server | `SRV-ZABBIX-96` | QEMU Ubuntu 22.04 LTS (2 NICs) | Huy (TV2) |
| **Server Farm** | AAA RADIUS Server | `SRV-RADIUS-96` | QEMU Ubuntu 22.04 LTS (2 NICs) | Kiên (TV4) |
| **Server Farm** | NetDevOps Control Node | `SRV-ANSIBLE-96` | QEMU Ubuntu 22.04 LTS (2 NICs) | Dương (TV1) |

---

## 2. Quy ước Phân bổ Bay Cổng trên Switch Cisco IOU

Để đồng nhất cấu hình CLI và viết kịch bản Ansible/Python không bị nhầm cổng, tất cả các Switch IOU (`DSW1..2`, `ASW1..4`) tuân thủ quy tắc phân chia Module (Bay) sau:
* **Bay 0 (`Ethernet0/0` – `Ethernet0/3`):** Cổng Uplink / Downlink kết nối giữa các tầng (Core $\leftrightarrow$ Distribution $\leftrightarrow$ Access).
* **Bay 1 (`Ethernet1/0` – `Ethernet1/3`):** Cổng gộp kênh băng thông cao (**EtherChannel LACP**) và các đường Trunk mở rộng.
* **Bay 2 (`Ethernet2/0` – `Ethernet2/3`):** Cổng **Access Ports** kết nối trực tiếp xuống thiết bị đầu cuối (PCs, Ubuntu Servers).

---

## 3. Bảng Đấu Nối Chi Tiết Từng Phân Lớp

### 3.1. Tầng WAN & Kết nối L3 Core – Distribution (Northbound Links)
*Tất cả các kết nối giữa `WAN-R1/R2` và `DSW1/DSW2` là cổng định tuyến Lớp 3 (`no switchport` trên DSW), chạy giao thức định tuyến động **OSPF Process 96 (Area 0)**.*

| Thiết bị Nguồn | Cổng Nguồn | Thiết bị Đích | Cổng Đích | Chế độ Cổng (Port Mode) | Giao thức / Ghi chú |
| :--- | :---: | :--- | :---: | :--- | :--- |
| `NAT-CLOUD` | `nat0` | `SW-INTERNET` | `Ethernet0` | Unmanaged L2 | Cấp DHCP/NAT ra Internet thật (`192.168.122.0/24`) |
| `SW-INTERNET` | `Ethernet1` | `WAN-R1-96` | `Gi0/0` | L3 Routed (`ip nat outside`) | WAN Uplink Primary (Nhận IP DHCP từ NAT) |
| `SW-INTERNET` | `Ethernet2` | `WAN-R2-96` | `Gi0/0` | L3 Routed (`ip nat outside`) | WAN Uplink Secondary (Nhận IP DHCP từ NAT) |
| `WAN-R1-96` | `Gi0/1` | `DSW1-96` | `Ethernet0/0` | L3 Routed (`no switchport`) | Subnet P2P #1 (`10.96.255.0/30`) - OSPF Area 0 |
| `WAN-R1-96` | `Gi0/2` | `DSW2-96` | `Ethernet0/0` | L3 Routed (`no switchport`) | Subnet P2P #2 (`10.96.255.4/30`) - OSPF Area 0 |
| `WAN-R2-96` | `Gi0/1` | `DSW2-96` | `Ethernet0/1` | L3 Routed (`no switchport`) | Subnet P2P #3 (`10.96.255.8/30`) - OSPF Area 0 |
| `WAN-R2-96` | `Gi0/2` | `DSW1-96` | `Ethernet0/1` | L3 Routed (`no switchport`) | Subnet P2P #4 (`10.96.255.12/30`) - OSPF Area 0 |

---

### 3.2. Liên kết Giữa 2 Switch Distribution (High-Speed EtherChannel)
*Sử dụng giao thức **LACP (`channel-group 1 mode active`)** gộp 2 cổng vật lý thành `Port-channel 1` chạy Trunking 802.1Q.*

| Thiết bị Nguồn | Cổng Nguồn | Thiết bị Đích | Cổng Đích | Giao thức Gộp kênh | Cấu hình VLAN Cho phép (Allowed VLANs) |
| :--- | :---: | :--- | :---: | :--- | :--- |
| `DSW1-96` | `Ethernet1/0` | `DSW2-96` | `Ethernet1/0` | `Po1` (LACP Active/Active) | Trunk `dot1q` (VLAN `10, 20, 30, 99`) |
| `DSW1-96` | `Ethernet1/1` | `DSW2-96` | `Ethernet1/1` | `Po1` (LACP Active/Active) | Trunk `dot1q` (VLAN `10, 20, 30, 99`) |

---

### 3.3. Liên kết Tầng Distribution Xuống Tầng Access (Redundant L2 Trunks)
*Quy tắc vàng: Trên cả 4 Switch Access (`ASW1..4`), cổng **`Ethernet0/0` luôn nối lên `DSW1-96`** và cổng **`Ethernet0/1` luôn nối lên `DSW2-96`**. Cấu hình chuẩn 802.1Q Trunking và chạy **Rapid-PVST+ (RSTP)** kèm **Root Guard** ở phía cổng Downlink của DSW.*

| Thiết bị Distribution | Cổng DSW | Thiết bị Access | Cổng ASW | Chế độ Cổng (Port Mode) | Trạng thái RSTP Dự kiến (Theo Chia tải) |
| :--- | :---: | :--- | :---: | :--- | :--- |
| `DSW1-96` | `Ethernet0/2` | `ASW1-96` | `Ethernet0/0` | 802.1Q Trunk (`10,20,30,99`) | Root Port (VLAN 10, 99) / Alt Port (VLAN 20, 30) |
| `DSW1-96` | `Ethernet0/3` | `ASW2-96` | `Ethernet0/0` | 802.1Q Trunk (`10,20,30,99`) | Root Port (VLAN 10, 99) / Alt Port (VLAN 20, 30) |
| `DSW1-96` | `Ethernet1/2` | `ASW3-96` | `Ethernet0/0` | 802.1Q Trunk (`10,20,30,99`) | Root Port (VLAN 10, 99) / Alt Port (VLAN 20, 30) |
| `DSW1-96` | `Ethernet1/3` | `ASW4-96` | `Ethernet0/0` | 802.1Q Trunk (`10,20,30,99`) | Root Port (VLAN 10, 99) / Alt Port (VLAN 20, 30) |
| `DSW2-96` | `Ethernet0/2` | `ASW1-96` | `Ethernet0/1` | 802.1Q Trunk (`10,20,30,99`) | Root Port (VLAN 20, 30) / Alt Port (VLAN 10, 99) |
| `DSW2-96` | `Ethernet0/3` | `ASW2-96` | `Ethernet0/1` | 802.1Q Trunk (`10,20,30,99`) | Root Port (VLAN 20, 30) / Alt Port (VLAN 10, 99) |
| `DSW2-96` | `Ethernet1/2` | `ASW3-96` | `Ethernet0/1` | 802.1Q Trunk (`10,20,30,99`) | Root Port (VLAN 20, 30) / Alt Port (VLAN 10, 99) |
| `DSW2-96` | `Ethernet1/3` | `ASW4-96` | `Ethernet0/1` | 802.1Q Trunk (`10,20,30,99`) | Root Port (VLAN 20, 30) / Alt Port (VLAN 10, 99) |

---

### 3.4. Liên kết Tầng Access Xuống End-User PCs & Server Farm (Southbound Edge)
*Bắt buộc bật `spanning-tree portfast` và `spanning-tree bpduguard enable` trên toàn bộ các cổng Access `Ethernet2/0 - 2` theo chuẩn Slide Chương 2 & Chương 3.*

#### A. Khối End-User PCs (`ASW1-96` & `ASW2-96`)
| Thiết bị Access | Cổng Switch | Thiết bị Đầu cuối | Cổng PC | VLAN Gán | Cấu hình Bảo mật Lớp 2 |
| :--- | :---: | :--- | :---: | :---: | :--- |
| `ASW1-96` | `Ethernet2/0` | `PC1-SALES` | `e0` | **VLAN 10** | `switchport mode access`, `portfast`, `bpduguard` |
| `ASW1-96` | `Ethernet2/1` | `PC3-ENG` | `e0` | **VLAN 20** | `switchport mode access`, `portfast`, `bpduguard` |
| `ASW2-96` | `Ethernet2/0` | `PC2-SALES` | `e0` | **VLAN 10** | `switchport mode access`, `portfast`, `bpduguard` |
| `ASW2-96` | `Ethernet2/1` | `PC4-ENG` | `e0` | **VLAN 20** | `switchport mode access`, `portfast`, `bpduguard` |

#### B. Khối Server Farm (`ASW3-96` & `ASW4-96` — Dual-Homed Linux Bonding `active-backup`)
| Thiết bị Access | Cổng Switch | Máy chủ Ubuntu 22.04 | Cổng VM (NIC) | VLAN Gán | Vai trò Card mạng trong `bond0` |
| :--- | :---: | :--- | :---: | :---: | :--- |
| `ASW3-96` | `Ethernet2/0` | `SRV-ZABBIX-96` | `Adapter 0` (`ens3`/`eth0`) | **VLAN 30** | Primary Active Slave (`mii-monitor 100ms`) |
| `ASW3-96` | `Ethernet2/1` | `SRV-RADIUS-96` | `Adapter 0` (`ens3`/`eth0`) | **VLAN 30** | Primary Active Slave (`mii-monitor 100ms`) |
| `ASW3-96` | `Ethernet2/2` | `SRV-ANSIBLE-96` | `Adapter 0` (`ens3`/`eth0`) | **VLAN 30** | Primary Active Slave (`mii-monitor 100ms`) |
| `ASW4-96` | `Ethernet2/0` | `SRV-ZABBIX-96` | `Adapter 1` (`ens4`/`eth1`) | **VLAN 30** | Standby Backup Slave |
| `ASW4-96` | `Ethernet2/1` | `SRV-RADIUS-96` | `Adapter 1` (`ens4`/`eth1`) | **VLAN 30** | Standby Backup Slave |
| `ASW4-96` | `Ethernet2/2` | `SRV-ANSIBLE-96` | `Adapter 1` (`ens4`/`eth1`) | **VLAN 30** | Standby Backup Slave |

*(Lưu ý: Tất cả các cổng không nằm trong bảng trên từ `Ethernet0/2..3`, `Ethernet1/0..3`, `Ethernet2/3`, `Ethernet3/0..3` trên các Switch Access phải được đưa vào `vlan 999` và `shutdown` để đảm bảo an ninh Lớp 2).*