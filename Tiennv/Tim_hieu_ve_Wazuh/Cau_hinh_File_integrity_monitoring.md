## Cấu hình File integrity monitoring

1. Basic usage

`Syscheck` được cấu hình trong tập tin `ossec.conf`. Nói chung, cấu hình này được đặt bằng các phần sau:

- frequency: Tần suất mà `syscheck` sẽ được chạy (tính bằng giây).

| Giá trị mặc định | 43200 |
| --- | --- |
| Giá trị được phép | 1 số dương, tính bằng giây |

- directories: Sử dụng tùy chọn này để thêm hoặc xóa các thư mục sẽ được theo dõi. Các thư mục phải được phân tách bằng dấu phẩy.

Tất cả các tệp và thư mục con trong các thư mục lưu ý cũng sẽ được theo dõi.

Ổ đĩa không có thư mục không hợp lệ. Tối thiểu là '.', ví dụ ( `D:\.`).

Điều này sẽ được thiết lập trên hệ thống sẽ được giám sát (hoặc trong `agent.conf`, nếu thích hợp).

| Giá trị mặc định | /etc,/usr/bin,/usr/sbin,/bin,/sbin |
| --- | --- |
| Giá trị được phép | Bất kỳ thư mục nào |

- ignore: Danh sách các tập tin hoặc thư mục sẽ bị bỏ qua (một mục nhập trên mỗi dòng). Nhiều dòng có thể được nhập để bao gồm nhiều tệp hoặc thư mục. Các tệp và thư mục này vẫn được kiểm tra, nhưng kết quả bị bỏ qua.

| Giá trị được phép | Bất kỳ thư mục hoặc tệp tin |
| --- | --- |
| Ví dụ | /etc/mtab |

- alert_new_files: Chỉ định nếu `syscheck` sẽ cảnh báo khi các tệp mới được tạo.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

Để cấu hình `syscheck`, một danh sách các tập tin và thư mục phải được xác định. Các tùy chọn `check_all` checks file size, permissions, owner, last modification date, inode và hash sums (MD5, SHA1, SHA256).

2. Cấu hình quét theo lịch trình

`Syscheck` có một tùy chọn `frequency` để cấu hình quét hệ thống. Trong ví dụ này, `syscheck` được cấu hình để chạy cứ sau 10 giờ.

```
<syscheck>
  <frequency>36000</frequency>
  <directories>/etc,/usr/bin,/usr/sbin</directories>
  <directories>/bin,/sbin</directories>
</syscheck>
```

3. Cấu hình giám sát thời gian thực

Giám sát thời gian thực được cấu hình với tùy chọn `realtime`. Tùy chọn này chỉ hoạt động với các thư mục chứ không phải với các tệp riêng lẻ. Phát hiện thay đổi thời gian thực bị tạm dừng trong quá trình quét `syscheck` định kỳ và kích hoạt lại ngay khi những lần quét này hoàn tất.

```
<syscheck>
  <directories check_all="yes" realtime="yes">c:/tmp</directories>
</syscheck>
```

4. Cấu hình giám sát who-data

> Được bổ sung ở phiên bản 3.4.0

Giám sát who-data được cấu hình với tùy chọn `whodata`. Tùy chọn này được thay thế cho tùy chọn `realtime`, có nghĩa là `whodata` ngụ ý theo dõi thời gian thực nhưng thêm thông tin who-data. Chức năng này sử dụng hệ thống con Linux Audit và Microsoft Windows SACL, do đó có thể cần cấu hình bổ sung.

```
<syscheck>
  <directories check_all="yes" whodata="yes">/etc</directories>
</syscheck>
```

5. Cấu hình để bỏ qua các tập tin

Các tệp và thư mục có thể được bỏ qua bằng cách sử dụng tùy chọn ignore (hoặc registry_ignore cho các mục Windows registry). Để tránh các kết quả báo động giả, `syscheck` có thể được cấu hình để bỏ qua các tệp nhất định không cần phải theo dõi.

```
<syscheck>
  <ignore>/etc/random-seed</ignore>
  <ignore>/root/dir</ignore>
  <ignore type="sregex">.log$|.tmp</ignore>
</syscheck>
```

6. Bỏ qua các tập tin thông qua các quy tắc

Cũng có thể bỏ qua các tệp bằng cách sử dụng các quy tắc, như trong ví dụ này:

```
<rule id="100345" level="0">
  <if_group>syscheck</if_group>
  <match>/var/www/htdocs</match>
  <description>Ignore changes to /var/www/htdocs</description>
</rule>
```

7. Thay đổi mức độ nghiêm trọng

Với quy tắc tùy chỉnh, mức độ cảnh báo `syscheck` có thể được thay đổi khi phát hiện thay đổi đối với một tệp hoặc mẫu tệp cụ thể.

```
<rule id="100345" level="12">
  <if_group>syscheck</if_group>
  <match>/var/www/htdocs</match>
  <description>Changes to /var/www/htdocs - Critical file!</description>
</rule>
```