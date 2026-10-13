# BÁO CÁO THỰC HÀNH LAB 6: XÁC THỰC ACTIVE DIRECTORY VÀ CHÍNH SÁCH TRUY CẬP TRÊN pfSense

**Sinh viên thực hiện:** Lê Thị Mỹ Hằng  
**Lớp:** 11_TMĐT  

---

## 1. Bài thực hành
* **Về Topology:** Cần dựng một môi trường hoàn toàn độc lập với LAB 5, bao gồm 4 interface (WAN, MGMT, LAN, DMZ). Việc tách riêng cổng MGMT giúp quản trị pfSense an toàn và không bị kẹt khi test các rule trên LAN/DMZ.
* **Về Tích hợp AD & Captive Portal:** pfSense sẽ liên kết với Domain Controller qua LDAP. Thay vì dùng quyền Domain Admin (rất rủi ro), mình sẽ dùng một service account riêng (`svc_pfsense_ldap`). Sau đó, bật Captive Portal trên LAN để ép user (`sv_a`, `sv_b`) đăng nhập.
* **Về Bản chất Firewall pfSense:** Mình nhận ra rào cản lớn nhất là pfSense **không thể** dùng trực tiếp tên user (LDAP username) làm điều kiện lọc (Source) trong Firewall Rule. Do đó, giải pháp là gán IP tĩnh cho Client-A và Client-B, sau đó sử dụng **Alias** để gom nhóm IP người dùng và IP website.
* **Về Logic xử lý Rule:** pfSense đọc rule theo nguyên tắc **First-Match** (từ trên xuống dưới). Phải luôn đặt các rule "Block" chi tiết ở trên và rule "Pass" rộng ở dưới.
* **Về Kiểm thử:** Hiểu được tầm quan trọng của việc phải luôn **Reset States** (`Diagnostics > States`) sau mỗi lần sửa đổi rule, nếu không kết quả test sẽ bị sai lệch do lưu vết kết nối cũ.

## 2. Tiến độ thực hiện (Các bước đã làm)
- [x] Đọc hiểu mô hình và phân tích yêu cầu đề bài.
- [x] Lên danh sách các thành phần cần tạo: pfSense-LAB6, DC01, Client-A, Client-B, DMZ-WEB.
- [ ] Khởi tạo và gán IP cho các máy ảo trên VirtualBox.
- [ ] Cấu hình Active Directory (DC01) và tạo user/OU.
- [ ] Kết nối pfSense với LDAP và thiết lập Captive Portal.
- [ ] Tạo Alias và cấu hình 7 kịch bản Firewall Rule.
- [ ] Kiểm thử, chụp ảnh minh chứng và trả lời câu hỏi lý thuyết.

## 3. Nhật ký lỗi & Cách khắc phục (Troubleshooting)
*(Phần này dùng để ghi chú lại những lỗi thực tế gặp phải trong lúc làm và cách mình đã tự sửa, hiện tại chưa cấu hình nên chưa ghi nhận lỗi)*

* **Lỗi 1:** [Đang cập nhật...]
  * **Nguyên nhân:** 
  * **Cách khắc phục:** 

* **Lỗi 2:** [Đang cập nhật...]
  * **Nguyên nhân:** 
  * **Cách khắc phục:** 

---
*Ghi chú thêm: Cuối bài nhớ xuất file `config.xml` để nộp kèm cùng file Word báo cáo.*
