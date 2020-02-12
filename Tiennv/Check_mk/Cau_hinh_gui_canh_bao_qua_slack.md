## Cấu hình gửi cảnh báo qua Slack

### Chuẩn bị Slack

Đăng nhập vào Slack của bạn và bấm vào (+) để tạo một channel mới dùng để nhận thông báo từ Check_mk.

<img src="img/105.png">

Sau khi đã tạo channel, truy cập vào link [sau](https://api.slack.com/apps), chọn `Create an App` để tạo app và lấy thông tin Incomming WebHooks

<img src="img/106.png">

Đặt tên cho app, chọn workspace rồi bấm `Create app`

<img src="img/107.png">

Sau khi tạo xong app, chọn `Incoming Webhooks`

<img src="img/108.png">

Activate Incoming Webhooks và Add New Webhook to Workspace:

<img src="img/109.png">

Chọn channel đã tạo ở trên rồi bấm `Allow`:

<img src="img/110.png">

Copy Webhook URL:

<img src="img/111.png">
