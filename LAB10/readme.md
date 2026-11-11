# LAB 1 – MÃ HÓA ĐỐI XỨNG CỔ ĐIỂN
**Java Swing (NetBeans GUI Builder) | Apache Maven | Microsoft SQL Server**

---

## 1. Thông tin sinh viên thực hiện

| Thông tin | Chi tiết |
| :--- | :--- |
| **Trường** | Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh |
| **Khoa** | Khoa Công nghệ Thông tin |
| **Học phần** | An toàn và Bảo mật thông tin |
| **Họ và tên sinh viên** | Đinh Ngọc Minh Khôi |
| **Mã số sinh viên** | 1150080140 |
| **Lớp học phần** | 11_ĐH_CNPM2 (DH_CNPM2) |

---

## 2. Video Demo Chương Trình (YouTube)

Xem video trình bày và chạy kiểm thử toàn bộ chức năng của bài thực hành Lab 1 tại đường dẫn bên dưới:

👉 **Link YouTube Demo:** [DÁN_LINK_YOUTUBE_CỦA_BẠN_VÀO_ĐÂY](https://youtu.be/lRE_ysqHl00)

*(Gợi ý: Bạn thay thế đường dẫn `https://www.youtube.com/watch?v=...` ở dòng trên bằng link video YouTube thực tế của bạn).*

---

## 3. Giới thiệu tổng quan dự án

Dự án `LAB1_MaHoaCoDien` (package `week_01`) cài đặt hoàn chỉnh hệ thống mã hóa, giải mã và phân tích thám mã cho 6 hệ mật mã đối xứng cổ điển trên giao diện đồ họa Java Swing, kết hợp lưu trữ nhật ký thao tác vào hệ quản trị cơ sở dữ liệu Microsoft SQL Server thông qua JDBC.

Hệ thống bao gồm 5 phần chính:
1. **Phần 1 – Chuẩn bị Project & Tiện ích tệp:** Cấu hình Maven (`pom.xml`) với thư viện `mssql-jdbc` và lớp tiện ích `FileTextIO.java` hỗ trợ đọc/ghi tệp văn bản `.txt` chuẩn UTF-8.
2. **Phần 2 – Sáu thuật toán cơ bản (Bài 1.1 – 1.6):**
   - **1.1. Mật mã Caesar (`CaesarCipher` / `frm_Caesar`):** Dịch chuyển ký tự theo công thức $C = (P + k) \pmod{26}$ và $P = (C - k) \pmod{26}$.
   - **1.2. Mật mã Vigenère (`VigenereCipher` / `frm_Vigenere`):** Mã hóa đa bảng dùng khóa từ lặp lại tuần hoàn theo công thức $C_i = (P_i + K_{i \bmod m}) \pmod{26}$.
   - **1.3. Mật mã Rail Fence (`RailFenceCipher` / `frm_RailFence`):** Mã hóa hoán vị theo đường zíc-zắc trên $k$ hàng ($rails \ge 2$).
   - **1.4. Mật mã Playfair (`PlayfairCipher` / `frm_Playfair`):** Mã hóa thay thế cặp 2 chữ cái (digram) trên ma trận khóa kích thước $5 \times 5$ ($I$ và $J$ dùng chung ô, đệm bằng ký tự $X$).
   - **1.5. Mật mã Transposition (`TranspositionCipher` / `frm_Transposition`):** Mã hóa hoán vị cột dựa trên vector thứ tự đọc cột $1..m$.
   - **1.6. Mật mã Autokey (`AutokeyCipher` / `frm_Autokey`):** Mở rộng dòng khóa bằng cách nối trực tiếp bản rõ vào sau khóa mồi.
3. **Phần 3 – Bài nâng cao 01 (Chương trình tổng hợp & SQL Server):**
   - Tích hợp 6 thuật toán vào một giao diện duy nhất (`frm_TongHop`) thông qua `JComboBox`.
   - Cấu hình kết nối động tới SQL Server (`frm_KetNoiSQL`, `DbConfig`) và tự động ghi/đọc lịch sử thao tác (`ENCRYPT` / `DECRYPT`) vào bảng `dbo.history` bằng `PreparedStatement` chống SQL Injection (`HistoryDao`, `frm_LichSu`).
4. **Phần 4 – Bài nâng cao 02 (Phân tích / Tấn công mật mã cổ điển):**
   - Cài đặt 3 công cụ thám mã tự động trên `frm_TanCong` (`AttackDemos`): Vét cạn 26 khóa Caesar chấm điểm tần suất tiếng Anh, Phân tích thống kê tần suất chữ cái $A-Z$, và Khôi phục khóa Vigenère khi biết trước cặp bản rõ/bản mã ($K_i = (C_i - P_i) \pmod{26}$).
5. **Phần 5 – Bảng điều khiển chính (`frm_Menu`):**
   - Giao diện trang chủ điều hướng trực tiếp tới toàn bộ 9 cửa sổ chức năng độc lập (`DISPOSE_ON_CLOSE`).

---

## 4. Cấu trúc thư mục mã nguồn

```text
LAB1_TuLam/
├── pom.xml                                  # Cấu hình Maven, JDK 17 và thư viện mssql-jdbc
├── README.md                                # Tài liệu hướng dẫn dự án
└── src/main/java/week_01/
    ├── FileTextIO.java                      # Đọc/ghi tệp văn bản .txt (UTF-8) với JFileChooser
    ├── CaesarCipher.java                    # Thuật toán mật mã Caesar
    ├── VigenereCipher.java                  # Thuật toán mật mã Vigenère
    ├── RailFenceCipher.java                 # Thuật toán mật mã hàng rào Rail Fence
    ├── PlayfairCipher.java                  # Thuật toán mật mã ma trận 5x5 Playfair
    ├── TranspositionCipher.java             # Thuật toán mật mã hoán vị cột Transposition
    ├── AutokeyCipher.java                   # Thuật toán mật mã khóa tự sinh Autokey
    ├── CipherTongHop.java                   # Lớp điều phối chung cho 6 thuật toán
    ├── DbConfig.java                        # Quản lý thông tin và kết nối JDBC SQL Server
    ├── HistoryDao.java                      # Truy vấn INSERT và SELECT TOP (50) bảng dbo.history
    ├── AttackDemos.java                     # Ba kỹ thuật phân tích / tấn công mật mã cổ điển
    ├── frm_Caesar.java                      # JFrame Form Bài 1.1 - Caesar
    ├── frm_Vigenere.java                    # JFrame Form Bài 1.2 - Vigenère
    ├── frm_RailFence.java                   # JFrame Form Bài 1.3 - Rail Fence
    ├── frm_Playfair.java                    # JFrame Form Bài 1.4 - Playfair
    ├── frm_Transposition.java               # JFrame Form Bài 1.5 - Transposition
    ├── frm_Autokey.java                     # JFrame Form Bài 1.6 - Autokey
    ├── frm_TongHop.java                     # JFrame Form Bài nâng cao 01 - Tổng hợp 6 thuật toán
    ├── frm_KetNoiSQL.java                   # JFrame Form Cấu hình và kiểm tra kết nối SQL Server
    ├── frm_LichSu.java                      # JFrame Form Xem bảng lịch sử dbo.history
    ├── frm_TanCong.java                     # JFrame Form Bài nâng cao 02 - Tấn công mật mã
    └── frm_Menu.java                        # JFrame Form Menu chính khởi chạy toàn bộ chương trình
```

---

## 5. Bộ dữ liệu kiểm thử chuẩn (Test Cases)

### 5.1. Kiểm thử 6 thuật toán cơ bản và Form tổng hợp

| Bài | Thuật toán | Plaintext (Đầu vào) | Key (Khóa) | Ciphertext chuẩn |
| :---: | :--- | :--- | :--- | :--- |
| **1.1** | Caesar | `HELLO` | `3` | `KHOOR` |
| **1.2** | Vigenère | `HELLO` | `KEY` | `RIJVS` |
| **1.3** | Rail Fence | `HELLO WORLD` | `3` | `HOREL OLLWD` |
| **1.4** | Playfair | `HELLOWORLD` | `KEYWORD` | `GYIZSCOKCFBU` |
| **1.5** | Transposition | `BAOMAT` | `2,3,1` | `AAOTBM` |
| **1.6** | Autokey | `HELLO` | `KEY` | `RIJSS` |

### 5.2. Kiểm thử 3 kịch bản thám mã (Phần 4 – `frm_TanCong`)

| Thí nghiệm | Kỹ thuật tấn công | Dữ liệu đầu vào | Kết quả đầu ra |
| :---: | :--- | :--- | :--- |
| **TN 1** | Brute Force Caesar | Ciphertext: `KHOOR` | Khóa đúng $k = 3$, Bản rõ: `HELLO <== có nghĩa nhất` |
| **TN 2** | Frequency Analysis | Ciphertext: `PHHW PH DIWHU WKH WRJD SDUWB` | Tổng 23 chữ; `H` và `W` cao nhất (5 lần, $21.7\%$) $\Rightarrow k = 3$ |
| **TN 3** | Known Plaintext (Vigenère) | Plain: `HELLO` / Cipher: `RIJVS` | Khóa suy ra từng vị trí: `KEYKE` (chu kỳ khóa `KEY`) |

---

## 6. Hướng dẫn cài đặt và chạy chương trình

### Bước 1: Yêu cầu môi trường
1. **Java Development Kit (JDK):** Phiên bản 17 trở lên.
2. **IDE:** Apache NetBeans IDE (phiên bản 20 trở lên).
3. **Hệ quản trị CSDL:** Microsoft SQL Server Express & SQL Server Management Studio (SSMS), đã bật giao thức mạng **TCP/IP** (Port tĩnh `1433`) và chế độ đăng nhập **SQL Server and Windows Authentication mode**.

### Bước 2: Khởi tạo Cơ sở dữ liệu SQL Server
Mở SSMS và chạy đoạn kịch bản SQL sau để tạo Database, bảng `dbo.history` và tài khoản kết nối:

```sql
CREATE DATABASE LAB1_MaHoaCoDien;
GO

USE LAB1_MaHoaCoDien;
GO

CREATE TABLE dbo.history (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    created_at DATETIME2 DEFAULT SYSDATETIME(),
    algorithm NVARCHAR(50) NOT NULL,
    operation VARCHAR(10) NOT NULL CHECK (operation IN ('ENCRYPT', 'DECRYPT')),
    input_text NVARCHAR(MAX),
    result_text NVARCHAR(MAX)
);
GO

CREATE LOGIN lab1_user WITH PASSWORD = N'123456', CHECK_POLICY = OFF;
GO

USE LAB1_MaHoaCoDien;
GO
CREATE USER lab1_user FOR LOGIN lab1_user;
GO
ALTER ROLE db_owner ADD MEMBER lab1_user;
GO
```

### Bước 3: Biên dịch và khởi chạy trên NetBeans
1. Mở dự án trong Apache NetBeans.
2. Nhấn tổ hợp phím **`Shift + F11`** (*Clean and Build*) để Maven tải thư viện `mssql-jdbc` và biên dịch toàn bộ lớp mã nguồn.
3. Nhấn phím **`F6`** (*Run Project*) để mở cửa sổ điều hướng chính `frm_Menu` (hoặc nhấn `Shift + F6` trên từng file `frm_*.java` để chạy riêng từng bài).