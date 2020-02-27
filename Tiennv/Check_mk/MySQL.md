## MySQL

Đẩy dữ liệu từ check_mk sang mysql.

### Tổng quan

Trước khi đi vào phần cài đặt, ta hãy tìm hiểu 1 chút về mysql.

MySQL là một hệ thống quản trị cơ sở dữ liệu mã nguồn mở (gọi tắt là RDBMS) hoạt động theo mô hình client-server. Với RDBMS là viết tắt của Relational Database Management System. MySQL được tích hợp apache, PHP. MySQL quản lý dữ liệu thông qua các cơ sở dữ liệu. Mỗi cơ sở dữ liệu có thể có nhiều bảng quan hệ chứa dữ liệu. MySQL cũng có cùng một cách truy xuất và mã lệnh tương tự với ngôn ngữ SQL. MySQL được phát hành từ thập niên 90s.

### Lịch sử hình thành và phát triển của MySQL

Quá trình hình thành và phát triển của MySQL được tóm tắt như sau:

Công ty Thuy Điển MySQL AB phát triển MySQL vào năm 1994.

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

