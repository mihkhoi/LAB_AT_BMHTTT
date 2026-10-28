# \# LAB 9: TẤN CÔNG DNS NỘI BỘ VÀ ĐẦU ĐỘC BỘ NHỚ ĐỆM (LOCAL DNS ATTACK \& DNS CACHE POISONING)

# 

# > Khảo sát lỗ hổng kiến trúc giao thức DNS: Thực nghiệm tấn công giả mạo phản hồi DNS trực tiếp (Direct DNS Spoofing) và đầu độc bộ nhớ đệm máy chủ đệ quy BIND9 (DNS Cache Poisoning) trên hạ tầng phân vùng mạng an toàn pfSense.

# 

# \---

# 

# \## 📌 THÔNG TIN TÁC GIẢ \& HỌC PHẦN

# 

# \- \*\*Sinh viên thực hiện:\*\* Đinh Ngọc Minh Khôi\[cite: 33]

# \- \*\*Mã số sinh viên (MSSV):\*\* 1150080140\[cite: 33]

# \- \*\*Lớp chuyên ngành:\*\* DH\_CNPM2\[cite: 33]

# \- \*\*Trường:\*\* Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh\[cite: 33]

# \- \*\*Học phần:\*\* An toàn thông tin mạng \& Quản trị hệ thống bảo mật\[cite: 33]

# \- \*\*Đường link YouTube video demo / báo cáo:\*\* \[https://www.youtube.com/watch?v=xKE9w6OSPyU] \*(Dán đường dẫn video thực hành của bạn vào đây)\*\[cite: 33]

# 

# \---

# 

# \## 🎯 MỤC TIÊU BÀI LAB

# 

# 1\. \*\*Kiểm chứng cơ chế phân giải tên miền cục bộ (Task 1):\*\* Phân tích thứ tự ưu tiên tra cứu tên miền của hệ điều hành Linux giữa bảng ánh xạ tĩnh `/etc/hosts` (qua thư viện `glibc` / `/etc/nsswitch.conf`) và máy chủ phân giải tập trung DNS Server.

# 2\. \*\*Thực nghiệm tấn công giả mạo phản hồi DNS trực tiếp (Task 2 - Direct DNS Spoofing):\*\* Sử dụng thư viện Scapy trên Kali Linux ở chế độ Promiscuous Mode để bắt gói tin DNS Query của Client, bóc tách Transaction ID trong thời gian thực và chế tạo phản hồi giả mạo để chiếm đoạt lưu lượng truy cập web.

# 3\. \*\*Thực nghiệm tấn công đầu độc bộ nhớ đệm máy chủ đệ quy (Task 3 - DNS Cache Poisoning):\*\* Tấn công trực tiếp vào máy chủ trung gian BIND9 Resolver, đánh lừa máy chủ lưu trữ bản ghi độc hại vào RAM (Cache), từ đó mở rộng phạm vi kiểm soát lên toàn bộ các thiết bị trong mạng nội bộ.

# 4\. \*\*Phân tích lưu lượng và đối chuẩn bảo mật:\*\* Sử dụng Wireshark để phân tích cấu trúc gói tin DNS, thẩm định tính trùng khớp của Transaction ID / UDP Port, kiểm tra tính dai dẳng (Persistence) theo TTL và đề xuất các giải pháp kiến trúc phòng vệ toàn diện (DNSSEC, DoT/DoH, DAI, RPZ).

# 

# \---

# 

# \## 🌐 MÔ HÌNH HỆ THỐNG \& ĐỊA CHỈ IP

# 

# \### 1. Phân bổ các máy ảo (VirtualBox)

# 

# | Tên máy ảo | Hệ điều hành | Cấu hình Card mạng VirtualBox | Địa chỉ IPv4 | Vai trò kỹ thuật |

# | :--- | :--- | :--- | :--- | :--- |

# | \*\*pfSense-Lab\*\* | FreeBSD (pfSense CE) | • Adapter 1: Host-only (`192.168.56.254`)<br>• Adapter 2: Internal (`dns-lab` / LAN)<br>• Adapter 3: Internal (`lab9-dmz` / DMZ) | `10.9.0.1/24` (LAN Gateway)<br>`172.16.6.1/24` (DMZ Gateway) | Firewall Gateway, định tuyến giữa phân vùng mạng LAN và DMZ, kiểm soát luồng dữ liệu liên mạng. |

# | \*\*DNS-SERVER\*\* | Ubuntu Server 24.04 LTS | Adapter 1: Internal (`dns-lab`) | `10.9.0.10/24` | Máy chủ phân giải đệ quy và chuyển tiếp nội bộ (BIND9 Forwarding Resolver). |

# | \*\*CLIENT-A\*\* | Ubuntu Client / Desktop | Adapter 1: Internal (`dns-lab`) | `10.9.0.100/24` | Máy trạm người dùng (Victim) phục vụ việc phát sinh truy vấn DNS và kiểm thử truy cập web. |

