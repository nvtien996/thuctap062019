## Giám sát 1 tiến trình

Gải sử rằng bạn muốn theo dõi các tiến trình đang chạy và đưa ra cảnh báo nếu 1 tiến trình quan trọng không chạy.

Ví dụ trên máy tính Windows và notepad.exe là tiến trình quan trọng để theo dõi:

- Trên Agent, mở tệp `local_iternal_options.conf` và thêm các dòng sau để chấp nhận remote command từ manager:

```
# Logcollector - Whether or not to accept remote commands from the manager
logcollector.remote_commands=1
wazuh_command.remote_commands=1
```

- Sau đó, trên manager, mở tệp `agent.conf` ( thường thì tất cả các agent thuộc về một nhóm được gọi `default` nên đường dẫn sẽ là `/var/ossec/etc/shared/default/agent.conf`) và thêm các dòng sau:

```
<localfile>
  <log_format>full_command</log_format>
  <command>tasklist</command>
  <frequency>120</frequency>
</localfile>
```

Tag `<frequency>` sẽ xác định mức độ thường xuyên lệnh sẽ được chạy (tính bằng giây).

- Sau đó, ta phải xác định rule tương ứng bên trong tệp `/var/ossec/etc/rules/local_rules.xml`:

```
<rule id="100010" level="6">
  <if_sid>530</if_sid>
  <match>^ossec: output: 'tasklist'</match>
  <description>Important process not running.</description>
  <group>process_monitor,</group>
</rule>

<rule id="100011" level="0">
  <if_sid>100010</if_sid>
  <match>notepad.exe</match>
  <description>Processes running as expected</description>
  <group>process_monitor,</group>
</rule>
```

Rule đầu tiên (100010) sẽ tạo ra 1 alert ("Important process not running"), trừ khi nó được ghi đè bởi quy tắc con của nó (100011) match với `notepad.exe` trong output của command. Bạn có thể thêm nhiều quy tắc con nếu cần để liệt kê tất cả các tiến trình quan trọng bạn muốn theo dõi. Bạn cũng có thể điều chỉnh ví dụ này để giám sát các tiến trình Linux bằng cách thay đổi `<command>` từ `tasklist` sang lệnh liệt kê các tiến trình trên Linux như `ps -auxw`.