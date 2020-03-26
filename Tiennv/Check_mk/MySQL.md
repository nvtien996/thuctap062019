## MySQL

Đẩy dữ liệu từ check_mk sang mysql.

### Tổng quan

Trước khi đi vào phần cài đặt, ta hãy tìm hiểu 1 chút về mysql.

MySQL là một hệ thống quản trị cơ sở dữ liệu mã nguồn mở (gọi tắt là RDBMS) hoạt động theo mô hình client-server. Với RDBMS là viết tắt của Relational Database Management System. MySQL được tích hợp apache, PHP. MySQL quản lý dữ liệu thông qua các cơ sở dữ liệu. Mỗi cơ sở dữ liệu có thể có nhiều bảng quan hệ chứa dữ liệu. MySQL cũng có cùng một cách truy xuất và mã lệnh tương tự với ngôn ngữ SQL. MySQL được phát hành từ thập niên 90s.

### Lịch sử hình thành và phát triển của MySQL

Quá trình hình thành và phát triển của MySQL được tóm tắt như sau:

Công ty Thụy Điển MySQL AB phát triển MySQL vào năm 1994.

Phiên bản đầu tiên của MySQL phát hành năm 1995

Công ty Sun Microsystems mua lại MySQL AB trong năm 2008

Năm 2010 tập đoàn Oracle thâu tóm Sun Microsystems. Ngay lúc đó, đội ngũ phát triển của MySQL tách MySQL ra thành 1 nhánh riêng gọi là MariaDB. Oracle tiếp tục phát triển MySQL lên phiên bản 5.5.

2013 MySQL phát hành phiên bản 5.6

2015 MySQL phát hành phiên bản 5.7

MySQL đang được phát triển lên phiên bản 8.0

MySQL hiện nay có 2 phiên bản là bản miễn phí (MySQL Community Server) và có phí (Enterprise Server).

### Cấu hình

#### Cấu hình trên cả node cmk và node mysql

- Cài MariaDB:

`yum install -y mariadb mariadb-server mariadb-devel`

- Cài các gói bổ trợ:

`yum install -y gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel xinetd unzip httpd php php-fpm curl`

- Backup lại file `sysctl.conf`:

`cp /etc/sysctl.conf /etc/sysctl.conf_backup`

- Chỉnh sửa file cấu hình:

```
sed -i '/msgmnb/d' /etc/sysctl.conf
sed -i '/msgmax/d' /etc/sysctl.conf
sed -i '/shmmax/d' /etc/sysctl.conf
sed -i '/shmall/d' /etc/sysctl.conf
printf "\n\nkernel.msgmnb = 131072000\n" >> /etc/sysctl.conf
printf "kernel.msgmax = 131072000\n" >> /etc/sysctl.conf
printf "kernel.shmmax = 4294967295\n" >> /etc/sysctl.conf
printf "kernel.shmall = 268435456\n" >> /etc/sysctl.conf
sysctl -e -p /etc/sysctl.conf
```

#### Trên node mysql

- Start và enable mariadb:

```
systemctl start mariadb
systemctl enable mariadb
```
- Đặt pass cho root:

`/usr/bin/mysqladmin -u root password 'password'`

> thay `password` bằng password bạn muốn đặt

- Tạo DB:

`mysql -u root -p'password'`

```
CREATE DATABASE nagios DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'ndoutils'@'%' IDENTIFIED BY 'ndoutils_password';
GRANT USAGE ON *.* TO 'ndoutils'@'%' IDENTIFIED BY 'ndoutils_password' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;
GRANT ALL PRIVILEGES ON nagios.* TO 'ndoutils'@'%' WITH GRANT OPTION ;
\q
```

> thay `ndoutils_password` bằng password của bạn

- Tắt firewall:

```
systemctl stop firewalld
systemctl disable firewalld
```

#### Cấu hình trên node Check_MK

- Tải và giải nén NDO Utils v2.1.3:

```
cd /tmp
wget https://github.com/NagiosEnterprises/ndoutils/releases/download/ndoutils-2.1.3/ndoutils-2.1.3.tar.gz
tar xzf ndoutils-2.1.3.tar.gz
```

- Biên dịch và cài đặt NDO Utils

