## Giám sát Windows service

### Các bước chuẩn bị

Để giám sát được các service chạy trong hđh Windows, trước tiên ta cần 1 máy tính (hoặc máy ảo) đã được cài đặt Windows lên đó.

Các bạn có thể tham khảo bài viết [sau](https://getlabsdone.com/10-easy-steps-to-install-windows-10-on-linux-kvm/) để biết cách cài đặt Windows lên KVM.

### Cài đặt agent

Sau khi đã có máy tình cài Windows, bước tiếp theo ta cần làm là cài đặt agent Check_mk lên máy tính Windows.

Bước 1: Tải xuống gói cài đặt agent

Copy đường link của file cài đặt qua máy Windows, thay vì là file `.exe` như thường lệ, file cài đặt sẽ có đuôi `.msi`

File cài đặt agent có thể được tìm thấy tại `WATO - CONFIGURATION` -> `Monitoring Agents`

<img src="img/153.png">

Link file cài đặt sẽ có dạng `http://ip_check_mk/tên_site/check_mk/agents/windows/check_mk_agent.msi`

Bước 2: Sau khi tải về, tiến hành cài đặt bình thường

<img src="img/154.png">

Bước 3: Kiểm tra Service đã hoạt động chưa

gõ `services.msc` vào ô tìm kiếm trong Windows, kiểm tra Check_mk service

<img src="img/155.png">

Bước 4: Nếu có sử dụng firewall, cần phải tạo rule cho firewall với port 6556

Nếu trong môi trường lab, các bạn cứ tắt firewall đi cho nhanh.

Vào `Windows Defender Firewall with Advanced Security`, chọn `Inbound Rules` -> `New Rule...`

<img src="img/156.png">

điền các thông số như sau

<img src="img/157.png">

<img src="img/158.png">

<img src="img/159.png">

<img src="img/160.png">

<img src="img/161.png">

Làm tương tự với Outbound Rule

### Add host trên Dashboard

Trên Dashboard, truy cập vào `WATO - CONFIGURATION` -> `Hosts` -> `New host`

<img src="img/162.png">

điền các thông tin cho host

<img src="img/163.png">

Lưu lại các thay đổi

<img src="img/164.png">

<img src="img/165.png">

kết quả:

<img src="img/166.png">

### Monitor windows services

Cách giám sát các services cụ thể trên Windows

Trên Dashboard, đi đến mục `WATO - CONFIGURATION` chọn `Host & Service Parameters` -> `Parameters for discovered services`

<img src="img/167.png">

Tiếp theo trong mục `DISCOVERY - AUTOMATIC SERVICE DETECTION` chọn `Windows Service Discovery`

<img src="img/168.png">

Chọn `Create rule in folder`

<img src="img/169.png">

Điền các thông tin

<img src="img/170.png">

Trên `WATO - CONFIGURATION` chọn `Hosts` -> `Discovery`

<img src="img/171.png">

Bấm `Start` rồi đợi 1 lúc

<img src="img/172.png">

Lưu lại các thay đổi

<img src="img/173.png">

<img src="img/174.png">

Kiểm tra trên Dashboard

<img src="img/175.png">