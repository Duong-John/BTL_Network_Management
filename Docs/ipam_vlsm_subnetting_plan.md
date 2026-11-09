# BẢNG QUY HOẠCH ĐỊA CHỈ IPAM & CHIA MẠNG CON VLSM
**Môn học:** Quản trị mạng (CO3151)  
**Nhóm:** Nhóm trưởng Nguyễn Ánh Dương (MSSV: `2310596` $\rightarrow$ Hằng số $M = 96$)  
**Dải mạng gốc (Classless Root Block):** `10.96.0.0/16` (Tổng cộng $2^{16} = 65.536$ địa chỉ IP)

---

## 1. Các Hằng Số Cấu Hình Bắt Buộc Theo Đề Bài ($M = 96$)

Các thành viên tuyệt đối tuân thủ các thông số định danh dưới đây khi cấu hình thiết bị trong cả **BTL1** và **BTL2**:

* **Hằng số nhóm ($M$):** `96`
* **Hậu tố Hostname bắt buộc:** `-96` (Ví dụ: `WAN-R1-96`, `DSW1-96`, `ASW1-96`)
* **Enable Secret chuẩn đề bài:** `Pass_96@CO3151` *(Lưu ý thống nhất dùng mật khẩu này trên toàn bộ thiết bị)*
* **Tài khoản Quản trị SSH / AAA (RADIUS):**
  * **Username:** `admin_96`
  * **Password / Secret:** `Pass_96@CO3151` (hoặc `Admin@96!` thống nhất trong file Inventory)
* **HSRP Group ID:** `96` (Câu lệnh chuẩn: `standby 96 ip 10.96.<VLAN>.254`)
* **SNMPv3 User / Community String (BTL2):** `snmp_user_96` / `community_96`
* **Tên Dashboard Zabbix (BTL2):** `NMS_Dashboard_Group_96`
* **Tên Script Python & Thư mục Backup (BTL2):** `backup_config_96.py`, `push_vlan_96.py` $\rightarrow$ Lưu vào `/backups/Group_96/`

---

## 2. Bảng Phân Bổ Mạng Con VLSM Tổng Thể (Từ Dải Gốc `10.96.0.0/16`)

Sử dụng kỹ thuật **VLSM (Variable Length Subnet Masking)**, dải địa chỉ `10.96.0.0/16` được quy hoạch thành các phân vùng mạng chức năng (Data/User VLANs, Server Farm, Management VLAN, Loopback Router-ID và các kết nối Point-to-Point Lớp 3) như sau:

| STT | Phân vùng Mạng (Segment) | VLAN ID | Địa chỉ Mạng (Network ID) | Prefix / Subnet Mask | Wildcard Mask (OSPF/ACL) | Dải IP Khả dụng (Usable Host Range) | Địa chỉ Broadcast | Default Gateway (HSRP VIP) | Số Host Tối đa |
| :-: | :--- | :-: | :--- | :--- | :--- | :--- | :--- | :--- | :-: |
| 1 | **IT / Admin (Sales)** | `10` | `10.96.10.0` | `/24` (`255.255.255.0`) | `0.0.0.255` | `10.96.10.1` – `10.96.10.253` | `10.96.10.255` | **`10.96.10.254`** | 254 |
| 2 | **Users (Engineering)** | `20` | `10.96.20.0` | `/24` (`255.255.255.0`) | `0.0.0.255` | `10.96.20.1` – `10.96.20.253` | `10.96.20.255` | **`10.96.20.254`** | 254 |
| 3 | **Server Farm Block** | `30` | `10.96.30.0` | `/24` (`255.255.255.0`) | `0.0.0.255` | `10.96.30.1` – `10.96.30.253` | `10.96.30.255` | **`10.96.30.254`** | 254 |
| 4 | **Management Network** | `99` | `10.96.99.0` | `/24` (`255.255.255.0`) | `0.0.0.255` | `10.96.99.1` – `10.96.99.253` | `10.96.99.255` | **`10.96.99.254`** | 254 |
| 5 | **Loopback0 (`WAN-R1-96`)** | L3 | `10.96.254.1` | `/32` (`255.255.255.255`) | `0.0.0.0` | `10.96.254.1` (Host Route) | - | OSPF RID `WAN-R1` | 1 |
| 6 | **Loopback0 (`WAN-R2-96`)** | L3 | `10.96.254.2` | `/32` (`255.255.255.255`) | `0.0.0.0` | `10.96.254.2` (Host Route) | - | OSPF RID `WAN-R2` | 1 |
| 7 | **Loopback0 (`DSW1-96`)** | L3 | `10.96.254.3` | `/32` (`255.255.255.255`) | `0.0.0.0` | `10.96.254.3` (Host Route) | - | OSPF RID `DSW1` | 1 |
| 8 | **Loopback0 (`DSW2-96`)** | L3 | `10.96.254.4` | `/32` (`255.255.255.255`) | `0.0.0.0` | `10.96.254.4` (Host Route) | - | OSPF RID `DSW2` | 1 |
| 9 | **P2P #1: `WAN-R1` $\leftrightarrow$ `DSW1`** | L3 | `10.96.255.0` | `/30` (`255.255.255.252`) | `0.0.0.3` | `10.96.255.1` – `10.96.255.2` | `10.96.255.3` | Point-to-Point OSPF | 2 |
| 10 | **P2P #2: `WAN-R1` $\leftrightarrow$ `DSW2`** | L3 | `10.96.255.4` | `/30` (`255.255.255.252`) | `0.0.0.3` | `10.96.255.5` – `10.96.255.6` | `10.96.255.7` | Point-to-Point OSPF | 2 |
| 11 | **P2P #3: `WAN-R2` $\leftrightarrow$ `DSW2`** | L3 | `10.96.255.8` | `/30` (`255.255.255.252`) | `0.0.0.3` | `10.96.255.9` – `10.96.255.10` | `10.96.255.11` | Point-to-Point OSPF | 2 |
| 12 | **P2P #4: `WAN-R2` $\leftrightarrow$ `DSW1`** | L3 | `10.96.255.12` | `/30` (`255.255.255.252`) | `0.0.0.3` | `10.96.255.13` – `10.96.255.14` | `10.96.255.15` | Point-to-Point OSPF | 2 |

