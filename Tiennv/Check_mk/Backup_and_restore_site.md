## Backup và restore site

Để bảo vệ dữ liệu giám sát của bạn trong trường hợp xảy ra lỗi phần cứng hoặc các lỗi tương tự, ta có thể sao lưu lại dữ liệu giám sát trên site. Bản sao lưu dữ liệu có thể được định cấu hình qua giao diện người dùng web của thiết bị.

Bản sao lưu đầy đủ bao gồm tất cả các cấu hình được xác định trên hệ thống, các tệp được cài đặt và tương tự như các phiên bản giám sát của bạn.

### Backup dữ liệu cho site

Chúng ta backup lại thông tin theo các bước sau:

Đầu tiên trên Web UI, chúng ta tìm đến `WATO - CONFIGURATION` -> `Backup` -> `Backup targets` để tạo 1 nơi chứa các file backup

<img src="img/176.png">

chọn `New backup target`

<img src="img/177.png">

Điền các thông tin và lưu lại, folder dùng để backup phải là folder mà group `omd` có quyền ghi

<img src="img/178.png">

Sau đó back lại và chọn `New job`

<img src="img/179.png">

<img src="img/180.png">

Điền các thông tin cho việc backup và lưu lại

<img src="img/181.png">

Bấm vào nút như trong hình để bắt đầu công việc backup

<img src="img/182.png">

Backup đang được diễn ra

<img src="img/183.png">

Sau khi backup xong

<img src="img/184.png">

### Restore dữ liệu đã backup

Việc này được thực hiện khi muốn restore lại dữ liệu trên cùng 1 site hoặc chuyển dữ liệu giữa các site với nhau

Các bước thực hiện như sau

Trên mục `WATO - CONFIGURATION`, chọn `Backup` -> `Restore`

<img src="img/185.png">

Chọn backup target muốn restore

<img src="img/186.png">

Chọn bản backup và restore dữ liệu

<img src="img/187.png">

Xác nhận việc restore

<img src="img/188.png">

Việc restore đang được diễn ra

<img src="img/189.png">

Hoàn tất restore

<img src="img/190.png">

### Backup vầ restore bằng câu lệnh

Các bước tiến hành

#### Backup site

Bước 1: Liệt kê các site đang có trên hệ thống:

`omd sites`

<img src="img/191.png">

Bước 2: Dừng site mà bạn muốn backup

`omd stop <tên_site>`

<img src="img/192.png">

Bước 3: Backup dữ liệu của site

`omd backup <tên_site> /opt/<tên_site>-bk.tar.gz`

Chú ý:

Tùy vào số lượng host, thời gian backup sẽ tăng theo

`<tên_site>` là tên site

`/opt/<tên_site>-bk.tar.gz`: nơi lưu trữ file, với dạng nén tar.gz

#### Restore site

Bước 1: Restore lại site

`omd restore <tên_site> /opt/<tên_site>-bk.tar.gz`

Bước 2: Xem lại status của các site

`omd status`

<img src="img/193.png">

Bước 3: Khởi động site vừa restore

`omd start <tên_site>`

<img src="img/194.png">