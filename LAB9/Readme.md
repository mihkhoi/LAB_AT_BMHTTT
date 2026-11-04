# BÁO CÁO THỰC HÀNH AN TOÀN HỆ THỐNG THÔNG TIN
## BÀI LAB 10: REMOTE DNS CACHE POISONING TRONG MÔI TRƯỜNG pfSense CÔ LẬP

### THÔNG TIN SINH VIÊN
* **Họ và tên:** Đinh Ngọc Minh Khôi
* **Mã số sinh viên (MSSV):** 1150080140
* **Lớp học phần:** DH_CNPM2
* **Khoa:** Công nghệ Thông tin - Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh
* **Thời gian thực hiện:** 04/11/2026
* **Link video demo (YouTube):** https://www.youtube.com/watch?v=9_HCQxomucE

---

## 1. MÔ HÌNH MẠNG VÀ PHÂN BỔ ĐỊA CHỈ IP

### 1.1. Bảng phân bổ IP
| Phân vùng mạng | Tên máy ảo / Thiết bị | Địa chỉ IP / Subnet | Interface / Gateway | Vai trò trong bài Lab |
| :--- | :--- | :--- | :--- | :--- |
| **pfSense** | Router / Firewall | `192.168.56.104/24`<br>`10.9.0.1/24`<br>`172.16.6.1/24`<br>`10.10.0.1/24` | `em0` (WAN/MGMT)<br>`em1` (DNSLAB)<br>`em2` (DMZ)<br>`em3` (ATTACKNET) | Định tuyến lưu lượng, áp đặt chính sách tường lửa cô lập phân vùng ATTACKNET. |
| **DNS-LAB** | `DNS-SERVER` | `10.9.0.10/24` | Gateway: `10.9.0.1` | Máy chủ DNS đệ quy phân giải tên miền (BIND 9.18 / Lab Resolver cổng 33333). |
| **DNS-LAB** | `VICTIM` | `10.9.0.100/24` | Gateway: `10.9.0.1`<br>DNS: `10.9.0.10` | Client người dùng cuối trong mạng nội bộ thực hiện truy vấn tên miền. |
| **DMZ** | `AUTH-DNS` (APP-TEST) | `172.16.6.20/24` | Gateway: `172.16.6.1` | Máy chủ Authoritative DNS quản trị zone `example.com` (IP gốc hợp lệ: `1.2.3.4`). |
| **ATTACKNET** | `ATTACKER` (Kali Linux) | `10.10.0.200/24` | Gateway: `10.10.0.1` | Máy tấn công từ xa, chạy kịch bản đầu độc cache và Web server giả mạo. |

---

## 2. QUÁ TRÌNH THỰC HIỆN VÀ CÁC LỆNH THỰC THI

### Bước 1: Cấu hình mạng và Firewall pfSense
1. Gán card mạng VirtualBox thứ 4 của pfSense vào Internal Network: `attack-net`.
2. Truy cập pfSense WebGUI (`https://192.168.56.104`):
   - **Interfaces > Assignments:** Thêm `em3` làm interface `ATTACKNET`.
   - **Interfaces > ATTACKNET:** Bật Interface, đặt Static IPv4 `10.10.0.1/24`, IPv4 Upstream gateway: `None`.
   - **Firewall > Rules > ATTACKNET:** Tạo rule cho phép IPv4 UDP tới `10.9.0.10:53` và TCP tới `10.10.0.200:80`.

### Bước 2: Thiết lập IP và kiểm tra thông mạng
* **Trên ATTACKER (Kali Linux):**
  ```bash
  sudo systemctl stop NetworkManager
  sudo ip link set eth0 up
  sudo ip addr replace 10.10.0.200/24 dev eth0
  sudo ip route replace default via 10.10.0.1 dev eth0
  ping -c 2 10.9.0.10