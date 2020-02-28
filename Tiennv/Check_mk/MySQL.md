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

`echo 'select * from nagios.nagios_logentries;' | mysql -u ndoutils -p'dsr11qaz' -h ip_node_mysql`