# BÁO CÁO THỰC HÀNH AN TOÀN HỆ THỐNG THÔNG TIN

## BÀI LAB 5: THIẾT LẬP MÔ HÌNH TƯỜNG LỬA PFSENSE

- **Họ và tên sinh viên:** Đinh Ngọc Minh Khôi
- **Mã số sinh viên (MSSV):** 1150080140
- **Lớp:** 11_ĐH_CNPM2
- **Mã bài lab:** Bài Lab 5 (pfSense Configuration)
- **Link Youtube:** https://youtu.be/8CBUiuFUM1M

---

## A. TỔNG QUAN

### 1. Mục tiêu

- Xây dựng mô hình mạng có tường lửa pfSense bảo vệ LAN và vùng DMZ trên môi trường ảo hóa VMware Workstation / VirtualBox.
- Thực hiện cấu hình các interface WAN, LAN, DMZ, cơ chế NAT và các quy tắc tường lửa (firewall rule).
- Sử dụng máy ảo Windows Server làm Domain Controller trong LAN và máy chủ dịch vụ IIS trong DMZ để kiểm thử lưu lượng.
- Thực hành và làm chủ nhiều tình huống firewall thực tế để hiểu sâu về cơ chế lọc gói Stateful Firewall và NAT.

### 2. Môi trường & mô hình mạng

Hệ thống được triển khai trên máy thật chạy **Windows 11** thông qua phần mềm ảo hóa **VMware Workstation** (hoặc Oracle VirtualBox). Ba phân vùng mạng độc lập được định tuyến và kiểm soát bởi tường lửa pfSense như sau:

- **Phân vùng WAN (em0 - Bridged):** Kết nối mạng Internet upstream qua card mạng vật lý của máy thật.
- **Phân vùng LAN (em1 - Host-only / VMnet1):** Dải mạng `10.0.0.0/8`, chứa Domain Controller và các máy trạm quản trị.
- **Phân vùng DMZ (em2 - Host-only / VMnet2):** Dải mạng `172.16.0.0/16`, chứa máy chủ dịch vụ Web IIS.
