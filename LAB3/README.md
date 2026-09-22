# Báo cáo Thực hành Lab 3: Nhận diện và Ứng phó các Mối đe dọa An toàn Thông tin

**Sinh viên thực hiện:** Đinh Ngọc Minh Khôi  
**Mã số sinh viên:** 1150080140  
**Mã lớp:** 11_ĐH_CNPM2  
**Link youtube:** https://youtu.be/p5XMinHuFZ4
**Môi trường thực hành:** Windows 11 25H2 x64 (OS build 26200.9445) trên VMware Workstation Pro 26H1, mạng Host-only.

## Giới thiệu

Kho lưu trữ này chứa các tài liệu, nhật ký (log) và báo cáo phân tích thuộc bài thực hành Lab 3 môn An toàn Thông tin. Mục tiêu của bài thực hành là thực hiện quy trình Baseline - Observe - Detect - Contain - Recover, đồng thời thu thập bằng chứng và phân tích các nhóm mối đe dọa an ninh mạng bao gồm: Malware, Tấn công mật khẩu (Brute Force, Keylogger), Backdoor/Persistence, DoS/DDoS, Sniffing/MITM và Social Engineering/Phishing.

## Cấu trúc thư mục

- `[MãLớp]-LAB3_MSSV-HoTen.docx`: Báo cáo chi tiết chứa phần trả lời 20 câu hỏi lý thuyết, phân tích và hình ảnh minh chứng thực hành trực tiếp từ máy ảo.
- `evidence_sha256.csv`: Tệp chứa mã băm SHA-256 của toàn bộ các file bằng chứng nhằm xác minh tính toàn vẹn của dữ liệu sau thời điểm thu thập.
- `Evidence/`: Thư mục chứa các tệp nhật ký tĩnh đã được làm sạch xuất ra từ quá trình thực hành (Sysmon logs, Autoruns logs, Network logs, CSV phân tích).

## Công cụ sử dụng

- **Hệ điều hành & Ảo hóa:** Windows 11, VMware Workstation Pro.
- **Bảo mật & Giám sát hệ thống:** Microsoft Defender Antivirus, Windows Event Log.
- **Bộ công cụ Sysinternals:** Sysmon (v15.22), Autoruns (v14.3), Process Explorer (v17.14).
- **Phân tích mạng & Kịch bản:** Wireshark (v4.6.8 + Npcap), Python (v3.14.7).

## Tiêu chuẩn Tuân thủ & An toàn (Disclaimer)

- Toàn bộ quá trình tạo tình huống và thu thập dấu vết được thực hiện hoàn toàn trong môi trường máy ảo (VM) cô lập (Network Adapter: Host-only).
- Kho lưu trữ này tuân thủ nghiêm ngặt quy định nộp bài: **Không** chứa thông tin xác thực thật (tài khoản/mật khẩu cá nhân), **không** chứa mã thông báo (token/cookie), **không** chứa tệp thực thi (.exe) và **không** tải lên các mẫu mã độc đã bị Defender cách ly.
- Mọi dữ liệu và tập lệnh đi kèm chỉ phục vụ duy nhất cho mục đích nghiên cứu học thuật và đánh giá môn học.
