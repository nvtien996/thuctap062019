## Giám sát truy cập vào 1 thư mục

- Trên Agent:

Install Audit:

Để sử dụng Audit system, bạn phải cài đặt gói audit trên hệ thống của mình. Nếu bạn chưa cài đặt gói này, hãy thực hiện lệnh sau với tư cách là người dùng root để cài đặt nó.

Red Hat, CentOS and Fedora:

`yum install -y audit audit-libs`

Debian, Ubuntu base Linux distributions:

`apt-get install auditd audispd-plugins`

Cho phép dịch vụ:

```
systemctl enable auditd.service
systemctl start auditd.service
```

Edit tệp `ossec.conf`:

Wazuh phải nhận biết được các sự kiện do Audit phát hiện. Vì vậy, nó cần được cấu hình để đọc tệp audit log file:

```
<localfile>
  <log_format>audit</log_format>
  <location>/var/log/audit/audit.log</location>
</localfile>
```

Restart Wazuh:

Sau đó, restart Wazuh agent để áp dụng các thay đổi:

Đối với Systemd:

`systemctl restart wazuh-agent`

Đối với SysV Init:

`service wazuh-agent restart`

Trong ví dụ này, tôi sẽ theo dõi mọi loại quyền truy cập trong thư mục `/home`:

```
auditctl -w /home -p w -k audit-wazuh-w
auditctl -w /home -p a -k audit-wazuh-a
auditctl -w /home -p r -k audit-wazuh-r
auditctl -w /home -p x -k audit-wazuh-x
```

## Giám sát hành động của người dùng

Ở đây tôi chọn kiểm tra tất cả các lệnh được chạy bởi người dùng có quyền quản trị viên. Cấu hình audit cho việc này khá đơn giản:

```
auditctl -a exit,always -F euid=0 -F arch=b64 -S execve -k audit-wazuh-c
auditctl -a exit,always -F euid=0 -F arch=b32 -S execve -k audit-wazuh-c
```

## Giám sát việc thực hiện lệnh cụ thể với quyền root

Wazuh cho phép bạn duy trì các danh sách flat-file CDB được biên dịch thành định dạng nhị phân đặc biệt để tạo điều kiện tra cứu hiệu suất cao trong quy tắc Wazuh. Các danh sách như vậy phải được tạo dưới dạng tệp, được thêm vào cấu hình Wazuh và sau đó được biên dịch. Sau đó, các quy tắc tìm kiếm các trường được giải mã trong các danh sách CDB đó như là một phần của tiêu chí khớp của chúng có thể được xây dựng.

Tôi sẽ tạo 1 danh sách các lệnh sẽ được theo dõi trong tệp `/var/ossec/etc/lists/kernel_control_commands`:

```
sudo
systemctl
reboot
restart
```

Sau đó, thêm dòng sau trong phần `ruleset` của tệp `/var/ossec/etc/ossec.conf` trên Wazuh-manager:

`<list>etc/lists/kernel_control_commands</list>`

Sau đó, thêm quy tắc tùy chỉnh sau:

```
<rule id="100010" level="8">
  <if_sid>100002</if_sid>
  <list field="audit.command" lookup="match_key">etc/lists/kernel_control_commands</list>
  <description>Audit: [Kernel modification] ($(audit.command)) Executed with loginuid user $(audit.auid): $(audit.execve.a0) $(audit.execve.a1) $(audit.execve.a2) </description>
  <group>audit_command,</group>
</rule>
```

quy tắc này sẽ chỉ kích hoạt nếu sự kiện trước đó đã được kích hoạt và lệnh được thực hiện khớp với danh sách CDB được chỉ định.

Cuối cùng, chạy lệnh `/var/ossec/bin/ossec-makelists` để biên dịch danh sách CDB và sau đó khởi động lại Wazuh-manager để làm cho quy tắc mới có hiệu lực.

## Giám sát người dùng đang chạy các lệnh với quyền root

Thêm quy tắc tùy chỉnh sau vào tệp `/var/ossec/etc/rules/local_rules.xml`:

```
<rule id="100002" level="6">
  <if_sid>80792</if_sid>
  <field name="audit.euid">0</field>
  <description>Audit: Root command execution: $(audit.exe) with loginuid user $(audit.auid)</description>
  <group>audit_command,</group>
</rule>
```