# | \*\*KALI\*\* | Kali Linux 2026.x | Adapter 1: Internal (`dns-lab`) | `10.9.0.200/24` | Máy tấn công (Attacker) bật Promiscuous Mode, chạy script Scapy giả mạo DNS và Web Server bẫy. |

# | \*\*APP-TEST\*\* | Ubuntu Server 24.04 LTS | Adapter 1: Internal (`lab9-dmz`) | `172.16.6.20/24` | Máy chủ Authoritative DNS tại DMZ chạy script `auth\_dns\_delayed.py` mô phỏng độ trễ Internet (1.5s). |

# 

# \### 2. Danh mục dịch vụ và thông số kỹ thuật then chốt

# 

# | Tên thành phần | Cổng dịch vụ / Giao thức | Giá trị \& Thiết lập | Mục đích sử dụng trong kịch bản Lab |

# | :--- | :--- | :--- | :--- |

# | \*\*BIND9 Resolver\*\* | Cổng 53 (UDP/TCP) | `named.conf.options` (`forward only -> 172.16.6.20`) | Tiếp nhận yêu cầu tra cứu từ Client, chuyển tiếp ra DMZ và lưu kết quả vào bộ đệm. |

# | \*\*Web Server Giả lập\*\* | Cổng 80 (TCP) | `python3 -m http.server 80` (`\~/LAB9/fake\_site/`) | Trang cảnh báo màu đỏ của Attacker để kiểm chứng việc người dùng bị dẫn dụ sang IP `10.9.0.200`. |

# | \*\*Promiscuous Mode\*\* | Tầng liên kết dữ liệu (Layer 2) | `Allow VMs` (trên card `eth0` máy KALI) | Cho phép card mạng của Attacker nghe trộm toàn bộ lưu lượng khung truyền trong switch ảo. |

# | \*\*Script dns\_spoof\_lab9.py\*\* | Scapy Python3 | `--mode victim` và `--mode resolver` | Bóc tách Transaction ID theo thời gian thực và chế tạo gói phản hồi DNS giả mạo để thắng cuộc đua. |

# 

# \---

# 

# \## 🛡️ BẢNG 5 FIREWALL RULES TRÊN GIAO DIỆN DNSLAB (PFSENSE)

# 

# Bộ quy tắc tường lửa được thiết lập trên giao diện `DNSLAB` (cổng opt2) nhằm kiểm soát chặt chẽ luồng lưu lượng liên mạng giữa dải LAN và DMZ:

# 

# | Thứ tự | Hành động (Action) | Nguồn (Source) | Đích (Destination) | Cổng dịch vụ | Mục đích bảo mật |

# | :---: | :---: | :---: | :---: | :---: | :--- |

# | \*\*Rule 1\*\* | Pass (IPv4 TCP/UDP) | `10.9.0.10` (DNS-SERVER) | `172.16.6.20` (AUTH-DNS) | `53 (DNS)` | Cho phép máy chủ BIND9 chuyển tiếp truy vấn phân giải sang máy chủ Authoritative tại DMZ. |

# | \*\*Rule 2\*\* | Pass (IPv4 TCP) | `DNSLAB subnets` | `172.16.6.10` (Web DMZ) | `80 (HTTP)` | Cho phép người dùng truy cập máy chủ Web hợp lệ trong phân vùng DMZ. |

# | \*\*Rule 3\*\* | Block (IPv4 \*) | `DNSLAB subnets` | `10.6.0.0/24` | `\*` | Cách ly hoàn toàn, ngăn chặn truy cập trái phép sang dải mạng LAB8 LAN-DOMAIN. |

# | \*\*Rule 4\*\* | Block (IPv4 \*) | `DNSLAB subnets` | `192.168.56.0/24` | `\*` | Chặn đứng mọi lưu lượng từ người dùng truy cập vào phân vùng quản trị mạng (MGMT). |

# | \*\*Rule 5\*\* | Block (IPv4 \*) | `DNSLAB subnets` | `\*` | `\*` | \*\*DEFAULT ISOLATION:\*\* Khóa mặc định toàn bộ lưu lượng còn lại ngoài phạm vi bài thực hành. |

# 

# \---

# 

# \## ⚙️ CẤU HÌNH DỊCH VỤ \& KỊCH BẢN TẤN CÔNG (SCAPY \& BIND9)

# 

# \### 1. Cấu hình BIND9 Resolver (`/etc/bind/named.conf.options`)

# 

# Máy chủ DNS-SERVER đóng vai trò Forwarding Resolver, tắt tính năng DNSSEC để phục vụ việc khảo sát lỗ hổng và chỉ định đường dẫn lưu cache dump:

# 

# ```text

# options {

# &#x20;   directory "/var/cache/bind";

# &#x20;   allow-query { any; };

# &#x20;   forwarders {

# &#x20;       172.16.6.20;

# &#x20;   };

# &#x20;   forward only;

# &#x20;   dnssec-validation no;

# &#x20;   listen-on { any; };

# &#x20;   listen-on-v6 { none; };

# &#x20;   dump-file "/var/cache/bind/dump.db";

# };

