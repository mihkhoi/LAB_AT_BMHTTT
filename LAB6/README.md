# BÀI THỰC HÀNH LAB 6: XÁC THỰC ACTIVE DIRECTORY VÀ CHÍNH SÁCH TRUY CẬP TRÊN PFSENSE

> **Học phần:** Thực hành An toàn hệ thống thông tin  
> **Sinh viên thực hiện:** Đinh Ngọc Minh Khôi  
> **Mã số sinh viên:** 1150080140  
> **Lớp chuyên ngành:** 11_ĐH_CNPM2  
> **Học kỳ / Năm học:** 2026  

---

## 🎥 VIDEO MINH CHỨNG THỰC HÀNH THỰC TẾ (DEMO VIDEO)

Toàn bộ quá trình triển khai cấu hình máy ảo, kết nối Active Directory, cấu hình Captive Portal, thiết lập bảng luật và kiểm thử thực nghiệm của sinh viên đã được ghi hình đầy đủ tại liên kết dưới đây:

- **Đường dẫn xem Video trực tiếp:** [https://youtu.be/_rO95Fe74bE](https://youtu.be/_rO95Fe74bE)
- **Tiêu đề video:** `lab6 1150080140 dinh ngoc minh khoi`
- **Thời lượng thực hành:** 01 giờ 10 phút (quay trực tiếp thao tác từng bước trên hệ thống)
- **Nội dung ghi hình:**
  1. Cấu hình 4 card mạng máy ảo pfSense-LAB6 trên Oracle VM VirtualBox.
  2. Cài đặt AD DS, DNS và tạo tài khoản `sv_a`, `sv_b` trên Windows Server DC01 (`10.6.0.10`).
  3. Cấu hình xác thực LDAP trên pfSense và kiểm tra bằng `Diagnostics > Authentication`.
  4. Triển khai Captive Portal Zone `LAB6_USERS` trên interface LAN và kiểm thử đăng nhập từ Client.
  5. Tạo Aliases cho Client và 4 site DMZ mô phỏng (`.10`, `.20`, `.30`, `.40`).
  6. Tạo Schedules (`LUNCH_WINDOW`, `WORK_HOURS`) và xây dựng Ruleset LAN phân cấp.
  7. Kiểm thử các kịch bản chặn/cho phép, kiểm tra Stateful Table, Reset States và trích xuất Firewall Logs.

---

## 1. MỤC TIÊU BÀI THỰC HÀNH

- Xây dựng hệ thống mạng phân vùng riêng biệt gồm 4 interface: **WAN**, **MGMT**, **LAN-DOMAIN** và **DMZ**.
- Tích hợp pfSense với máy chủ **Active Directory (Windows Server)** thông qua giao thức **LDAP (TCP 389)** để phục vụ xác thực người dùng tập trung.
- Triển khai **Captive Portal** trên phân vùng LAN, yêu cầu các máy trạm xác thực danh tính bằng tài khoản Domain (`sv_a`, `sv_b`) trước khi được cấp quyền truy cập mạng.
- Thiết lập các định danh **Firewall Aliases** cho máy trạm, nhóm chính sách và các website mô phỏng trong vùng DMZ.
- Áp dụng các chính sách tường lửa theo kịch bản: chặn trang tìm việc (Job Site), chặn trang nội dung cấm (Restricted Site), cho phép mạng xã hội (Social Site) theo khung giờ nghỉ trưa (**Schedule**) và giới hạn dịch vụ trong giờ hành chính.
- Kiểm soát luồng lưu lượng cô lập vùng DMZ, phân tích cơ chế **Stateful Inspection (State Table)**, phân biệt hành vi **Block vs Reject**, và đọc nhật ký kiểm tra gói tin trong **Firewall Logs**.

---

## 2. KIẾN TRÚC VÀ MÔ HÌNH MẠNG (TOPOLOGY)

### 2.1. Phân hoạch Card mạng pfSense (VirtualBox)

| Adapter | Chế độ VirtualBox (Network Mode) | Tên mạng gán | Vai trò / Interface | Địa chỉ IP / Cấu hình |
| :---: | :--- | :--- | :---: | :--- |
| **Adapter 1** | NAT Network | `LAB6-WAN` | WAN | Nhận DHCP từ `192.168.250.0/24` (Gateway: `192.168.250.1`) |
| **Adapter 2** | Host-Only Adapter | `VirtualBox Host-Only Ethernet Adapter` | MGMT | `192.168.56.254/24` (Chỉ dành cho máy thật truy cập WebGUI) |
| **Adapter 3** | Internal Network | `lab6-lan` | LAN | `10.6.0.1/24` (Gateway cho DC01 và các máy Client) |
| **Adapter 4** | Internal Network | `lab6-dmz` | DMZ | `172.16.6.1/24` (Gateway cho máy chủ DMZ-WEB) |

### 2.2. Danh sách thiết bị và máy chủ trong hệ thống

| Thiết bị / Máy ảo | Hệ điều hành khuyến nghị | Địa chỉ IPv4 tĩnh | Gateway / DNS | Vai trò / Dịch vụ cung cấp |
| :--- | :--- | :--- | :--- | :--- |
| **pfSense-LAB6** | pfSense CE 2.7.x | Đa interface | Theo từng cổng | Tường lửa biên, Captive Portal, LDAP Client |
| **DC01** | Windows Server 2019/2022 | `10.6.0.10/24` | GW: `10.6.0.1`<br>DNS: `10.6.0.10` | AD DS domain `lab6.test`, DNS Server, chứa user `sv_a`, `sv_b` |
| **Client-A** | Windows 10/11 | `10.6.0.21/24` | GW: `10.6.0.1`<br>DNS: `10.6.0.10` | Máy trạm đại diện cho người dùng A |
| **Client-B** | Windows 10/11 | `10.6.0.22/24` | GW: `10.6.0.1`<br>DNS: `10.6.0.10` | Máy trạm đại diện cho người dùng B |
| **DMZ-WEB** | Ubuntu Server 22.04 LTS | `172.16.6.10/24`<br>IP phụ: `.20`, `.30`, `.40` | GW: `172.16.6.1` | Máy chủ Nginx lắng nghe các IP mô phỏng danh mục website |
| **Máy thật** | Windows / macOS | `192.168.56.1/24` | Không GW / DNS | Quản trị thiết bị qua WebGUI `https://192.168.56.254` |

---

## 3. DANH MỤC ALIAS VÀ SCHEDULE ĐÃ CẤU HÌNH

### 3.1. Danh mục Firewall Aliases (`Firewall > Aliases`)

- **Host Aliases:**
  - `CLIENT_A`: `10.6.0.21` (Máy đại diện tài khoản A)
  - `CLIENT_B`: `10.6.0.22` (Máy đại diện tài khoản B)
  - `POLICY_CLIENTS`: `10.6.0.21, 10.6.0.22` (Nhóm máy trạm áp dụng chính sách chung)
- **Website Aliases (Mô phỏng trong DMZ):**
  - `GENERAL_SITE`: `172.16.6.10` (Website thông thường)
  - `JOB_SITE`: `172.16.6.20` (Trang tuyển dụng / việc làm)
  - `SOCIAL_SITE`: `172.16.6.30` (Trang mạng xã hội)
  - `RESTRICTED_SITE`: `172.16.6.40` (Trang nội dung cấm)
- **Port Aliases:**
  - `WEB_PORTS`: `80, 443` (Dịch vụ duyệt web HTTP và HTTPS)

### 3.2. Lịch trình thời gian (`Firewall > Schedules`)

- `LUNCH_WINDOW`: 11:30 - 13:00 (Thứ 2 đến Thứ 6) — Khung giờ nghỉ trưa cho phép truy cập `SOCIAL_SITE`.
- `WORK_HOURS`: 08:00 - 17:00 (Thứ 2 đến Thứ 6) — Giới hạn các dịch vụ Internet chỉ được dùng cổng web trong giờ làm việc.

---

## 4. BẢNG TẬP LUẬT TƯỜNG LỬA TỔNG HỢP TRÊN INTERFACE LAN

Các quy tắc được sắp xếp theo đúng cơ chế **First-Match** (ưu tiên từ trên xuống dưới):

1. **Rule 1 (Block):** Source `CLIENT_A` $\rightarrow$ Destination `JOB_SITE` | Port: `TCP/80` (Bật Log)
2. **Rule 2 (Block):** Source `POLICY_CLIENTS` $\rightarrow$ Destination `RESTRICTED_SITE` | Port: `TCP/80` (Bật Log)
3. **Rule 3 (Pass):** Source `CLIENT_B` $\rightarrow$ Destination `SOCIAL_SITE` | Port: `TCP/80` | Schedule: `LUNCH_WINDOW`
4. **Rule 4 (Block):** Source `CLIENT_B` $\rightarrow$ Destination `SOCIAL_SITE` | Port: `TCP/80` (Chặn khi ngoài giờ nghỉ)
5. **Rule 5 (Pass):** Source `POLICY_CLIENTS` $\rightarrow$ Destination `GENERAL_SITE` | Port: `TCP/80`
6. **Rule 6 (Pass):** Source `POLICY_CLIENTS` $\rightarrow$ Destination `Any` | Port: `WEB_PORTS` | Schedule: `WORK_HOURS`
7. **Rule 7 (Block):** Source `POLICY_CLIENTS` $\rightarrow$ Destination `Any` | Port: `Any` (Explicit Deny cuối ruleset, bật Log)

---

## 5. VIDEO BÀI GIẢNG THAM KHẢO THÊM TRÊN YOUTUBE

- **[pfSense 2.3 - Configure LDAP Authentication](https://www.youtube.com/watch?v=vT8MiaB3Wxo)** *(Minh Hoàng Lê)*: Hướng dẫn kết nối pfSense với Active Directory qua LDAP.
- **[pfSense Captive Portal w/ Active Directory Authentication](https://www.youtube.com/watch?v=aCgsEAfn36c)** *(TechCare IT Solutions)*: Cấu hình Captive Portal với backend LDAP.
- **[pfSense Time-Based Firewall Rules & Schedules](https://www.youtube.com/watch?v=oJ34QK070B4)** *(Frimley Computing)*: Tạo Schedule áp dụng chính sách theo giờ.
- **[Basic Setup and Configuring pfSense Firewall Rules](https://www.youtube.com/watch?v=bjr0rm93uVA)** *(Lawrence Systems)*: Nguyên lý xây dựng tập luật tường lửa và Aliases.
- **[Troubleshooting Firewall Rules & States With pfTop in pfSense](https://www.youtube.com/watch?v=eb1pTs7XamA)** *(Lawrence Systems)*: Kiểm tra State Table và xử lý sự cố.

---

## 6. CẤU TRÚC THƯ MỤC NỘP BÀI

```text
├── 11_DH_CNPM2-Lab6_1150080140-DinhNgocMinhKhoi.docx   # Báo cáo Word hoàn chỉnh (12 câu hỏi + bảng kết quả)
├── config.xml                                          # File cấu hình xuất từ Diagnostics > Backup & Restore
├── README.md                                           # File thuyết minh và link video thực hành YouTube
└── screenshots/                                        # Bộ 8 ảnh chụp màn hình minh chứng
    ├── 01_virtualbox_4adapters.png                    # Cấu hình 4 card mạng của pfSense-LAB6
    ├── 02_ldap_auth_success.png                        # Kết quả xác thực LDAP cho user sv_a / sv_b
    ├── 03_captive_portal_client.png                    # Giao diện Captive Portal trên trình duyệt máy trạm
    ├── 04_firewall_aliases.png                         # Danh sách IP và Port Aliases
    ├── 05_firewall_schedules.png                       # Danh mục lịch trình LUNCH_WINDOW và WORK_HOURS
    ├── 06_lan_firewall_rules.png                       # Toàn bộ ruleset phân cấp trên tab LAN
    ├── 07_client_test_results.png                      # Kiểm thử Client-A (General Site OK, Job Site blocked)
    └── 08_firewall_logs.png                            # Nhật ký System Logs Firewall ghi nhận gói tin bị chặn
```

---

## 7. CÁC LƯU Ý KỸ THUẬT QUAN TRỌNG KHI VẬN HÀNH

1. **Nguyên lý First-Match:** Luôn đặt các quy tắc cụ thể lên trên các quy tắc cho phép diện rộng.
2. **Cơ chế Schedule:** Rule Pass có gắn Schedule khi ngoài giờ sẽ biến mất chứ không tự đảo ngược thành Block, do đó bắt buộc phải có rule Block bên dưới để chặn ngoài giờ.
3. **Quản lý phiên Stateful:** Sau khi đổi rule, luôn vào `Diagnostics > States > Reset States` để dọn sạch phiên kết nối cũ.
4. **Sao lưu trước khi nộp:** Xuất file cấu hình `config.xml` qua `Diagnostics > Backup & Restore > Download configuration as XML` để nộp kèm.