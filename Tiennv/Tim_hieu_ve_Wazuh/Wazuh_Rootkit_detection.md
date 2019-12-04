## Wazuh Rootkit detection

Rootkit và trojan detection được thực hiện bằng 2 tệp `rootkit_files.txt` và `rootkit_trojans.txt`. Ngoài ra, kiểm tra ở mức độ thấp khác được thực hiện để phát hiện rootkit ở cấp độ kernel. Bạn có thể sử dụng các khả năng này bằng cách thêm vào các tệp này trong `ossec.conf`:

```
<rootcheck>
    <rootkit_files>/var/ossec/etc/shared/rootkit_files.txt</rootkit_files>
    <rootkit_trojans>/var/ossec/etc/shared/rootkit_trojans.txt</rootkit_trojans>
</rootcheck>
```

Đây là các tùy chọn có sẵn cho thành phần `rootcheck`:

- `rootkit_files`: Unix-based application level rootkit signatures

- `rootkit_trojans`: Unix-based application level trojan signatures

- `check_files`: Kích hoạt hoặc vô hiệu hóa kiểm tra rootkit. Mặc định là yes

- `check_trojans`: Kích hoạt hoặc vô hiệu hóa kiểm tra trojan. Mặc định là yes

- `check_dev`: Kiểm tra các tệp đáng ngờ trong hệ thống tập tin `/dev`. Mặc định là yes

- `check_sys`: Quét toàn bộ hệ thống để tìm sự bất thường ở mức độ thấp. Mặc định là yes

- `check_pids`: Kiểm tra các tiến trình tìm sự bất thường. Mặc định là yes

- `check_ports`: Kiểm tra tất cả các listening port xem có bất thường không. Mặc định là yes

- `check_if`: Kiểm tra các interface cho sự bất thường. Mặc định là yes

Rootcheck giúp bạn đáp ứng yêu cầu PCI DSS 11.4 liên quan đến xâm nhập, trojan và phần mềm độc hại nói chung:

11.4 : Sử dụng các kỹ thuật phát hiện xâm nhập hoặc ngăn chặn xâm nhập để phát hiện hoặc ngăn chặn sự xâm nhập vào mạng. Giữ cho tất cả các công cụ phát hiện và ngăn chặn xâm nhập, baseline và chữ ký được cập nhật. Các kỹ thuật phát hiện xâm nhập hoặc ngăn chặn xâm nhập (như IDS/IPS) so sánh lưu lượng truy cập vào mạng với các chữ ký được biết đến hoặc hành vi của hàng ngàn loại thỏa hiệp (hacker tool, Trojan và phần mềm độc hại khác) và gửi cảnh báo hoặc stop khi nó xảy ra.

Tùy chọn `rootcheck`:

Tùy chọn cấu hình để theo dõi policy và phát hiện bất thường

- `base_directory`: Thư mục cơ sở sẽ được thêm tiền tố vào các tùy chọn sau:

	- `rootkit_files`

	- `rootkit_trojans`

	- `system_audit`

| Giá trị mặc định (UNIX) | / |
| --- | --- |
| Giá trị mặc định (Windows) | C:\ |
| Giá trị được phép | Đường dẫn đến 1 thư mục |

- `ignore`: Danh sách các tập tin hoặc thư mục sẽ bị bỏ qua (một mục nhập trên mỗi dòng). Nhiều dòng có thể được nhập để bao gồm nhiều tệp hoặc thư mục. Những tập tin và thư mục này sẽ bị bỏ qua trong quá trình quét.

| Giá trị được phép | Bất kỳ thư mục hoặc tên tập tin |
| --- | --- |
| Có hiệu lực cho | `check_sys`, `check_dev`, `check_files` |
| Ví dụ | `/etc` |

- `rootkit_files`: Thay đổi vị trí của cơ sở dữ liệu tập tin rootkit.

| Giá trị mặc định | `/var/ossec/etc/shared/rootkit_files.txt` |
| --- | --- |
| Giá trị được phép | Một tệp có chữ ký của tập tin rootkit |

- `rootkit_trojans`: Thay đổi vị trí của cơ sở dữ liệu rootkit trojan.

| Giá trị mặc định | `/var/ossec/etc/shared/rootkit_trojans.txt` |
| --- | --- |
| Giá trị được phép | Một tập tin với chữ ký trojan |

- `windows_audit`: Chỉ định đường dẫn đến tệp định nghĩa Windows audit.

| Giá trị mặc định | không có |
| --- | --- |
| Gía trị được phép | Đường dẫn đến tệp định nghĩa Windows audit |

- `system_audit`: Chỉ định đường dẫn đến tệp định nghĩa audit cho các hệ thống Unix-like.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Tệp định nghĩa audit cho các hệ thống Unix-like |

- `windows_apps`: Chỉ định đường dẫn đến tệp định nghĩa ứng dụng Windows.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Đường dẫn đến một tập tin ứng dụng Windows def. |

