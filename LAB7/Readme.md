# BÁO CÁO LAB 7: XÂY DỰNG CHÍNH SÁCH WEB PROTECTION TRÊN MÔ HÌNH pfSense + WEB PROXY

* **Lớp:** 11_TMĐT
* **MSSV:**1150070011
* **Họ và tên:** Lê Thị Mỹ Hằng
---
## 1. Mục tiêu bài học & Tổng quan
* **Kế thừa hệ thống:** Bài thực hành tiếp nối toàn bộ hạ tầng mạng từ Lab 6 gồm pfSense làm Gateway/Firewall (`10.6.0.1`), máy chủ Domain Controller (`DC01` - `10.6.0.10`), các máy trạm (`Client-A`, `Client-B`) và máy chủ vùng DMZ (`DMZ-WEB` - `172.16.6.10`)[cite: 1, 2].
* **Chuyển đổi kiến trúc Web Protection:** Do các gói phần mềm Squid/SquidGuard trên pfSense đã bị Netgate đánh dấu là *deprecated* (ngừng hỗ trợ do lỗ hổng bảo mật upstream), bài lab triển khai tầng lọc nội dung L7 trên một máy chủ **WEB-PROXY độc lập chạy Ubuntu Server 24.04 LTS (`10.6.0.30`)** sử dụng **E2Guardian**, **Squid** và **ClamAV**[cite: 1, 2, 6].
* **Phân định ranh giới chức năng:** 
  * **pfSense:** Kiểm soát tầng mạng (L3/L4), chặn kết nối trực tiếp ra ngoài để chống bypass, ép buộc toàn bộ lưu lượng web đi qua proxy.
  * **WEB-PROXY:** Đảm nhận việc lọc URL, phân loại nội dung (Hard Block, Warning, Exception, Schedule) và quét virus file tải về.
---

## 2. Các bước triển khai chính trong Lab
1. **Chuẩn bị và cấu hình máy ảo WEB-PROXY (`10.6.0.30`):**
   * Cài đặt Ubuntu Server 24.04 LTS, cấu hình mạng nội bộ thông với LAN.
   * Cài đặt gói `squid`, `e2guardian` và `clamav-daemon`.
   * Cấu hình Squid lắng nghe trên `127.0.0.1:3128` (chỉ cho phép localhost/E2Guardian truy cập) và E2Guardian lắng nghe ở cổng `10.6.0.30:8080`.
2. **Thiết lập website mô phỏng (`DMZ-WEB` & `DC01`):**
   * Sử dụng script khởi tạo Nginx trên `DMZ-WEB` (`172.16.6.10`) để dựng 13 virtual host đại diện cho các nhóm nội dung (`.lab7.test`) kèm theo các file test dung lượng và file mẫu EICA.
   * Tạo phân vùng phân giải tên miền `lab7.test` trên DNS Server của `DC01` trỏ về IP của DMZ-WEB.
3. **Cấu hình pfSense ép luồng và chống Bypass:**
   * Tạo các Alias quản lý (`LAB7_CLIENTS`, `LAB7_PROXY`, `WEB_PORTS`).
   * Thiết lập Firewall Rules trên giao diện LAN: Cho phép DNS, cho phép client gọi tới proxy (`TCP 8080`), đồng thời **chặn cứng** kết nối trực tiếp `TCP 80/443` ra ngoài để ngăn chặn việc người dùng tắt proxy trên trình duyệt.
4. **Xây dựng chính sách lọc nội dung (E2Guardian Policy):**
   * **Hard Block:** Chặn hoàn toàn các nhóm website Geo-Sim (China, Middle East), Job Search và Game độc hại.
   * **Warning / Temporary Bypass:** Cảnh báo các nhóm như Weapons, Politics, Religion, Streaming kèm tính năng mở khóa tạm thời trong 5 phút (`bypass = 300`).
   * **Exception / Whitelist:** Cho phép hoàn toàn các trang Education và Normal.
   * **Antivirus Scanning:** Kích hoạt ClamAV content scanner để chặn file mã độc/mẫu kiểm thử (EICAR) khi tải từ trang download.
---

## 3. Nhật ký lỗi & Sự cố phát sinh (Troubleshooting Log)
  * **Hiện tượng:** 
  * **Nguyên nhân:** 
  * **Cách khắc phục:**
