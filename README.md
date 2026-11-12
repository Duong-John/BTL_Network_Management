# Bài tập lớn 1 Quản trị Mạng
## Topology:
![Topology](Images/Topology.png)
## Thành viên - Nhiệm vụ:
> Note: Project folder là 'BTL_v0'
1. Dương:
- [x] Thiết kế Topology trên GNS3 Server (IOU L2/L3, Ubuntu 22.04).
- [x] Quy hoạch bảng IPAM/VLSM chi tiết cho dải 10.96.0.0/16 (VLAN 10, 20, 30, 99 + các subnet /30 P2P).
- [x] Cấu hình cặp WAN-R1-96 & WAN-R2-96: OSPF, NAT Overload ra Cloud Internet.
- [ ] Thiết lập chuẩn quản trị cơ sở trên toàn mạng: Hostname -96, Secret Pass_96@CO3151, SSHv2, SVI VLAN 99.
> Note: Khi mở console của 2 con Router sẽ yêu cầu đăng nhập, với username: admin_96 và password là: Pass_96@CO3151
2. Huy
- [x] Phụ trách 4 Switch Access (SW-A1-96 đến SW-A4-96).
- [ ] Cấu hình VLAN 10, 20, 30, 99, Trunking 802.1Q, đồng bộ Native VLAN.
- ~~[ ] Cấu hình EtherChannel LACP (mode active) cho các đường Uplink kép.~~
- [ ] Kích hoạt PortFast & BPDU Guard trên các cổng nối PC/Server.
- [ ] Vẽ sơ đồ cây RSTP (Root/Designated/Alternate Port) cho báo cáo.

3. Nghĩa
- [x] Phụ trách cặp Multilayer Switch (SW-D1-96, SW-D2-96) và đường High-Speed EtherChannel LACP nối giữa chúng.
- [x] Cấu hình SVI Inter-VLAN Routing (10, 20, 30, 99) và OSPF nối lên 2 Border Router.
- [x] Cấu hình HSRP Group 96 (VIP .254) chia tải chéo (D1 Active VLAN 10, 99; D2 Active VLAN 20, 30) kèm preempt & Interface Tracking.
- [x] Cấu hình RSTP Root Bridge Primary/Secondary đồng bộ với HSRP và bật Root Guard.

4. Kiên
- [x] Phụ trách các máy trạm PC (VLAN 10, 20) và 3 máy chủ Ubuntu (VLAN 30) ở mức kết nối mạng cơ bản.
- [ ] Thực thi kịch bản Failover Test (ngắt cổng Uplink, tắt Switch Distribution), đo thời gian hội tụ 1–2s bằng ping/traceroute.
- [ ] Xuất toàn bộ running-config, biên tập Báo cáo BTL1_BaoCao_NhomXX.pdf và Slide BTL1_Slide_NhomXX.pptx.


## Quy hoạch IPAM - VLSM:
[IPAM - VLMS](Docs/ipam_vlsm_subnetting_plan.md)

## Danh sách đấu nối cổng:
[Port mapping](Docs/master_patching_matrix.md)