```
cd /tmp/ndoutils-2.1.3/
./configure --prefix=/omd/sites/wjbu/usr/local/nagios/ --enable-mysql --with-ndo2db-user=wjbu -with-ndo2db-group=wjbu
make all
make install
make install-config
```

> thay `wjbu` bằng site của bạn

- Configure DB:

```
cd db
./installdb -u 'ndoutils' -p 'ndoutils_password' -h 'ip_node_mysql' -d nagios
cd ..
```

> thay `ip_node_mysql` bằng ip máy cài đặt mysql server

`chown -R wjbu /omd/sites/wjbu/usr/local/`

`su wjbu`

```
cp /tmp/ndoutils-2.1.3/src/ndomod-3x.o ~/usr/local/nagios/bin/ndomod.o
cp /tmp/ndoutils-2.1.3/src/ndo2db-3x ~/usr/local/nagios/bin/ndo2db
chmod 0744 ~/usr/local/nagios/bin/ndo*
mv ~/usr/local/nagios/etc/ndo2db.cfg-sample ~/usr/local/nagios/etc/ndo2db.cfg
mv ~/usr/local/nagios/etc/ndomod.cfg-sample ~/usr/local/nagios/etc/ndomod.cfg
```

`vi ~/usr/local/nagios/etc/ndo2db.cfg`

tìm và sửa các dòng sau:

```
db_host=ip_node_mysql
db_user=ndoutils
db_pass=ndoutils_password
```

`exit`

- Install deamon-init :

`make install-init`

- Thực hiện các commands sau để tạo `Broker Module`:

```
printf "\n\n# NDOUtils Broker Module\n" >> /omd/sites/wjbu/etc/nagios/nagios.cfg
printf "broker_module=/omd/sites/wjbu/usr/local/nagios/bin/ndomod.o config_file=/omd/sites/wjbu/usr/local/nagios/etc/ndomod.cfg\n" >> /omd/sites/wjbu/etc/nagios/nagios.cfg
```

- Start và enable `ndo2db.service`:

```
systemctl enable ndo2db.service
systemctl start ndo2db.service
```

- Restart site giám sát:

`omd restart wjbu`

- Kiểm tra thông tin dữ liệu được đẩy ra MySQL:

`echo 'select * from nagios.nagios_logentries;' | mysql -u ndoutils -p'ndoutils_password' -h 192.168.20.82`

### Data Processing Options

Các  NDOUtils NDOMOD broker module có thể được cấu hình để chỉ xử lý một số loại dữ liệu. Điều này được định nghĩa trong tệp `ndomod.cfg`.

Tệp tin `ndomod.cfg` như trong bài này có đường dẫn là `/opt/omd/sites/wjbu/usr/local/nagios/etc/ndomod.cfg`

Với những phiên bản Nagios đầu tiên, các tùy chọn khác nhau đã được xác định bằng cách tính toán bitmask và được xác định với:

`data_processing_options=xxxx`

trong đó `xxxx` là số bitmask.

Và để tính toán số bitmask, ta sẽ cần giá trị của các tùy chọn sau:

| Option | Giá trị |
| --- | --- |
| NDOMOD_PROCESS_PROCESS_DATA | 1 |
| NDOMOD_PROCESS_TIMED_EVENT_DATA | 2 |
| NDOMOD_PROCESS_LOG_DATA | 4 |
| NDOMOD_PROCESS_SYSTEM_COMMAND_DATA | 8 |
| NDOMOD_PROCESS_EVENT_HANDLER_DATA | 16 |
| NDOMOD_PROCESS_NOTIFICATION_DATA | 32 |
| NDOMOD_PROCESS_SERVICE_CHECK_DATA | 64 |
| NDOMOD_PROCESS_HOST_CHECK_DATA | 128 |
| NDOMOD_PROCESS_COMMENT_DATA | 256 |
| NDOMOD_PROCESS_DOWNTIME_DATA | 512 |
| NDOMOD_PROCESS_FLAPPING_DATA | 1024 |
| NDOMOD_PROCESS_PROGRAM_STATUS_DATA | 2048 |
| NDOMOD_PROCESS_HOST_STATUS_DATA | 4096 |
| NDOMOD_PROCESS_SERVICE_STATUS_DATA | 8192 |
| NDOMOD_PROCESS_ADAPTIVE_PROGRAM_DATA | 16384 |
| NDOMOD_PROCESS_ADAPTIVE_HOST_DATA| 32768 |
| NDOMOD_PROCESS_ADAPTIVE_SERVICE_DATA | 65536 |
| NDOMOD_PROCESS_EXTERNAL_COMMAND_DATA | 131072 |
| NDOMOD_PROCESS_OBJECT_CONFIG_DATA | 262144 |
| NDOMOD_PROCESS_MAIN_CONFIG_DATA | 524288 |
| NDOMOD_PROCESS_AGGREGATED_STATUS_DATA | 1048576 |
| NDOMOD_PROCESS_RETENTION_DATA | 2097152 |
| NDOMOD_PROCESS_ACKNOWLEDGEMENT_DATA | 4194304 |
| NDOMOD_PROCESS_STATE_CHANGE_DATA | 8388608 |
| NDOMOD_PROCESS_CONTACT_STATUS_DATA | 16777216 |
| NDOMOD_PROCESS_ADAPTIVE_CONTACT_DATA | 33554432 |