---

## 3. Bảng Thiết Kế Chia Tải Chéo HSRP (Group 96) & Đồng Bộ RSTP Root Bridge

Theo tiêu chuẩn thiết kế từ Slide Chương 2 & Chương 3, thiết bị giữ vai trò **HSRP Active Gateway** cho VLAN nào thì **bắt buộc phải là RSTP Root Bridge Primary** cho chính VLAN đó để tối ưu đường đi của gói tin Lớp 2 và Lớp 3:

| VLAN | Tên Phân vùng | IP SVI trên `DSW1-96` | IP SVI trên `DSW2-96` | Virtual IP (VIP - Grp 96) | HSRP Active & RSTP Root Primary | HSRP Standby & RSTP Root Secondary | Cơ chế Tracking & Preempt |
| :-: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | `SALES` | `10.96.10.252/24` | `10.96.10.253/24` | **`10.96.10.254`** | **`DSW1-96`**<br>*(HSRP Prio 110 / STP Prio 24576)* | **`DSW2-96`**<br>*(HSRP Prio 100 / STP Prio 28672)* | `standby 96 preempt`<br>`standby 96 track 1 decrement 20` |
| **VLAN 99** | `MANAGEMENT` | `10.96.99.252/24` | `10.96.99.253/24` | **`10.96.99.254`** | **`DSW1-96`**<br>*(HSRP Prio 110 / STP Prio 24576)* | **`DSW2-96`**<br>*(HSRP Prio 100 / STP Prio 28672)* | `standby 96 preempt`<br>`standby 96 track 1 decrement 20` |
| **VLAN 20** | `ENGINEERING` | `10.96.20.252/24` | `10.96.20.253/24` | **`10.96.20.254`** | **`DSW2-96`**<br>*(HSRP Prio 110 / STP Prio 24576)* | **`DSW1-96`**<br>*(HSRP Prio 100 / STP Prio 28672)* | `standby 96 preempt`<br>`standby 96 track 2 decrement 20` |
| **VLAN 30** | `SERVER_FARM` | `10.96.30.252/24` | `10.96.30.253/24` | **`10.96.30.254`** | **`DSW2-96`**<br>*(HSRP Prio 110 / STP Prio 24576)* | **`DSW1-96`**<br>*(HSRP Prio 100 / STP Prio 28672)* | `standby 96 preempt`<br>`standby 96 track 2 decrement 20` |

---

## 4. Bảng Cấp Phát IP Chi Tiết Cho Từng Thiết Bị (IP Allocation Table)

