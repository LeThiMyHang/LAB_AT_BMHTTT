# BÁO CÁO LAB 4: KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## THÔNG TIN SINH VIÊN
- **Họ và tên:** Lê Thị Mỹ Hằng
- **MSSV:** 1150070011
- **Lớp:** 11_TMĐT

## 1. PHIÊN BẢN MÔI TRƯỜNG
- **Máy thật (Host):** Windows 10 x64.
- **Phần mềm ảo hóa:** VMware Workstation.
- **Máy quét (VM 1):** Kali Linux.
- **Máy đích (VM 2):** Windows 10 x64.

## 2. CÁCH DỰNG MÔI TRƯỜNG
- Tạo mạng nội bộ thông qua Virtual Network Editor (VMnet1 - Host-only) với dải IP `192.168.56.0/24`.
- Cấu hình Network Adapter của cả hai máy ảo Kali Linux và Windows 10 sang chế độ **Custom: VMnet1 (Host-only)**. Tuyệt đối không dùng NAT hay Bridged để đảm bảo cách ly an toàn.
- Tạm thời tắt Windows Defender Firewall trên máy đích để kiểm tra kết nối ping ban đầu.

## 3. CÁC TÌNH HUỐNG ĐÃ THỰC HIỆN
- [x] **Host Discovery:** Dùng lệnh `-sn` phát hiện các host đang hoạt động (nhận diện MAC Address thuộc VMware).
- [x] **Quét cổng TCP:** Thực hiện quét cổng tàng hình qua kỹ thuật TCP SYN Scan (`-sS`).
- [x] **Nhận diện dịch vụ và OS:** Dùng lệnh `-sV` và `-A` để xác định chi tiết phần mềm, phiên bản dịch vụ và hệ điều hành.
- [x] **Quét bằng NSE:** Khai thác thông tin từ dịch vụ SMB ở cổng 445 bằng script `smb-os-discovery`.
- [x] **Xuất báo cáo (Log):** Xuất log dưới dạng văn bản thường qua cờ `-oN` để làm hồ sơ bằng chứng.
- [x] **Tình huống Hardening:** Thực hiện quét đối chiếu trước và sau khi kích hoạt lại Windows Defender Firewall. Kết quả xác nhận 1000 cổng quét mặc định đã chuyển sang trạng thái `filtered` do `no-response`.

## 4. KẾT QUẢ
- **Đánh giá:** PASS (Đã thực hiện đủ các lệnh yêu cầu, môi trường cô lập an toàn, log output khớp thời gian thực).

## 5. LỖI GẶP PHẢI VÀ CÁCH KHẮC PHỤC
- **Lỗi 1:** Bị từ chối quyền (Permission denied) khi quét SYN scan.
  - **Khắc phục:** Cấp quyền quản trị bằng cách thêm `sudo` vào trước lệnh Nmap.
- **Lỗi 2:** Quét ra 0 hosts hoặc báo lỗi `Unable to split netmask from target expression`.
  - **Khắc phục:** Lỗi cú pháp do gõ dư dấu gạch chéo `/` ở cuối địa chỉ IP (VD: `192.168.56.129/`). Đã xóa ký tự thừa và thực thi quét thành công.
- **Lỗi 3:** Ping từ máy quét sang máy đích bị request timeout.
  - **Khắc phục:** Tường lửa của máy đích đang chặn gói tin ICMP. Vào Windows Defender Firewall tắt cấu hình chặn mạng Private/Public để thông mạng thực hành.oX` để xuất ra file XML thành công trước, sau đó mới có dữ liệu đầu vào cho lệnh `xsltproc`[cite: 9].
