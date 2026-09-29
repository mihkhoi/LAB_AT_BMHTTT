# BÁO CÁO THỰC HÀNH AN TOÀN HỆ THỐNG THÔNG TIN

## LAB 4: KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

---

### 1. THÔNG TIN SINH VIÊN

- **Họ và tên:** Đinh Ngọc Minh Khôi
- **Mã số sinh viên (MSSV):** 1150080140
- **Lớp:** 11_DH_CNPM2
- **Link Video Demonstration (YouTube):** [https://youtu.be/ysVe6Nu5Ok0]


---

### 2. MÔ HÌNH VÀ MÔI TRƯỜNG TRIỂN KHAI

- **Mô hình mạng:** Mô hình mạng cô lập VirtualBox / VMware Host-Only (`192.168.56.0/24`), đảm bảo an toàn tuyệt đối và không phát tán lưu lượng quét ra mạng thực tế.
- **Thông tin phân bổ địa chỉ IP:**
  - **Máy thật (Windows Host):** `192.168.56.1/24` (Quản trị máy ảo, card Host-Only Adapter VMnet1)
  - **Máy quét (Kali Linux VM):** `192.168.56.10/24` (Cài đặt công cụ Nmap, thực thi các kỹ thuật quét)
  - **Máy đích kiểm thử (Metasploitable 2 VM):** `192.168.56.101/24` (Máy chủ Linux cố ý chứa nhiều dịch vụ và lỗ hổng phục vụ nghiên cứu)

---

### 3. TÓM TẮT NỘI DUNG THỰC HIỆN

1. **Thiết lập hạ tầng mạng Host-Only:**
   - Cấu hình dải mạng cô lập `192.168.56.0/24`, tắt chế độ Bridge/NAT trên máy đích Metasploitable 2 để cách ly an toàn.
   - Kiểm tra thông tuyến bằng lệnh `ping -c 4 192.168.56.101` từ Kali Linux sang máy đích.
   - Tạo bản sao lưu Snapshot `Before-LAB4` trước khi can thiệp hệ thống.

2. **Khám phá thiết bị mạng (Host Discovery):**
   - Thực hiện lệnh `sudo nmap -sn 192.168.56.0/24` để lập bản đồ các thiết bị đang hoạt động.
   - Ghi nhận đầy đủ địa chỉ IP, địa chỉ MAC, nhà sản xuất card mạng (Vendor) và nhận diện vai trò từng node mạng.

3. **Khảo sát cổng TCP và so sánh các kỹ thuật quét:**
   - **TCP Connect Scan (`-sT`):** Hoàn tất đầy đủ chu trình bắt tay ba bước qua socket `connect()`, không yêu cầu quyền root.
   - **SYN Stealth Scan (`-sS`):** Áp dụng kỹ thuật bắt tay nửa chừng (half-open), can thiệp raw packet ngắt phiên bằng RST, yêu cầu quyền quản trị cao nhất.
   - **FIN / Xmas / NULL Scan (`-sF`, `-sX`, `-sN`):** Thử nghiệm phản ứng dựa trên kẽ hở RFC 793, phân tích nguyên nhân kết quả trả về `open|filtered` trên hệ thống.
   - **ACK Scan (`-sA`):** Gửi cờ ACK nhằm phân tích chính sách tường lửa (phân biệt `unfiltered` và `filtered`) thay vì tìm cổng mở.

4. **Quét cổng UDP có kiểm soát:**
   - Thực hiện `sudo nmap -sU --top-ports 20` để khảo sát 20 cổng UDP thông dụng.
   - Phân tích cơ chế giới hạn tần suất gói tin lỗi (ICMP Rate Limiting) và lý do dịch vụ UDP thường hiển thị trạng thái `open|filtered`.

5. **Nhận diện dịch vụ, phiên bản và hệ điều hành:**
   - Dò quét phiên bản chuyên sâu bằng `-sV`: Thu thập banner nhận diện các dịch vụ nhạy cảm như FTP (vsftpd 2.3.4), SSH (OpenSSH 4.7p1), Web (Apache 2.2.8), SMB (Samba 3.X) và MySQL.
   - Nhận diện hệ điều hành bằng `-O`: Phân tích TCP/IP stack fingerprinting, suy đoán nhân Linux 2.6.X.
   - Quét tổng hợp chuyên sâu `-A` để kết hợp dò OS, Version, Traceroute và Script mặc định.

6. **Đánh giá nguy cơ bảo mật với Nmap Scripting Engine (NSE):**
   - Thu thập thông tin định danh máy chủ và miền qua `smb-os-discovery` trên cổng 445.
   - Kiểm tra nguy cơ tồn tại lỗ hổng EternalBlue bằng tập lệnh `smb-vuln-ms17-010`.

7. **Lập hồ sơ chứng cứ và xuất dữ liệu báo cáo:**
   - Xuất dữ liệu đa định dạng: văn bản thường (`-oN`), tệp máy đọc được (`-oX`) và Grepable (`-oG`).
   - Sử dụng công cụ `xsltproc` để chuyển đổi tệp XML sang giao diện web HTML trực quan.

8. **Thực nghiệm củng cố an toàn hệ thống (Before / After Hardening):**
   - Thực hiện quét trước can thiệp (`Before`): Cổng 80 đang mở chạy dịch vụ HTTP Apache.
   - Triển khai biện pháp phòng thủ: Dừng hoàn toàn dịch vụ web bằng lệnh `sudo /etc/init.d/apache2 stop`.
   - Quét đối chứng sau can thiệp (`After`): Cổng 80 chuyển từ `open` sang `closed`, chứng minh việc loại bỏ dịch vụ dư thừa giúp thu hẹp trực tiếp bề mặt tấn công.

9. **Trả lời toàn bộ câu hỏi phân tích lý thuyết và thực tiễn:** 
   - Đánh giá chi tiết sự khác biệt giữa các trạng thái cổng `open`, `closed`, `filtered`.
   - Phân tích sâu nguyên lý bắt tay TCP, cơ chế Raw Socket, giới hạn của OS Fingerprinting, và đề xuất 3 giải pháp phòng thủ chuyên sâu giảm thiểu bề mặt tấn công hệ thống.

---