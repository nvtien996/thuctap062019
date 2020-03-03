## Check và cảnh báo khi có dòng log mới được thêm vào

- Tải xuống script trên máy agent:

`cd /usr/lib/check_mk_agent/local/ && wget https://github.com/nvtien996/thuctap062019/blob/master/Tiennv/Check_mk/plugin-script/check_log_line`

- Phân quyền cho script:

`chmod +x check_log_line`

- Trên server Checkmk chạy các lệnh sau:

```
su tên_site
cmk -I hostname
cmk -O
```

> Lưu ý: `hostname` ở đây là của host được giám sát chứ không phải hostname của Checkmk server

- Sau đó kiểm tra kết quả trên Web UI