### 4.1. Khối Thiết Bị Mạng (Routers & Switches)
| Thiết bị (Hostname) | Cổng / Giao diện (Interface) | Địa chỉ IP & Prefix | Default Gateway / Ghi chú |
| :--- | :--- | :--- | :--- |
| **`WAN-R1-96`** | `Gi0/0` (WAN Outside) | `DHCP` (`192.168.122.x/24`) | Gateway: `192.168.122.1` (NAT Cloud) |
| | `Gi0/1` (To `DSW1 e0/0`) | `10.96.255.1/30` | OSPF Area 0 (`point-to-point`) |
| | `Gi0/2` (To `DSW2 e0/0`) | `10.96.255.5/30` | OSPF Area 0 (`point-to-point`) |
| | `Loopback0` | `10.96.254.1/32` | OSPF Router-ID & SSH Management |
| **`WAN-R2-96`** | `Gi0/0` (WAN Outside) | `DHCP` (`192.168.122.y/24`) | Gateway: `192.168.122.1` (NAT Cloud) |
| | `Gi0/1` (To `DSW2 e0/1`) | `10.96.255.9/30` | OSPF Area 0 (`point-to-point`) |
| | `Gi0/2` (To `DSW1 e0/1`) | `10.96.255.13/30` | OSPF Area 0 (`point-to-point`) |
| | `Loopback0` | `10.96.254.2/32` | OSPF Router-ID & SSH Management |
| **`DSW1-96`** | `Ethernet0/0` (To `WAN-R1 Gi0/1`) | `10.96.255.2/30` | `no switchport` - OSPF Area 0 |
| | `Ethernet0/1` (To `WAN-R2 Gi0/2`) | `10.96.255.14/30` | `no switchport` - OSPF Area 0 |
| | `Loopback0` | `10.96.254.3/32` | OSPF Router-ID |
| | `Vlan10`, `Vlan20`, `Vlan30`, `Vlan99` | `.252/24` ở mỗi VLAN | HSRP Group `96` (VIP `.254`) |
| **`DSW2-96`** | `Ethernet0/0` (To `WAN-R1 Gi0/2`) | `10.96.255.6/30` | `no switchport` - OSPF Area 0 |
| | `Ethernet0/1` (To `WAN-R2 Gi0/1`) | `10.96.255.10/30` | `no switchport` - OSPF Area 0 |
| | `Loopback0` | `10.96.254.4/32` | OSPF Router-ID |
| | `Vlan10`, `Vlan20`, `Vlan30`, `Vlan99` | `.253/24` ở mỗi VLAN | HSRP Group `96` (VIP `.254`) |
| **`ASW1-96`** | `Vlan99` (Management SVI) | `10.96.99.11/24` | `ip default-gateway 10.96.99.254` |
| **`ASW2-96`** | `Vlan99` (Management SVI) | `10.96.99.12/24` | `ip default-gateway 10.96.99.254` |
| **`ASW3-96`** | `Vlan99` (Management SVI) | `10.96.99.13/24` | `ip default-gateway 10.96.99.254` |
| **`ASW4-96`** | `Vlan99` (Management SVI) | `10.96.99.14/24` | `ip default-gateway 10.96.99.254` |

---

### 4.2. Khối Máy Trạm (End-User PCs) & Máy Chủ (Server Farm)
| Thiết bị | Giao diện (Interface) | VLAN | Địa chỉ IP & Prefix | Default Gateway (HSRP VIP) | DNS Server |
| :--- | :--- | :-: | :--- | :--- | :--- |
| **`PC1-SALES`** | `e0` (VPCS) | `10` | `10.96.10.11/24` | `10.96.10.254` | `8.8.8.8` |
| **`PC2-SALES`** | `e0` (VPCS) | `10` | `10.96.10.12/24` | `10.96.10.254` | `8.8.8.8` |
| **`PC3-ENG`** | `e0` (VPCS) | `20` | `10.96.20.11/24` | `10.96.20.254` | `8.8.8.8` |
| **`PC4-ENG`** | `e0` (VPCS) | `20` | `10.96.20.12/24` | `10.96.20.254` | `8.8.8.8` |
| **`SRV-ZABBIX-96`** | `bond0` (`ens3` + `ens4`) | `30` | `10.96.30.10/24` | `10.96.30.254` | `8.8.8.8, 1.1.1.1` |
| **`SRV-RADIUS-96`** | `bond0` (`ens3` + `ens4`) | `30` | `10.96.30.20/24` | `10.96.30.254` | `8.8.8.8, 1.1.1.1` |
| **`SRV-ANSIBLE-96`** | `bond0` (`ens3` + `ens4`) | `30` | `10.96.30.30/24` | `10.96.30.254` | `8.8.8.8, 1.1.1.1` |