- `windows_malware`: Chỉ định đường dẫn đến tệp định nghĩa phần mềm độc hại Windows.

| Giá trị mặc định | không có |
| --- | --- |
| Giá trị được phép | Đường dẫn đến tệp định nghĩa phần mềm độc hại Windows |

- `scanall`: Cấu hình `rootcheck` để quét toàn bộ hệ thống. Tùy chọn này có thể dẫn đến một số báo động giả.

| Giá trị mặc định | no |
| --- | --- |
| Giá trị được phép | yes, no |

- `frequency`: Tần suất mà `rootcheck` sẽ được thực thi (tính bằng giây).

| Giá trị mặc định | 36000 |
| --- | --- |
| Giá trị được phép | Số dương (s) |

- `disabled`: Vô hiệu hóa việc thực hiện `rootcheck`.

| Giá trị mặc định | no |
| --- | --- |
Giá trị được phép | yes, no |

- `check_dev`: Kích hoạt hoặc vô hiệu hóa việc kiểm tra `/dev`.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_files`: Kích hoạt hoặc vô hiệu hóa việc kiểm tra file.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_if`: Cho phép hoặc vô hiệu hóa việc kiểm tra network interface.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_pids`: Cho phép hoặc vô hiệu hóa việc kiểm tra các ID tiến trình.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_ports`: Cho phép hoặc vô hiệu hóa việc kiểm tra các port.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_sys`: Cho phép hoặc vô hiệu hóa việc kiểm tra các đối tượng hệ thống tệp bất thường.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_unixaudit`: Kích hoạt hoặc vô hiệu hóa việc kiểm tra unixaudit.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_winapps`: Kích hoạt hoặc vô hiệu hóa việc kiểm tra winapps.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_winaudit`: Kích hoạt hoặc vô hiệu hóa việc kiểm tra winaudit.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `check_winmalware`: Kích hoạt hoặc vô hiệu hóa kiểm tra phần mềm độc hại Windows.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

- `skip_nfs`: Kích hoạt hoặc vô hiệu hóa việc quét các hệ thống tệp được gắn trên mạng (Hoạt động trên Linux và FreeBSD). Hiện tại, `skip_nfs` sẽ loại trừ việc kiểm tra các tệp mount CIFS hoặc NFS.

| Giá trị mặc định | yes |
| --- | --- |
| Giá trị được phép | yes, no |

### Cấu hình Unix mặc định

```
<!-- Policy monitoring -->
  <rootcheck>
  <disabled>no</disabled>
  <check_unixaudit>yes</check_unixaudit>
  <check_files>yes</check_files>
  <check_trojans>yes</check_trojans>
  <check_dev>yes</check_dev>
  <check_sys>yes</check_sys>
  <check_pids>yes</check_pids>
  <check_ports>yes</check_ports>
  <check_if>yes</check_if>

  <!-- Frequency that rootcheck is executed - every 12 hours -->
  <frequency>43200</frequency>

  <rootkit_files>/var/ossec/etc/shared/rootkit_files.txt</rootkit_files>
  <rootkit_trojans>/var/ossec/etc/shared/rootkit_trojans.txt</rootkit_trojans>

  <system_audit>/var/ossec/etc/shared/system_audit_rcl.txt</system_audit>
  <system_audit>/var/ossec/etc/shared/system_audit_ssh.txt</system_audit>
  <system_audit>/var/ossec/etc/shared/cis_debian_linux_rcl.txt</system_audit>

  <skip_nfs>yes</skip_nfs>
</rootcheck>
```

### Use case

Wazuh thực hiện một số thử nghiệm để phát hiện rootkit. Một trong số đó là kiểm tra các tập tin ẩn trong `/dev`. Thư mục `/dev` chỉ nên chứa các tệp dành riêng cho thiết bị như đĩa cứng IDE chính (`/dev/hda`), kernel random number generators (`/dev/random` và `/dev/urandom`), v.v. Bất kỳ tệp bổ sung nào, bên ngoài các tệp dành riêng cho thiết bị dự kiến, sẽ được kiểm tra vì nhiều rootkit sử dụng `/dev` như một phân vùng lưu trữ để ẩn tập tin. Trong ví dụ sau, tệp `.hid` được phát hiện bởi OSSEC và tạo cảnh báo tương ứng.

```
[root@manager /]ls -a /dev | grep '^\.'
.
..
.hid
[root@manager /]tail -n 25 /var/ossec/logs/alerts/alerts.log
Rule: 502 (level 3) -> 'Ossec server started.'
ossec: Ossec started.

** Alert 1454086362.26393: mail  - ossec,rootcheck
2016 Jan 29 16:52:42 manager->rootcheck
Rule: 510 (level 7) -> 'Host-based anomaly detection event (rootcheck).'
File '/dev/.hid' present on /dev. Possible hidden file.
```