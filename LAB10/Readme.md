Tên: LÊ THỊ MỸ HẰNG
Lớp: 11_TMĐT
MSSV: 1150070011

**LAB 10: REMOTE DNS CACHE POISONING TRONG MÔI TRƯỜNG pfSense CÔ LẬP  ** 
**1. Nội dung bài thực hành**
Mục tiêu: Hiểu sự khác biệt cốt lõi giữa Local DNS Spoofing (Lab 9) và Remote DNS Cache Poisoning / Kaminsky Attack.
Trong bài lab này, máy tấn công (Kali) bị tách sang một subnet riêng biệt (ATTACKNET) qua tường lửa pfSense nên không thể nghe lén (sniff) lưu lượng truy vấn hay biết trước Transaction ID (TXID) của DNS Resolver.
Cơ chế mô phỏng: Do BIND9 hiện đại đã mặc định bật ngẫu nhiên hóa cổng nguồn (Source-Port Randomization), bài lab sử dụng một resolver mô phỏng có chủ đích (vuln_resolver_lab10.py cố định cổng nguồn UDP 33333) kết hợp với cơ chế race harness trên AUTH-DNS/RELAY (auth_dns_relay_lab10.py) để tái hiện điều kiện tấn công vét cạn TXID 16-bit và tiêm bản ghi ủy quyền độc hại (NS + Glue A record) vào bộ nhớ đệm trong môi trường cô lập an toàn với tên miền .test. Sau khi kiểm chứng tấn công, hệ thống được khôi phục lại BIND9 hiện đại để thực hiện hardening và đối chiếu kết quả.
Sơ đồ IP và Thành phần mạng:pfSense DNSLAB: 10.9.0.1/24 (Gateway mạng DNS-LAB - Kế thừa)
DNS-SERVER: 10.9.0.10/24 (BIND9 / Lab Resolver - Kế thừa)   VICTIM: 10.9.0.100/24 (DNS Client kiểm thử - Kế thừa)
pfSense ATTACKNET: 10.10.0.1/24 (Gateway mạng Attacker - Mới thêm ở Adapter 6)
ATTACKER (Kali Linux): 10.10.0.200/24 (Remote Attacker + Evil DNS + Fake Web - Chuyển từ mạng DNSLAB sang ATTACKNET)
AUTH-DNS/RELAY (APP-TEST): 172.16.6.20/24 (Authoritative DNS hợp lệ + Race Harness - Kế thừa)
DMZ-WEB: 172.16.6.10/24 (Website hợp lệ ban đầu - Kế thừa)   HOST MGMT: 192.168.56.1 (Quản trị WebGUI pfSense - Kế thừa)
**2. Thực hành những gì**
**Bước 1: Chuẩn bị môi trường và chuyển đổi mô hình mạng từ Lab 9 sang Lab 10**
Tắt các máy ảo (pfSense, DNS-SERVER, VICTIM, Kali) và tạo snapshot LAB10_START_FROM_LAB9 trên pfSense cùng snapshot trên Kali trước khi đổi card mạng.
Trên VirtualBox của pfSense, bật Adapter 6 ở chế độ Internal Network với tên attack-net.
Khởi động pfSense, gán interface mới với tên ATTACKNET, đặt IP tĩnh 10.10.0.1/24 (tuyệt đối không chọn Upstream Gateway).
Trên máy Kali, tháo hoặc tắt card mạng dns-lab, gắn card Internal Network tên attack-net và đặt Promiscuous Mode = Deny. Khởi động Kali, cấu hình IP tĩnh 10.10.0.200/24, gateway 10.10.0.1 và kiểm tra kết nối bằng lệnh ip -br addr, ip route, ping -c 3 10.10.0.1.
Thiết lập Firewall Rules trên pfSense:
Trên tab DNSLAB: Cho phép 10.9.0.100 -> 10.9.0.10 (TCP/UDP 53); 10.9.0.10 -> 172.16.6.20 (UDP 53); 10.9.0.10 -> 10.10.0.200 (UDP 53 - phục vụ sau khi bị poison); 10.9.0.0/24 -> 10.10.0.200 (TCP 80 - để Victim mở web giả); chặn và ghi log (Block+Log) truy cập sang LAN 10.6.0.0/24, MGMT 192.168.56.0/24 và Any.
Trên tab ATTACKNET: Cho phép 10.10.0.200 -> 10.9.0.10 (TCP/UDP 53 - kích hoạt truy vấn); 10.10.0.200 -> 172.16.6.20 (UDP 5300 - gửi batch TXID tới race relay); chặn và ghi log (Block+Log) truy cập sang LAN 10.6.0.0/24, MGMT 192.168.56.0/24 và Any.   Nhấn Apply Changes và Reset States sau khi cấu hình rule.
**Bước 2: Thiết lập Baseline trên DNS-SERVER, AUTH-DNS và kiểm tra từ VICTIM**
Trên DNS-SERVER (10.9.0.10): Cài đặt gói cần thiết (bind9, dnsutils, python3-dnspython), sao lưu file named.conf.options và named.conf.local của Lab 9, chép cấu hình baseline của Lab 10 vào (chỉ cho phép 10.9.0.0/24 đệ quy, forward example.lab10.test về 172.16.6.20, bật dnssec-validation auto kèm validate-except cho namespace .test), kiểm tra bằng sudo named-checkconf và khởi động lại bind9.
Trên AUTH-DNS/RELAY (172.16.6.20): Dừng dịch vụ DNS cũ của Lab 9, cài python3-dnspython và chạy script sudo python3 auth_dns_relay_lab10.py (trả về 172.16.6.10 cho www.example.lab10.test, giữ cửa sổ chờ 3 giây trước khi trả NXDOMAIN cho các tên miền con ngẫu nhiên).
Trên VICTIM (10.9.0.100): Xóa cache và kiểm tra baseline bằng sudo resolvectl flush-caches, resolvectl query www.example.lab10.test và curl [http://www.example.lab10.test](http://www.example.lab10.test). Xác nhận kết quả trả về đúng IP hợp lệ 172.16.6.10.
**Bước 3: Xác nhận tính "Remote" của môi trường Lab**
Trên Kali, chạy lệnh sudo tcpdump -ni any udp port 53, đồng thời từ VICTIM thực hiện truy vấn www.example.lab10.test.
Chứng minh Kali hoàn toàn không nhìn thấy gói tin trao đổi giữa 10.9.0.10 và 172.16.6.20. Để phân tích quá trình tấn công ở các bước sau, thực hiện bắt gói trực tiếp trên DNS-SERVER hoặc APP-TEST với bộ lọc udp.port == 53 hoặc ip.addr == 10.9.0.10 && udp.port == 33333.
**Bước 4: Thực hiện Pha tấn công (Task 1 - Remote Cache Poisoning / Kaminsky Simulation)**
Trên DNS-SERVER: Tạm dừng BIND9 (sudo systemctl stop bind9) và khởi chạy resolver mô phỏng có lỗ hổng cố định cổng nguồn 33333 bằng lệnh sudo python3 vuln_resolver_lab10.py.
Trên Kali (10.10.0.200):Terminal 1: Chạy DNS độc hại ủy quyền cho domain mục tiêu bằng sudo python3 evil_dns_lab10.py.
Terminal 2: Tạo thư mục ~/LAB10/fake_site, chép fake_index.html thành index.html và mở HTTP server tại cổng 80 bằng sudo python3 -m http.server 80 --directory ~/LAB10/fake_site.
Terminal 3: Chạy script tấn công python3 kaminsky_remote_lab10.py để gửi truy vấn tên miền con ngẫu nhiên dưới example.lab10.test và bắn các batch dự đoán TXID qua cổng UDP 5300 của race relay.
Quan sát log: Trên AUTH-DNS xuất hiện [PENDING] và [HIT], trên DNS-SERVER xuất hiện [CACHE] delegation poisoned, và script trên Kali xác nhận www.example.lab10.test A = 10.10.0.200.
**Bước 5: Kiểm chứng trên VICTIM và Phân tích kết quả (Task 2)**
Trên VICTIM: Chạy lại sudo resolvectl flush-caches, resolvectl query www.example.lab10.test và curl [http://www.example.lab10.test](http://www.example.lab10.test). Xác nhận IP trả về đã bị đổi thành 10.10.0.200 và nội dung trang web hiển thị giao diện giả mạo (LAB10 Fake Site).
Thu thập minh chứng bắt gói (Wireshark/tcpdump) thể hiện truy vấn upstream từ 10.9.0.10:33333 -> 172.16.6.20:53, gói phản hồi giả mạo chứa bản ghi Authority NS = ns.evil.example.lab10.test kèm Glue A = 10.10.0.200 (nằm trong cùng phạm vi bailiwick của zone example.lab10.test), và các truy vấn sau đó của resolver đi thẳng tới 10.10.0.200:53.
**Bước 6: Khôi phục BIND9 hiện đại và Hardening hệ thống**
Trên DNS-SERVER: Dừng resolver mô phỏng (sudo pkill -f vuln_resolver_lab10.py) và bật lại dịch vụ BIND9 (sudo systemctl start bind9).
Kiểm tra cơ chế bảo vệ của BIND9:Xác nhận ACL đệ quy chỉ giới hạn trong dải 10.9.0.0/24 (Kali ở mạng 10.10.0.0/24 không thể lạm dụng làm open resolver).
Chạy sudo rndc flush và bắt gói bằng sudo tcpdump -ni any host 172.16.6.20 and udp port 53 khi tạo nhiều truy vấn từ VICTIM để chứng minh BIND 9.18 sử dụng cổng nguồn UDP thay đổi ngẫu nhiên thay vì cố định ở cổng 33333.
Dọn dẹp sau bài lab: Tắt hoặc xóa rule UDP 5300 trên pfSense, vô hiệu hóa interface ATTACKNET nếu không dùng cho lab tiếp theo, và hoàn thành bảng so sánh cùng 12 câu hỏi phân tích trong báo cáo.   
**3. Lỗi gặp phải**
