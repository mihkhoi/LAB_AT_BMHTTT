# BÁO CÁO THỰC HÀNH AN TOÀN HỆ THỐNG THÔNG TIN

## LAB 1: EXAMINING SSH & TELNET IN WIRESHARK

---

### 1. THÔNG TIN SINH VIÊN

- **Họ và tên:** [Dinh Ngoc Minh Khoi]
- **Mã số sinh viên (MSSV):** [1150080140]
- **Lớp:** [DH_CNPM2]
- **Link Video Demonstration (YouTube):** [https://youtu.be/-QypBRxpeF8]

---

### 2. MÔ HÌNH VÀ MÔI TRƯỜNG TRIỂN KHAI

- **Mô hình mạng:** Client - Server - Attacker kết nối chung switch mạng nội bộ (Host-Only/LAN Segment).
- **Thông tin phân bổ địa chỉ IP:**
  - **Server:** `10.0.0.1/24` (Dịch vụ Telnet Port 23 & SSH Port 22)
  - **Client:** `10.0.0.2/24` (Sử dụng PuTTY)
  - **Attacker / Packet Capture:** `10.0.0.3/24` hoặc trực tiếp tại Client (Sử dụng Wireshark)

---

### 3. TÓM TẮT NỘI DUNG THỰC HIỆN

1. **Cấu hình mạng:** Thiết lập IP tĩnh trên 2 máy ảo Windows/Linux, kiểm tra độ trễ và tính liên thông bằng lệnh Ping.
2. **Triển khai Telnet Server:** Kích hoạt tính năng Telnet trên máy chủ, phân quyền tài khoản sinh viên vào nhóm `TelnetClients`.
3. **Phân tích giao thức Telnet bằng Wireshark:**
   - Kết nối PuTTY qua Port 23.
   - Dùng tính năng `Follow TCP Stream` trên Wireshark.
   - _Kết quả:_ Toàn bộ Username, Password và các lệnh điều khiển (`dir`, `whoami`) đều bị lộ dưới dạng Cleartext (văn bản rõ). Thử nghiệm đổi mật khẩu độ phức tạp cao vẫn bị bắt trọn vẹn.
4. **Triển khai SSH Server:** Khởi chạy OpenSSH Server trên máy chủ tại Port 22.
5. **Phân tích giao thức SSH bằng Wireshark:**
   - Bắt tay xác thực Host Key Fingerprint trên PuTTY.
   - Phân tích luồng dữ liệu trên Wireshark.
   - _Kết quả:_ 100% dữ liệu truyền tải sau pha bắt tay đều được mã hóa bằng thuật toán đối xứng, Wireshark chỉ ghi nhận các gói tin `Encrypted packet`.
6. **Trả lời toàn bộ câu hỏi phân tích:** Đánh giá chi tiết mô hình CIA, phân tích metadata, nguyên lý Host Key và đề xuất giải pháp Hardening SSH.

---
