Họ và tên: Lê Thị Mỹ Hằng (LTMH)
Lớp: 11_TMĐT
MSSV: 1150070011
Bài thực hành: LAB 8 - Local DNS Attack (DNS Spoofing và Cache Poisoning trong mạng ảo cô lập)   
1. Lab hướng dẫn
   Kế thừa hệ thống tường lửa pfSense và các phân đoạn mạng sẵn có từ các bài Lab trước (gồm WAN, DMZ, MGMT và LAN-DOMAIN), sau đó mở rộng thêm một nhánh mạng nội bộ cô lập mới tên là DNS-LAB (10.9.0.0/24) trên cổng OPT2 của pfSense để thực hành tấn công DNS mà không làm ảnh hưởng tới hệ thống Domain Controller cũ.
   Thiết lập quy tắc tường lửa (Firewall Rules) trên pfSense để cô lập mạng DNS-LAB: không cho phép đi ra WAN trong lúc thử nghiệm tấn công, chặn truy cập sang mạng LAN-DOMAIN và MGMT, chỉ cho phép máy chủ DNS nội bộ hỏi máy chủ AUTH-DNS mô phỏng trong vùng DMZ qua cổng 53 và cho phép các máy trong mạng DNS-LAB truy cập máy chủ web hợp lệ DMZ-WEB qua cổng 80.   Cài đặt và cấu hình máy chủ DNS-SERVER (10.9.0.10) chạy dịch vụ BIND9 trên Ubuntu Server 24.04 làm máy chủ phân giải đệ quy/lưu đệm (recursive/caching resolver) cho mạng 10.9.0.0/24, đồng thời quản lý zone thuận/nghịch nội bộ và chuyển tiếp (forward) tên miền thử nghiệm sang máy chủ AUTH-DNS (172.16.6.20).
   Cấu hình máy trạm VICTIM (10.9.0.100) dùng Ubuntu 24.04 trỏ DNS duy nhất về DNS-SERVER, cấu hình máy ATTACKER Kali Linux (10.9.0.200) bật chế độ Promiscuous Mode (Allow VMs) trên VirtualBox để bắt gói tin bằng Wireshark, chạy web server giả mạo và thực thi script Scapy.
   Khởi chạy dịch vụ DNS có thẩm quyền mô phỏng (auth_dns_delayed.py) trên máy APP-TEST (172.16.6.20) với độ trễ phản hồi 1.5 giây để kiểm tra trạng thái Baseline trước khi tấn công.
   Thực hiện 3 nhiệm vụ tấn công DNS cốt lõi theo mô hình SEED Labs:Task 1: Chuyển hướng tên miền cục bộ trên máy nạn nhân bằng cách chỉnh sửa file /etc/hosts.
   Task 2: Giả mạo phản hồi DNS trực tiếp tới máy nạn nhân (Direct DNS Spoofing) bằng cách bắt gói truy vấn trên mạng LAN và gửi gói trả lời giả mạo thắng cuộc đua thời gian trước khi phản hồi thật về tới nơi.
   Task 3: Đầu độc bộ nhớ đệm của máy chủ DNS (DNS Cache Poisoning) bằng cách giả mạo phản hồi từ AUTH-DNS gửi về cho DNS-SERVER để lưu bản ghi giả vào cache của BIND9.   Thực hiện xóa cache, khôi phục lại cấu hình gốc sau khi thử nghiệm và phân tích các phương pháp phòng thủ (Hardening) như bật lại DNS COOKIE, tắt Promiscuous Mode, kiểm soát DNS egress và phân đoạn mạng
2. Bài thực hành
Tạo snapshot an toàn và gắn thêm card mạng thứ 5 (Internal Network: dns-lab) trên pfSense, cấu hình cổng OPT2 (DNSLAB) mang địa chỉ IP tĩnh 10.9.0.1/24 (không đặt gateway, không bật DHCP) và thiết lập xong 5 rule cô lập trên tường lửa pfSense.
Dựng và cấu hình IP tĩnh cho 3 máy ảo trong mạng DNS-LAB gồm: DNS-SERVER (10.9.0.10), VICTIM (10.9.0.100) và ATTACKER Kali (10.9.0.200), tắt toàn bộ card NAT tạm thời sau khi cài xong các gói công cụ để đảm bảo cô lập mạng.
Cấu hình hoàn chỉnh dịch vụ BIND9 trên DNS-SERVER với các file cấu hình zone thuận (db.lab9.test), zone nghịch (db.10.9.0), cấu hình forwarder về 172.16.6.20 và kiểm tra cú pháp thành công bằng named-checkconf, named-checkzone.
Chạy script mô phỏng AUTH-DNS có độ trễ 1.5s trên máy APP-TEST (172.16.6.20) và dựng trang web giả mạo tại cổng 80 trên máy Kali (10.9.0.200); kiểm tra Baseline từ máy VICTIM phân giải và truy cập đúng về máy chủ DMZ-WEB hợp lệ (172.16.6.10).
Thực hiện thành công Task 1: Chỉnh sửa file /etc/hosts trên VICTIM khiến lệnh getent và curl bị chuyển hướng về trang web giả trên Kali (10.9.0.200), trong khi lệnh dig hỏi trực tiếp DNS-SERVER vẫn trả về IP thật (172.16.6.10), sau đó khôi phục lại file /etc/hosts ban đầu.
Thực hiện thành công Task 2 (Direct DNS Spoofing): Chạy script ở chế độ victim trên Kali, bắt được gói truy vấn từ VICTIM và gửi gói phản hồi giả mạo khớp Transaction ID và UDP Port về cho VICTIM trước phản hồi thật, làm VICTIM truy cập vào trang web giả 10.9.0.200.
Thực hiện thành công Task 3 (DNS Cache Poisoning): Chạy script ở chế độ resolver trên Kali, gửi gói phản hồi giả mạo từ 172.16.6.20 về cho DNS-SERVER (10.9.0.10), kiểm tra file dump cache (/var/cache/bind/named_dump.db) xác nhận bản ghi giả 10.9.0.200 đã nằm trong bộ nhớ đệm của BIND9.
Bắt và phân tích được các gói tin DNS Query, Forged Response và Legitimate Response trên Wireshark bằng bộ filter chuẩn.
Hoàn tất bước khôi phục môi trường (flush cache trên BIND9 và trên VICTIM để kết quả phân giải trở lại 172.16.6.10, bật lại DNS COOKIE mặc định, chuyển Promiscuous Mode của Kali về Deny) và trả lời đầy đủ 10 câu hỏi phân tích của bài Lab.   
3. Lỗi gặp phải
