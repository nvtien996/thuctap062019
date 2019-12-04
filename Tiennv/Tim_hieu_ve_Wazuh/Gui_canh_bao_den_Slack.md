## Gửi cảnh báo đến Slack

Việc tích hợp này cho phép nhận thông báo vào kênh Slack nhờ vào Incoming Webhooks, một cách đơn giản để gửi tin nhắn từ các ứng dụng của bên thứ 3 (trong trường hợp này là Wazuh).

Đầu tiên vào link [này](https://slack.com/create) để đăng ký

Sau đó, ta cần xác nhận email.

Tiếp theo, làm theo hướng dẫn.

Tiếp đó, vào link [này](https://api.slack.com/apps) để tạo 1 Slack app.

Sau khi đã tạo app, bấm vào mục Incoming Webhooks và bật nó lên.

Sau đó, kéo xuống cuối trang web và bấm vào mục Add New Webhook to Workspace.

Tiếp theo bấm vào ô Copy tại mục Webhook URL để copy url.

Sau khi đã có được Slack url, ta cần cấu hình trong tệp `ossec.conf`:

```
<integration>
  <name>slack</name>
  <hook_url>https://hooks.slack.com/services/...</hook_url> <!-- Replace with your Slack hook URL -->
  <alert_format>json</alert_format>
</integration>
```

Cuối cùng khởi động lại Wazuh-manager

Đối với Systemd:

`systemctl restart wazuh-manager`

Đối với SysV Init:

`service wazuh-manager restart`