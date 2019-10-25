## Cấu hình log rotation

#### Yêu cầu

Thực hành cấu hình log rotation:

- rotate file log theo giờ

- theo kích thước file log

- nén lại các file cũ

- xóa các file sau khi nén được 2 file

#### Thực hành

Để thực hiện log rotation, ta sẽ sử dụng 1 công cụ là logrotate. Nó sẽ load cấu hình trong file /etc/logrotate.conf theo mặc định của script logrotate

Hệ thống Linux sẽ chạy chương trình logrotate theo lịch crontab, mặc định sẽ là daily, nó được lưu tại /etc/cron.daily/logrotate

- Vì mặc định logrotate được chạy theo cron.daily, nên nếu muốn rotate log theo giờ, ta phải di chuyển script của logrotate từ /etc/cron.daily sang /etc/cron.hourly

`mv /etc/cron.daily/logrotate /etc/cron.hourly`

- Tại thư mục /etc/logrotate.d/ tạo 1 file có tên testhourlylog với nội dung như sau

```
/tmp/hourlylog.log
{
	missingok
	hourly
	rotate 2
	compress
}
```

trong đó:

/tmp/hourlylog: đường dẫn của file log`

missingok: nếu file log bị mất hoặc không có tồn tại `*.log` thì logrotate sẽ di chuyển tới phần cấu hình log của file log khác mà không phải xuất ra thông báo lỗi

hourly: rotate log file mỗi giờ

rotate 2: số lượng file log cũ đã được giữ lại sau khi rotate (2)

compress: logrotate sẽ nén tất cả các file log lại sau khi đã được rotate (mặc định bằng gzip)

- Tạo 1 file mới có tên testsizelog với nội dung sau

```
/tmp/sizelog.log
{
	missingok
	size 10M
	rotate 2
	compress
}
```

trong đó:

/tmp/sizelog: đường dẫn file log`

size 10M: rotate dựa theo dung lượng file (10M, các đơn vị kích thước file có thể sử dụng là k, M, G)