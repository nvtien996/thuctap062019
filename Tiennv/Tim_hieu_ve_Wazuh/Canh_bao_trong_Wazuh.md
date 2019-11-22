## Cấu hình các cảnh báo trong Wazuh

### Định nghĩa ngưỡng cảnh báo

Mỗi event trên Wazuh Agent được đặt security level mặc định là 1. Tất cả các event này tăng lên sẽ tạo ra cảnh báo trên Wazuh Manager.

- Cấu hình

Ngưỡng cảnh báo được cấu hình trên file `ossec.conf` sử dụng XML tag. Chi tiết về các option có thể thiết lập xem tại [đây](https://documentation.wazuh.com/3.x/user-manual/reference/ossec-conf/alerts.html#reference-ossec-alerts)

Cấu hình mặc định sẽ như sau:

```
<alerts>
  <log_alert_level>3</log_alert_level>
  <email_alert_level>12</email_alert_level>
</alerts>
```

`log_alert_level`

Đặt mức độ nghiêm trọng tối thiểu cho các cảnh báo sẽ được lưu trữ vào `alert.log` và / hoặc `alert.json`

| Giá trị mặc định | 3 |
| --- | --- |
| Giá trị được phép | Mọi cấp độ từ 1 đến 16 |

`email_alert_level`

Đặt mức độ nghiêm trọng tối thiểu cho 1 cảnh báo để tạo thông báo email

> Cảnh báo: Đây là mức tối thiểu để cảnh báo kích hoạt email. Cài đặt này ghi đè cấu hình cảnh báo granular email. Đặt cài đặt này thành 10 sẽ ngăn việc gửi email cho các cảnh báo có mức thấp hơn 10, ngay cả khi có các cài đặt trong mức tham chiếu cấu hình granular email thấp hơn 10. Các quy tắc riêng lẻ có thể ghi đè lên điều này bằng tùy chọn `alert_by_email`, điều này buộc cảnh báo email bất kể ngưỡng cảnh báo global hoặc granular alert.

| Giá trị mặc định | 12 |
| --- | --- |
| Giá trị được phép | Mọi cấp độ từ 1 đến 16 |

`use_geoip`

> Tùy chọn này không còn dùng nữa kể từ phiên bản 2.0.

Tùy chọn này không có hiệu lực và sẽ dẫn đến lỗi phân tích cú pháp trừ khi được biên dịch với [LIBGEOIP_ENABLED](https://github.com/wazuh/wazuh/blob/master/src/config/alerts-config.c#L61).

| Giá trị mặc định | không có |
| Giá trị được phép | không có |

- Khi bất kỳ giá trị nào bị thay đổi trong file `ossec.conf`, cần phải restart lại service để có tác dụng :

`systemctl restart wazuh-manager`

### Tích hợp với API bên ngoài

`Integrator` là một daemon cho phép Wazuh kết nối tới các API bên ngoài và các tool cảnh báo như Slack và PagerDuty

Tích hợp này cho phép kiểm tra các file nguy hại sử dụng [VirusTotal database](https://documentation.wazuh.com/3.x/user-manual/capabilities/virustotal-scan/index.html)

#### Cấu hình Integrator

- `Integrator` không mở theo mặc định. Mở theo câu lệnh sau:

```
/var/ossec/bin/ossec-control enable integrator
/var/ossec/bin/ossec-control restart
```

- Các tích hợp được cấu hình trong tệp `ossec.conf` nằm trong thư mục cài đặt Wazuh (/var/ossec/etc/). Để định cấu hình tích hợp, hãy thêm cấu hình sau vào phần `<ossec_config>`:

```
<integration>
  <name> </name>
  <hook_url> </hook_url> <!-- Required for Slack -->
  <api_key> </api_key> <!-- Required for PagerDuty and VirusTotal -->

  <!-- Optional filters -->
  <rule_id> </rule_id>
  <level> </level>
  <group> </group>
  <event_location> </event_location>
</integration>
```

- Sau khi bật daemon và định cấu hình tích hợp, khởi động lại trình quản lý Wazuh để áp dụng các thay đổi:

	- Đối với Systemd:
	
	`systemctl restart wazuh-manager`
	
	- Đối với SysV Init:
	
	`service wazuh-manager restart`

- Tham chiếu cấu hình đầy đủ cho Integrator daemon:

`XML section name`:

```
<integration>
</integration>
```

Các option:

`name`: tên của service được tích hợp

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | slack, pagerduty, virustotal, bất kỳ chuỗi nào bắt đầu bằng 'custom-' |

> Chú ý: Trong trường hợp custom external integration, tên phải bắt đầu bằng `custom-` ví dụ: `custom-myintegration`.

`hook_url`: đây là URL được cung cấp bởi Slack khi mục tích hợp được bật ở phía Slack. Điều này là bắt buộc đối với Slack.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Slack URL |

`api_key`: đây là key mà bạn đã lấy từ API PagerDuty hoặc VirusTotal. Điều này là bắt buộc đối với PagerDuty và VirusTotal.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | PagerDuty/VirusTotal Api key |

Optional filters:

`level`: bộ lọc này cảnh báo theo cấp quy tắc để chỉ cảnh báo với mức được chỉ định hoặc cao hơn được đẩy.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Bất kỳ mức cảnh báo nào từ 0 đến 16 |

`rule_id`: bộ lọc này cảnh báo theo rule ID

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Các rule ID được phân tách bằng dấu phẩy |

`group`: bộ lọc này cảnh báo theo nhóm quy tắc. Đối với tích hợp VirusTotal, chỉ có các quy tắc từ nhóm `syscheck` có sẵn.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Bất kỳ nhóm quy tắc hoặc nhóm quy tắc được phân tách bằng dấu phẩy |

`event_location`: bộ lọc này cảnh báo theo nơi sự kiện bắt nguồn. Theo cú pháp [OS_Regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html).

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Bất kỳ single log file nào |

`alert_format`: điều này ghi tệp cảnh báo ở định dạng JSON. Trình tích hợp sử dụng tệp này để tìm nạp giá trị các trường

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | json |

`max_log`: độ dài tối đa của một đoạn cảnh báo sẽ được gửi đến Integrator. Chuỗi dài hơn sẽ bị cắt ngắn với `...`

| Giá trị mặc định | 165 |
| --- | --- |
| Giá trị được phép | Bất kỳ số nguyên từ 165 đến 1024 |

Ví dụ cấu hình:

```
<!-- Integration with Slack -->
<integration>
  <name>slack</name>
  <hook_url>https://hooks.slack.com/services/...</hook_url> <!-- Replace with your Slack hook URL -->
  <level>10</level>
  <group>multiple_drops|authentication_failures</group>
  <alert_format>json</alert_format>
</integration>

<!-- Integration with PagerDuty -->
<integration>
  <name>pagerduty</name>
  <api_key>API_KEY</api_key> <!-- Replace with your PagerDuty API key -->
</integration>

<!-- Integration with VirusTotal -->
<integration>
  <name>virustotal</name>
  <api_key>API_KEY</api_key> <!-- Replace with your VirusTotal API key -->
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>

<!--Custom external Integration -->
<integration>
  <name>custom-integration</name>
  <hook_url>WEBHOOK</hook_url>
  <level>10</level>
  <group>multiple_drops|authentication_failures</group>
  <api_key>APIKEY</api_key> <!-- Replace with your external service API key -->
  <alert_format>json</alert_format>
</integration>
```

### Cấu hình syslog output

- Wazuh có thể cấu hình để gửi các cảnh báo tới syslog như sau (trong file `ossec.conf`):

```
<ossec_config>
  <syslog_output>
    <level>9</level>
    <server>192.168.1.241</server>
  </syslog_output>

  <syslog_output>
    <server>192.168.1.240</server>
  </syslog_output>
</ossec_config>
```

Cấu hình trên sẽ gửi cảnh báo đến đại chỉ 192.168.1.240 và nếu mức cảnh báo cao hơn 9, các cảnh báo cũng sẽ được gửi đến 192.168.1.241.

- Để áp dụng các thay đổi, hãy khởi động lại Wazuh:

	- Đối với Systemd:
	
	`systemctl restart wazuh-manager`
	
	- Đối với SysV Init:
	
	`service wazuh-manager restart`
- Các tùy chọn có sẵn trong phần syslog output như sau:

`XML section name`:

```
<syslog_output>
</syslog_output>
```

Các tùy chọn:

`server`: IP Address hoặc hostname của syslog server.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Bất kỳ địa chỉ IP hợp lệ |

`port`: port để forward các cảnh báo đến

| Giá trị mặc định | 514 |
| --- | --- |
| Giá trị được phép | Bất kỳ port hợp lệ |

`level`: Mức tối thiểu của các cảnh báo sẽ được chuyển tiếp.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Mọi cấp độ từ 1 đến 16 |

`group`: Nhóm quy tắc của các cảnh báo sẽ được chuyển tiếp.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Bất kỳ nhóm hợp lệ. Tách nhiều nhóm bằng ký tự `|` |

> Chú ý: Quan sát rằng tất cả các nhóm phải được kết thúc bởi dấu phẩy `,`

`rule_id`: rule_id của các cảnh báo sẽ được chuyển tiếp.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Bất kỳ rule_id hợp lệ |

`location`: Trường vị trí đề cập đến nguồn gốc của cảnh báo, có thể là:

	- syscheck
	
	- rootcheck
	
	- File path
	
	- Command or its alias
	
	- command_tag (wodle)
	
	- aws-cloudtrail
	
	- cis-cat
	
	- vulnerability-detector
	
	- syscollector

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Bất kỳ vị trí hợp lệ |

`use_fqdn`: Chuyển đổi cho hostname đầy đủ hoặc bị cắt ngắn được cấu hình trên máy chủ. Theo mặc định, ossec cắt ngắn hostname ở giai đoạn đầu tiên ('.') khi tạo thông báo nhật ký hệ thống.

| Giá trị mặc định | no |
| --- | --- |
Giá trị được phép | yes, no |

`format`: Định dạng output của cảnh báo. Khi `jsonout_output` trong phần `global` được kích hoạt, các cảnh báo được đọc từ `alert.json` thay vì `alert.log` cho định dạng JSON.

| Giá trị mặc định | default |
| --- | --- |
| Giá trị được phép | default - cef: output data in the ArcSight Common Event Format - splunk: output data in a Splunk-friendly format - json: output data in the JSON format, có thể được sử dụng bởi nhiều công cụ khác nhau |

Ví dụ cấu hình:

```
<syslog_output>
  <server>192.168.1.3</server>
  <level>7</level>
  <format>json</format>
</syslog_output>
```

### Cấu hình email cảnh báo

Wazuh có thể được cấu hình để gửi thông báo email đến một hoặc nhiều địa chỉ email khi các quy tắc nhất định được kích hoạt hoặc cho các báo cáo sự kiện hàng ngày.

Email mẫu:

```
From: Wazuh <you@example.com>               5:03 PM (2 minutes ago)
to: me
-----------------------------
Wazuh Notification.
2017 Mar 08 17:03:05

Received From: localhost->/var/log/secure
Rule: 5503 fired (level 5) -> "PAM: User login failed."
Src IP: 192.168.1.37
Portion of the log(s):

Mar  8 17:03:04 localhost sshd[67231]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.1.37
uid: 0
euid: 0
tty: ssh

 --END OF NOTIFICATION
```

- Để định cấu hình Wazuh để gửi thông báo qua email, cài đặt email phải được định cấu hình trong phần `<global>` của tệp `ossec.conf`:

```
<ossec_config>
    <global>
        <email_notification>yes</email_notification>
        <email_to>me@test.com</email_to>
        <smtp_server>mail.test.com..</smtp_server>
        <email_from>wazuh@test.com</email_from>
    </global>
    ...
</ossec_config>
```

Để xem tất cả các tùy chọn cấu hình email có sẵn, hãy đi tới phần [global section](https://documentation.wazuh.com/3.10/user-manual/reference/ossec-conf/global.html#reference-ossec-global)

- Khi các cấu hình trên đã được định cấu hình, `email_alert_level` cần được đặt ở mức cảnh báo tối thiểu để kích hoạt email. Theo mặc định, mức này được đặt thành 12.

```
<ossec_config>
  <alerts>
      <email_alert_level>10</email_alert_level>
  </alerts>
  ...
</ossec_config>
```

Ví dụ này sẽ đặt mức tối thiểu thành 10.

- Sau khi alert_levelđã được định cấu hình, Wazuh cần được khởi động lại để thay đổi có hiệu lực:

	- Đối với Systemd:
	
	`systemctl restart wazuh-manager`
	
	- Đối với SysV Init:
	
	`service wazuh-manager restart`

> Chú ý: Wazuh không xử lý xác thực SMTP. Nếu dịch vụ email của bạn sử dụng điều này, bạn sẽ cần phải định cấu hình chuyển tiếp máy chủ email.

### Các option của Granular email

Wazuh cũng cho phép các tùy chọn cấu hình chi tiết để thông báo qua email. Dưới đây là một số cấu hình mẫu. Để biết thêm thông tin, hãy xem phần [email_alerts section](https://documentation.wazuh.com/3.x/user-manual/reference/ossec-conf/global.html#reference-ossec-global).

> Chú ý: Mức tối thiểu được cấu hình trong phần `alerts` cũng sẽ áp dụng và ghi đè các cấu hình này. Ví dụ: nếu bạn định cấu hình hệ thống của mình để gửi email khi quy tắc 526 được kích hoạt, nhưng quy tắc có mức thấp hơn mức tối thiểu được chỉ định, cảnh báo sẽ không được gửi.

- Thông báo qua email dựa trên level

```
<email_alerts>
  <email_to>you@example.com</email_to>
  <level>4</level>
  <do_not_delay />
</email_alerts>
```

Điều này sẽ gửi email đến `you@example.com` khi bất kỳ quy tắc nào có mức lớn hơn hoặc bằng 4 được kích hoạt.

- Thông báo qua email dựa trên level và agent

```
<email_alerts>
  <email_to>you@example.com</email_to>
  <event_location>server1</event_location>
  <do_not_delay />
</email_alerts>
```

Điều này sẽ gửi email đến you@example.comkhi quy tắc kích hoạt trên server1.

Trường `event_location` có thể được cấu hình để theo dõi một specific log, hostname hoặc mạng (IP)

Tông báo qua email dựa trên rule ID

```
<email_alerts>
  <email_to>you@example.com</email_to>
  <rule_id>515, 516</rule_id>
  <do_not_delay />
</email_alerts>
```

Điều này sẽ gửi email khi các quy tắc 515 hoặc 516 được kích hoạt trên bất kỳ agent nào.

Thông báo qua email dựa trên group:

Thông báo qua email có thể được định cấu hình để gửi email dựa trên một hoặc nhiều nhóm quy tắc:

```
<email_alerts>
  <email_to>you@example.com</email_to>
  <group>pci_dss_10.6.1,</group>
</email_alerts>
```

Điều này sẽ gửi cảnh báo khi bất kỳ quy tắc nào là một phần của nhóm `pci_dss_10.6.1` được kích hoạt trên bất kỳ thiết bị được giám sát nào.

- Multiple options và multiple emails

Ví dụ này cho thấy các khả năng cảnh báo email. Thông báo qua email có thể được gửi đến nhiều địa chỉ email, mỗi địa chỉ có một tiêu chí duy nhất:

```
<ossec_config>
  <email_alerts>
      <email_to>alice@test.com</email_to>
      <event_location>server1|server2</event_location>
  </email_alerts>
  <email_alerts>
      <email_to>is@test.com</email_to>
      <event_location>/log/secure$</event_location>
  </email_alerts>
  <email_alerts>
      <email_to>bob@test.com</email_to>
      <event_location>192.168.</event_location>
  </email_alerts>
  <email_alerts>
      <email_to>david@test.com</email_to>
      <level>12</level>
  </email_alerts>
 </ossec_config>
```

Cấu hình này sẽ gửi:

	- 1 email đến `alice@test.com` nếu có bất kỳ cảnh báo nào trên server1 hoặc server2 được kích hoạt
	
	- 1 email đến `is@test.com` nếu cảnh báo đến từ `/log/secure/`
	
	- 1 email đến `bob@test.com` nếu cảnh báo đến từ bất kỳ máy nào trên mạng 192.168.0.0/24
	
	- 1 email đến `david@test.com` nếu các cảnh báo có mức bằng hoặc cao hơn 12

- Bắt buộc gửi cảnh bảo bới email

Cũng có thể buộc một cảnh báo email về khai báo quy tắc dưới mức tối thiểu được cấu hình. Để làm như vậy, bạn cần sử dụng một trong các tùy chọn bên dưới .

Các giá trị có thể cho tùy chọn này là:

	- `alert_by_email`: Luôn cảnh báo qua email
	
	- `no_email_alert`: Không bao giờ cảnh báo qua email
	
	- `no_log`: Không ghi lại log thông báo này

Ví dụ:

```
<rule id="502" level="3">
  <if_sid>500</if_sid>
  <options>alert_by_email</options>
  <match>Ossec started</match>
  <description>Ossec server started.</description>
</rule>
```

Cấu hình này sẽ gửi email mỗi khi quy tắc 502 được kích hoạt bất kể mức tối thiểu được đặt là gì.

### Cấu hình xác thực SMTP server

Nếu máy chủ SMTP của bạn sử dụng xác thực (ví dụ như Gmail), chuyển tiếp máy chủ sẽ cần được định cấu hình vì Wazuh không hỗ trợ điều này. Postfix có thể được cấu hình để cung cấp khả năng này. 

- Cài đặt các gói cần thiết:

	- Ubuntu:
	
	`apt-get install postfix mailutils libsasl2-2 ca-certificates libsasl2-modules`
	
	- CentOS:
	
	`yum update && yum install postfix mailx cyrus-sasl cyrus-sasl-plain`

- Định cấu hình Postfix trong tệp `/etc/postfix/main.cf`, thêm các dòng này vào cuối tệp:

	- Ubuntu:
	
	```
	relayhost = [smtp.gmail.com]:587
	smtp_sasl_auth_enable = yes
	smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
	smtp_sasl_security_options = noanonymous
	smtp_tls_CAfile = /etc/ssl/certs/thawte_Primary_Root_CA.pem
	smtp_use_tls = yes
	```
	
	- CentOS:
	
	```
	relayhost = [smtp.gmail.com]:587
	smtp_sasl_auth_enable = yes
	smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
	smtp_sasl_security_options = noanonymous
	smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
	smtp_use_tls = yes
	```

- Định cấu hình địa chỉ email và mật khẩu:

```
echo [smtp.gmail.com]:587 USERNAME@gmail.com:PASSWORD > /etc/postfix/sasl_passwd
postmap /etc/postfix/sasl_passwd
chmod 400 /etc/postfix/sasl_passwd
```

- Bảo mật mật khẩu DB:

```
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

- Khởi động lại Postfix:

`systemctl reload postfix`

- Kiểm tra cấu hình với:

`echo "Test mail from postfix" | mail -s "Test Postfix" -r "you@example.com" you@example.com`

Kiểm tra email cảu bạn tại `you@example.com`.

- Cấu hình Wazuh trong tệp `/var/ossec/etc/ossec.conf` như sau:

```
<global>
  <email_notification>yes</email_notification>
  <smtp_server>localhost</smtp_server>
  <email_from>USERNAME@gmail.com</email_from>
  <email_to>you@example.com</email_to>
</global>
```

- Khởi động lại Wazuh:

	- Đối với Systemd:
	
	`systemctl restart wazuh-manager`
	
	- Đối với SysV Init:
	
	`service wazuh-manager restart`