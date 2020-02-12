## Cấu hình gửi cảnh báo qua email

### Cấu hình mail relay sử dụng Gmail

Chúng ta thực hiện các bước sau để có thể sử dụng Gmail để cho Check_mk có thể gửi cảnh báo khi có sự cố xảy ra với host/service.

Trên OMD server, chúng ta thực hiện các bước sau để cấu hình Postfix thành một Mail Relay sử dụng Gmail.

- Bước 1: Cài đặt `postfix` và gói `mailutil`:

`yum install -y postfix mailx cyrus-sasl cyrus-sasl-plain`

- Bước 2: Cấu hình xác thực cho Gmail:

Các file cấu hình của postfix được lưu trữ trong `/etc/postfix`, chúng ta tạo file để lưu trữ thông tin của Gmail.

sửa file `etc/postfix/sasl_passwd` và thêm nội dung sau:

`[smtp.gmail.com]:587    username@gmail.com:password`

trong đó:

	- username: Tên đăng nhập Gmail
	
	- password: Mật khẩu của Gmail

- Bước 3: Cấu hình `Postfix`:

Chỉnh sửa file cấu hình `/etc/postfix/main.cf` và thêm những dòng sau:

```
# enable SASL authentication
smtp_sasl_auth_enable = yes
# disallow methods that allow anonymous authentication.
smtp_sasl_security_options = noanonymous
# where to find sasl_passwd
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
# Enable STARTTLS encryption
smtp_use_tls = yes
# where to find CA certificates
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

- Bước 4: Xử lý thông tin xác thực Gmail:

Sử dụng `postmap` để mã hóa file xác thực, kết quả mã hóa được lưu tại `sasl_passwd.db`

`postmap /etc/postfix/sasl_passwd`

- Bước 5: Bảo mật cho Password và Hash Database Files:

```
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

- Bước 6: Khởi động lại `postfix`:

`systemctl restart postfix`

- Bước 7: Cài đặt Gmail, cho phép truy cập bằng ứng dụng kém an toàn:

Đăng nhập vào nick gmail đã nhập vào ở bước 2, truy cập vào link https://myaccount.google.com/lesssecureapps?pli=1 để cài đặt tính năng này.

- Bước 8: Kiểm tra Mail relay:

Chúng ta kiểm tra hoạt động của nó bằng cách gửi đi một email:

`echo "Test email." | mail -s "Test postfix relay." nobody@example.com`

Thay địa chỉ email của bạn vào câu lệnh.

Vào hộp thư của bạn kiểm tra email.

<img src="img/61.png">

### Cấu hình gửi email trong Check_mk

- Thêm group nhận mail cảnh báo:

Bước 1: Trên tab `WATO - CONFIGURATION`, chọn `Contact Groups` -> `New contact group`:

<img src="img/62.png">

Bước 2: Điền thông tin cho Group và lưu lại

<img src="img/63.png">

- Thêm user nhận mail cảnh báo:

Bước 1: Trên tab `WATO - CONFIGURATION`, chọn `Users` -> `New User`:

<img src="img/64.png">

Bước 2: Điền thông tin cho User và lưu lại:

<img src="img/65.png">

xem lại thông tin user vừa add:

<img src="img/66.png">

Bước 3: Active những thay đổi:

<img src="img/67.png">

<img src="img/68.png">

- Gửi mail cảnh báo cho host:

Bước 1: Thêm Rule gửi email cho Host

truy cập `WATO - CONFIGURATION` -> `Notifications` -> `New Rule`

<img src="img/69.png">

Bước 2: Điền thông tin cho rule

Trong ví dụ này, tôi sẽ thêm **Rule** với **Description là** `Send mail for host when status change` (tùy chọn) và **Group** nhận bản tin cảnh báo là `group1` cho host `cent7`, các tùy chọn như sau:

<img src="img/70.png">

Bước 3: Lưu lại các thay đổi:

<img src="img/71.png">

<img src="img/72.png">

Bước 4: Chỉnh thời gian lặp lại thông báo nếu sự cố chưa được khắc phục:

Tại tab `WIEWS`, chọn `Hosts` -> `All Hosts` và bấm vào biểu tượng 3 que bên cạnh tên host, chọn `Parameters for this host`:

<img src="img/73.png">

Kéo xuống bên dưới, tìm đến rule `Periodic notifications during host problems`:

<img src="img/74.png">

thêm 1 rule mới cho host:

<img src="img/75.png">

Điền thời gian gửi lại thông báo (Đơn vị: Phút; Mặc định: 120 phút), chọn host muốn áp dụng Rule (Nếu không xác định host cụ thể, mặc định Check_mk sẽ áp dụng cho tất cả các host) và save lại.

<img src="img/76.png">

Lưu lại các thay đổi:

<img src="img/77.png">

<img src="img/78.png">

- Gửi cảnh báo cho Service:

Bước 1: Thêm Rule gửi email cho Service:

tại tab `WATO - CONFIGURATION` chọn `Notifications` -> `New Rule`

<img src="img/79.png">

Bước 2: Điền thông tin cho rule:

thêm **Rule** với **Description** là `Send mail for services when status change` (tùy chọn) và **User** nhận cảnh báo là `user1`, để gửi cảnh báo cho service `SSH` và save lại

<img src="img/80.png">

Bước 3: Lưu lại các thay đổi

<img src="img/81.png">

<img src="img/82.png">

- Cấu hình cảnh báo leo thang:

Bước 1: Tạo các group nhận cảnh báo:

tại tab `WATO - CONFIGURATION`, chọn `Contact Groups` -> `New contact group`, tạo 1 group mới tên là `group2`:

<img src="img/83.png">

Bước 2: Add user vào group vừa tạo

tại tab `WATO - CONFIGURATION`, chọn `Users` -> `New user`, tạo 1 user mới là `user2` và thêm vào group mới tạo:

<img src="img/84.png">

Bước 3: Tạo rule nhận email cho các group

Tìm đến tab `WATO - CONFIGURATION`, chọn `Notifications` -> `New Rule`:

<img src="img/85.png">

Điền thông tin cho rule thứ nhất ở mức cảnh báo level 1:

<img src="img/86.png">

Tạo tiếp rule cho mức cảnh báo level 2:

<img src="img/87.png">

Bước 4: Lưu lại các thay đổi:

<img src="img/88.png">

<img src="img/89.png">

- Cảnh báo âm thanh:

Tính năng này sẽ phát âm thanh trên Web UI mỗi khi host/service thay đổi trạng thái. Chúng ta cấu hình nó như sau:

Vào `WATO - CONFIGURATION`, chọn `Global Settings` -> `User Interface` và bật tính năng `Enable sounds in views` chuyển từ `OFF` thành `ON` như hình:

<img src="img/90.png">

Lưu lại các thay đổi:

<img src="img/91.png">

<img src="img/92.png">