Ví dụ: trong Nagios XI, bitmask của 67108669 được sử dụng bao gồm tất cả các tùy chọn này EXCEPT cho NDOMOD_PROCESS_TIMED_EVENT_DATA, NDOMOD_PROCESS_HOST_CHECK_DATA và NDOMOD_PROCESS_SERVICE

`data_processing_options=67108669`

Nhưng kể từ ndoutils2.x, các tùy chọn này có thể được xác định riêng lẻ thay vì sử dụng `data_processing_options`

Hãy thêm ký tự `#` ở phía trước dòng `data_processing_options` (nếu có).

Bây giờ dán đoạn sau vào tệp (sau dòng bạn vừa thêm `#`):

```
acknowledgement_data=1
adaptive_contact_data=1
adaptive_host_data=1
adaptive_program_data=1
adaptive_service_data=1
aggregated_status_data=1
comment_data=1
contact_status_data=1
downtime_data=1
event_handler_data=1
external_command_data=1
flapping_data=1
host_check_data=0
host_status_data=1
log_data=1
main_config_data=1
notification_data=1
object_config_data=1
process_data=1
program_status_data=1
retention_data=1
service_check_data=0
service_status_data=1
statechange_data=1
system_command_data=1
timed_event_data=0
```

Lưu lại file.

Do ở đây tôi chỉ quan tâm đến các dữ liệu liên quan đến host và service nên tôi chỉ cần các tùy chọn như sau:

```
adaptive_host_data=1
adaptive_service_data=1
host_check_data=1
host_status_data=1
log_data=1
service_check_data=1
service_status_data=1
```

Restart lại ndo2db service và site giám sát:

```
systemctl restart ndo2db.service
omd restart wjbu
```

### Thời gian lưu trữ trong database table

Một số bảng cơ sở dữ liệu chứa dữ liệu sự kiện Nagios có thể lớn dần theo thời gian. Hầu hết các quản trị viên sẽ muốn sắp xếp lại các bảng này và chỉ giữ một lượng dữ liệu nhất định trong đó. Các tùy chọn bên dưới được sử dụng để chỉ định thời gian (tính bằng PHÚT) dữ liệu sẽ được cho phép lưu lại trong các bảng khác nhau trước khi bị xóa. Sử dụng giá trị `0` cho bất kỳ giá trị nào có nghĩa là bảng cụ thể đó KHÔNG được tự động dọn dẹp.

Các tùy chọn bên dưới sẽ được lưu trong tệp `/opt/omd/sites/wjbu/usr/local/nagios/etc/ndo2db.cfg` như bên trên đã cấu hình

```
# Keep timed events for 24 hours
max_timedevents_age=1440

# Keep system commands for 1 week
max_systemcommands_age=10080

# Keep service checks for 1 week
max_servicechecks_age=10080

# Keep host checks for 1 week
max_hostchecks_age=10080

# Keep event handlers for 31 days
max_eventhandlers_age=44640

# Keep external commands for 31 days
max_externalcommands_age=44640

# Keep notifications for 31 days
max_notifications_age=44640

# Keep contactnotifications for 31 days
max_contactnotifications_age=44640

# Keep contactnotificationmethods for 31 days
max_contactnotificationmethods_age=44640

# Keep logentries for 90 days
max_logentries_age=129600

# Keep acknowledgements for 31 days
max_acknowledgements_age=44640
```