# LAB 7: BẢO VỆ DỊCH VỤ WEB VỚI PFSENSE VÀ CỤM CONTENT FILTER

> Hệ thống phòng thủ theo chiều sâu (Defense-in-Depth): Kết hợp Stateful Firewall (Layer 4) và Web Proxy / Content Filter / Antivirus (Layer 7).

---

## 📌 THÔNG TIN TÁC GIẢ & HỌC PHẦN

- **Sinh viên thực hiện:** Đinh Ngọc Minh Khôi
- **Mã số sinh viên (MSSV):** 1150080140
- **Lớp chuyên ngành:** DH_CNPM2
- **Trường:** Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh
- **Học phần:** An toàn thông tin mạng & Quản trị hệ thống bảo mật
- **Đường link YouTube video demo / báo cáo:** [https://youtu.be/RSPvhqs-0fM] _(Dán đường dẫn video thực hành của bạn vào đây)_

---

## 🎯 MỤC TIÊU BÀI LAB

1. Xây dựng và kiểm soát luồng dữ liệu mạng thông qua tường lửa pfSense bằng hệ thống IP Aliases và Port Aliases.
2. Triển khai kiến trúc chống lách luật (Anti-bypass Web Proxy) qua bộ 7 Firewall Rules trên giao diện LAN của pfSense.
3. Thiết lập chuỗi dịch vụ bảo vệ web hai tầng trên Ubuntu Server:
   - **E2Guardian (Port 8080):** Tiếp nhận yêu cầu từ client, lọc tên miền, từ khóa trang web và kiểm soát định dạng tệp tin nguy hiểm.
   - **Squid (Port 3128 - Localhost):** Đóng vai trò backend caching proxy, chỉ nhận kết nối nội bộ từ E2Guardian và thay mặt client tải tài nguyên từ Internet.
   - **ClamAV Daemon:** Tự động rà quét chữ ký mã độc trên các tệp tin tải về trước khi chuyển giao cho người dùng.
4. Kiểm thử thực tế các kịch bản: chặn bypass, duyệt web hợp lệ và chặn tên miền nằm trong danh sách đen.

---

## 🌐 MÔ HÌNH HỆ THỐNG & ĐỊA CHỈ IP

### 1. Phân bổ các máy ảo (VirtualBox)

| Tên máy ảo       | Hệ điều hành            | Cấu hình Card mạng VirtualBox                                                                                                    | Địa chỉ IPv4                | Vai trò kỹ thuật                                                                  |
| :--------------- | :---------------------- | :------------------------------------------------------------------------------------------------------------------------------- | :-------------------------- | :-------------------------------------------------------------------------------- |
| **pfSense-LAB6** | FreeBSD (pfSense CE)    | • Adapter 1: Host-only (`192.168.56.254`)<br>• Adapter 2: Internal Network (`lab6-lan`)<br>• Adapter 3: NAT / Bridged (Internet) | `10.6.0.1/24` (LAN Gateway) | Firewall Gateway, định tuyến mạng, thực thi kiểm soát truy cập và chống bypass.   |
| **WEB-PROXY**    | Ubuntu Server 24.04 LTS | Adapter 1: Internal Network (`lab6-lan`)                                                                                         | `10.6.0.30/24`              | Cụm dịch vụ bảo vệ web: E2Guardian (cổng 8080), Squid (cổng 3128), ClamAV daemon. |
| **CLIENT-A**     | Ubuntu Server / Desktop | Adapter 1: Internal Network (`lab6-lan`)                                                                                         | `10.6.0.21/24`              | Máy trạm người dùng thuộc phân vùng LAN phục vụ quá trình kiểm thử an toàn mạng.  |

### 2. Danh mục Aliases cấu hình trên pfSense

| Tên Alias        | Loại (Type) | Giá trị cấu hình                   | Mục đích sử dụng trong chính sách                                     |
| :--------------- | :---------- | :--------------------------------- | :-------------------------------------------------------------------- |
| **LAB7_PROXY**   | Host(s)     | `10.6.0.30`                        | Gom nhóm địa chỉ máy chủ dịch vụ Proxy tập trung trong dải mạng LAN.  |
| **LAB7_CLIENTS** | Host(s)     | `10.6.0.21`, `10.6.0.22`           | Gom nhóm các máy trạm người dùng bị áp chính sách duyệt web bắt buộc. |
| **WEB_PORTS**    | Port(s)     | `80, 443`                          | Nhóm các cổng duyệt web tiêu chuẩn HTTP và HTTPS.                     |
| **MAIL_PORTS**   | Port(s)     | `25, 110, 143, 465, 587, 993, 995` | Nhóm các cổng gửi nhận thư điện tử theo yêu cầu chuẩn của bài lab.    |

---

## 🛡️ BẢNG 7 FIREWALL RULES TRÊN GIAO DIỆN LAN (PFSENSE)

Thứ tự các luật được sắp xếp theo nguyên tắc độ ưu tiên từ trên xuống dưới:

|   Thứ tự   |   Hành động (Action)   | Nguồn (Source) | Đích (Destination) | Cổng dịch vụ | Mục đích bảo mật                                                               |
| :--------: | :--------------------: | :------------: | :----------------: | :----------: | :----------------------------------------------------------------------------- |
| **Rule 1** |  Pass (IPv4 TCP/UDP)   | `LAN subnets`  |        `*`         |  `53 (DNS)`  | Cho phép phân giải tên miền DNS ra ngoài Internet hoặc DC01.                   |
| **Rule 2** |    Pass (IPv4 TCP)     | `LAB7_CLIENTS` |    `LAB7_PROXY`    |    `8080`    | Cho phép máy trạm kết nối đến dịch vụ E2Guardian trên WEB-PROXY.               |
| **Rule 3** |    Pass (IPv4 TCP)     | `LAN subnets`  |        `*`         | `MAIL_PORTS` | Cho phép lưu lượng email doanh nghiệp theo yêu cầu đề bài.                     |
| **Rule 4** |    Block (IPv4 TCP)    | `LAB7_CLIENTS` |        `*`         | `WEB_PORTS`  | **CHỐNG BYPASS:** Chặn máy trạm kết nối trực tiếp cổng 80/443 ra ngoài.        |
| **Rule 5** |    Pass (IPv4 TCP)     |  `LAB7_PROXY`  |        `*`         | `WEB_PORTS`  | Cho phép duy nhất máy WEB-PROXY đại diện kết nối web ra ngoài.                 |
| **Rule 6** |    Pass (IPv4 TCP)     |  `LAB7_PROXY`  |   `172.16.6.10`    | `80 (HTTP)`  | Cho phép proxy truy cập máy chủ web thử nghiệm trong phân vùng DMZ.            |
| **Rule 7** | Block / Deny (IPv4 \*) | `LAN subnets`  |        `*`         |     `*`      | **EXPLICIT DENY:** Chặn tất cả lưu lượng còn lại không khớp với các luật trên. |

---

## ⚙️ CẤU HÌNH DỊCH VỤ TRÊN WEB-PROXY (`10.6.0.30`)

### 1. Cấu hình Squid Backend (`/etc/squid/squid.conf`)

Squid chỉ lắng nghe trên localhost để ngăn chặn client kết nối trực tiếp bỏ qua bộ lọc:

```text
http_port 127.0.0.1:3128
http_access allow localhost
http_access deny all
```

### 2. Cấu hình danh sách chặn của E2Guardian (`/etc/e2guardian/lists/`)

- **Chặn tên miền (`bannedsitelist`):**
  ```text
  jobsite.local
  vietnamworks.com
  topcv.vn
  linkedin.com
  facebook.com
  tiktok.com
  ```
- **Chặn từ khóa tìm việc (`bannedphraselist`):**
  ```text
  <tim viec>
  <tuyen dung>
  <recruitment>
  <job search>
  ```
- **Chặn phần mở rộng tệp thực thi (`bannedextensionlist`):**
  ```text
  .exe
  .msi
  .bat
  .cmd
  .scr
  .ps1
  ```

### 3. Lệnh kiểm tra trạng thái dịch vụ và cổng lắng nghe

```bash
# Kiểm tra trạng thái 3 tiến trình
sudo systemctl is-active squid e2guardian clamav-daemon

# Kiểm tra cổng dịch vụ đang mở
sudo ss -tlnp | grep -E '8080|3128'
```

---

## 🧪 KỊCH BẢN KIỂM THỬ THỰC TẾ

### Kịch bản 1: Kiểm tra chống Bypass (Truy cập web trực tiếp)

- **Lệnh thực thi:**
  ```bash
  curl -I --connect-timeout 4 http://example.com
  ```
- **Kết quả:** `curl: (28) Resolving timed out after 4000 milliseconds` (hoặc `Connection timed out`).
- **Ý nghĩa:** Rule 4 trên pfSense đã chặn đứng kết nối trực tiếp cổng 80/443 của máy trạm.

### Kịch bản 2: Duyệt Web hợp lệ qua Proxy E2Guardian

- **Lệnh thực thi:**
  ```bash
  curl -x http://10.6.0.30:8080 -I http://example.com
  ```
- **Kết quả:** Trả về `HTTP/1.1 200 OK`.
- **Ý nghĩa:** E2Guardian tiếp nhận yêu cầu, chuyển qua Squid tải dữ liệu và trả lời client thành công.

### Kịch bản 3: Chặn tên miền cấm trong chính sách

- **Lệnh thực thi:**
  ```bash
  curl -x http://10.6.0.30:8080 -s http://vietnamworks.com | grep -i "denied"
  ```
- **Kết quả:** `<title>E2Guardian - Access Denied</title>`.
- **Ý nghĩa:** E2Guardian phát hiện tên miền nằm trong danh sách đen và chặn ngay lập tức.

---

## 📁 CẤU TRÚC THƯ MỤC NỘP BÀI

```text
├── BaoCao_LAB7_Web_Protection_DinhNgocMinhKhoi.docx   # File báo cáo Word hoàn chỉnh
├── README.md                                         # Tài liệu tóm tắt và hướng dẫn này
└── screenshots/                                      # Thư mục chứa 5 ảnh chụp minh chứng
    ├── minhchung1_aliases.png
    ├── minhchung2_firewall_rules.png
    ├── minhchung3_netplan_proxy.png
    ├── minhchung4_services_status.png
    └── minhchung5_curl_testing.png
```
