## Cấu hình ngưỡng cảnh báo trong Check_mk

Theo mặc định thì Check_mk đã có những ngưỡng cảnh báo cho từng dịch vụ, tuy nhiên những thông số đó chưa hẳn đã ổn đối với hệ thống của chúng ta vì thế chúng ta cần phải cấu hình lại ngưỡng cảnh báo của dịch vụ.

Thực hiện như sau:

- Trên Web UI, chúng ta tìm đến `VIEWS`, mở tab `Services` và chọn `All services`

<img src="img/54.png">

- Trong ví dụ này, tôi sẽ đặt ngưỡng cảnh báo khi thư mục `/` được sử dụng 77% dung lượng thì cảnh báo `Warning` và 88% dung lượng là `Critical`. Bấm vào biểu tượng 3 que, chọn mục `Parameters for this service`

<img src="img/55.png">

- Chọn `Filesystems (used space and growth)`:

<img src="img/56.png">

- Chọn `Create rule in folder` để tạo rule mới:

<img src="img/57.png">

- Điền các thông tin và lưu lại:

<img src="img/58.png">

- Kích hoạt các thay đổi:

<img src="img/59.png">

<img src="img/60.png">