# BÁO CÁO LAB 4: KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## THÔNG TIN SINH VIÊN
- **Họ và tên:** Lê Thị Mỹ Hằng
- **MSSV:** 1150070011
- **Lớp:** 11_TMĐT

## 1. PHIÊN BẢN MÔI TRƯỜNG
- **Máy thật (Host):** Windows 10
- **Phần mềm ảo hóa:** VirtualBox
- **Máy quét (VM 1):** Kali Linux
- **Máy đích (VM 2):** Metasploitable 2

## 2. CÁCH DỰNG MÔI TRƯỜNG
- Tạo mạng nội bộ (Host-Only Network) trên VirtualBox với dải IP `192.168.56.0/24`[cite: 9].
- Cấu hình Network Adapter của cả Kali Linux và Metasploitable 2 sang chế độ **Host-Only Adapter**[cite: 9].
- Tuyệt đối không dùng chế độ NAT hay Bridged trong quá trình quét để đảm bảo mạng khép kín, không gây ảnh hưởng ra bên ngoài[cite: 9].

## 3. CÁC TÌNH HUỐNG ĐÃ THỰC HIỆN
- [x] **Host Discovery:** Dùng lệnh `-sn` để quét và phát hiện các host đang hoạt động trong mạng[cite: 9].
- [x] **Quét cổng TCP:** Thực hiện khảo sát qua các kỹ thuật TCP Connect (`-sT`), SYN Scan (`-sS`), FIN/Xmas/NULL (`-sF`, `-sX`, `-sN`), và ACK Scan (`-sA`)[cite: 9].
- [x] **Quét cổng UDP:** Quét 20 cổng UDP phổ biến nhất bằng lệnh `-sU --top-ports 20`[cite: 9].
- [x] **Nhận diện dịch vụ và OS:** Chạy lệnh nhận diện phiên bản dịch vụ (`-sV`), hệ điều hành (`-O`) và quét tổng hợp Aggressive (`-A`)[cite: 9].
- [x] **Quét bằng NSE:** Khai thác thông tin SMB qua script `smb-os-discovery` và kiểm tra lỗ hổng MS17-010 bằng `smb-vuln-ms17-010`[cite: 9].
- [x] **Xuất báo cáo (Log):** Xuất log dưới dạng văn bản thường (`-oN`), XML (`-oX`), Grepable (`-oG`) và dùng công cụ `xsltproc` để chuyển đổi XML sang HTML[cite: 9].
- [x] **Tình huống Hardening:** Thực hiện quét đối chiếu (Before/After) trước và sau khi thay đổi cấu hình bảo mật (tắt dịch vụ/bật firewall)[cite: 9].

## 4. KẾT QUẢ
- **Đánh giá:** PASS (Đã thực hiện đủ các lệnh, log output khớp timestamp và đã làm sạch file trước khi tải lên).

## 5. LỖI GẶP PHẢI VÀ CÁCH KHẮC PHỤC

  - **Cách khắc phục:** Vào Settings của VirtualBox kiểm tra, đảm bảo cả 2 máy ảo đều dùng chung adapter `192.168.56.x` và tắt NAT[cite: 9].
- **Lỗi 3:** Báo lỗi "No such file" khi xuất file HTML bằng `xsltproc`[cite: 9].
  - **Cách khắc phục:** Cần phải chạy lệnh Nmap có cờ `-oX` để xuất ra file XML thành công trước, sau đó mới có dữ liệu đầu vào cho lệnh `xsltproc`[cite: 9].
