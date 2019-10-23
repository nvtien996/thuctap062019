## Tổng quan về log

#### Khái niệm về log

Tong máy tính, log đơn giản là 1 file nhật ký ghi lại các hành động hay các sự kiện xảy ra trong hệ thống.

Log file thường là các file văn bản thông thường dưới dạng "clear text" và có thể dễ dàng đọc nó bằng việc sử dụng các trình soạn thao văn bản như Notepad (Windows) hay vi, vim, nano ... (linux) hoặc các trình xem văn bản thông thường như cat, tail, head ...

Đối với các hệ thống linux thì file log thường được lưu trữ tại thư mục /var/log

#### Công dụng của log

Log liên tục ghi lại các thông báo về hoạt động cũng như các sự kiện trong hệ thống. Từ các tệp nhật ký, người quản trị hệ thống có thể quan sát và tìm thấy các chi tiết về hiệu suất hoạt động, bảo mật, thông báo lỗi và các vấn đề tiềm ẩn đối với hệ thống. Do đó, các vấn đề nào mà máy chủ đang gặp phải đều có thể tìm thấy và giải quyết bằng cách đọc các file log

Log chủ yếu được dùng cho mục đích quản trị hệ thống, khắc phục sự cố khi hệ thống xảy ra vấn đề, troubleshooting trong quá trình cài đặt chương trình hay các services, truy vết các event đã và đang xảy ra, tra cứu nhanh các thông tin hệ thống, ...

Về cơ bản, phân tích log là điều đầu tiên mà người quản trị cần làm khi phát hiên ra sự cố

#### Các loại log

Log có thể được chia làm 4 loại cơ bản

- Application logs

- Event logs

- Service logs

- System logs

#### 1 số thuật ngữ cơ bản về log

có 1 số thuật ngữ cần biết như sau

| Thuật ngữ | Ý nghĩa |
| --- | --- |
| Facility | Facility giúp kiểm soát log đến dựa vào nguồn gốc được quy định như từ ứng dụng hay tiến trình nào. Syslog sử dụng facility để quy hoạch lại log, như vậy có thể coi facility là đại diện cho đối tượng tạo ra log message (kernel, process, apps, ...) |
| Priority (level) | mức độ quan trọng của log message được chỉ định, các mức được định nghĩa trong syslog, từ việc gỡ lỗi, thông tin dịch vụ cho đến các cảnh báo quan trọng khác |
| Selector | sự kết hợp giữa facility và level (facility.level), tức khi 1 event xảy ra thì nón sẽ xem có match bất kỳ selector nào không, nếu có thì hành động (action) tương ứng khi cấu hình sẽ được thực thi |
| Action | đại diện cho địa chỉ của log message tương ứng với facility.level. Action có thể là tên 1 file (ghi log vào file), 1 thiết bị (gửi đến thiết bị) 1 host name (gửi đến máy chủ có tên host name tương ứng - cần mở cổng 514), hay 1 danh sách người dùng ngăn cách bằng dấu `,` (gửi đến người dùng tương ứng) hoặc dấu `*` |

#### Các nguồn sinh log (Facility Level)

| Facility code | Keyword | Mô tả |
| 0 | kernel | Kernel messages |
| 1 | user | User-level messages |
| 2 | mail | Mail system |
| 3 | deamon | System deamons |
| 4 | auth | Security / authentication messages |
| 5 | syslog | Messages được tạo nội bộ bởi syslogd |
| 6 | lpr | Line printer subsystem |
| 7 | news | Network news subsystem (đặc biệ là từ NNTP - Network News Transfer Protocol - máy chủ quản lý các nhóm tin |
| 8 | uucp | UUCP subsystem (Unix to Unix Copy Program - 1 giao thức cũ đáng chú ý được sử dụng để phân phối email) |
| 9 | cron | Clock deamon |
| 10 | authpriv | Security / authentication messages (cũng giống như auth nhưng khi log này được ghi vào file chỉ có thể được đọc bởi user được lựa chọn |
| 11 | ftp | FTP deamon |
| 12 | ntp | NTP subsystem |
| 13 | security | Log audit |
| 14 | console | Log alert |
| 15 | solaris-cron | Scheduling deamon |
| 16 - 23 | local0 - local7 | Local use |

#### Các mức cảnh báo của log (Priority Level)

Log message trong hệ thống là rất nhiều, vì vậy để thuận tiên cho việc phân tích mức độ quan trọng của log, mỗi dòng log dều đueoecj gắn 1 mã cảnh báo, tương ứng với mức độ quan trọng của dòng log đó

Thống kê số lượng về mức độ cảnh báo của các file log cũng cho ta thấy 1 phần tình trạng hệ thống.

| Code | Mức cảnh báo | Keyword | Keyword không còn sử dụng | Ý nghĩa |
| --- | --- | --- | --- | --- |
| 0 | Emergency | emerg | panic | Thông báo tình trạng khẩn cấp, chẳng hạn như hệ thống có sự cố xảy ra, thường được gửi đến tất cả người dùng |
| 1 | Alert | alert | | Hệ thống hỏng hóc, cần can thiệp ngay lập tức |
| 2 | Critical | crit | | Sự cố quan trọng, chẳng hạn như lỗi phần cứng |
| 3 | Error | err | error | Thông báo lỗi thông thường |
| 4 | Warning | warning | warn | Mức cảnh báo đối với hệ thống |
| 5 | Notice | notice | | Thông báo này không phải là lỗi, chú ý đến hệ thống |
| 6 | Informational | info | | Thông báo thông tin |
| 7 | Debug | debug | | Thông báo gỡ lỗi |

#### 1 số file log quan trọng

Như đã nói ở trên thì các file log thường sẽ được lưu tại thư mục /var/log

Dưới đây là các file log quan trọng trong hệ thống

- /var/log/syslog hoặc /var/log/message: thông báo chung cũng như thông tin liên qua đến hệ thống. Về cơ bản, nhật ký này lưu trữ tất cả dữ liệu hoạt động trên toàn hệ thống. Lưu ý rằng hoạt động cho các hệ thống dựa trên Redhat như CentOS hoặc Rhel được lưu trữ trong tệp message; trong khi Ubuntu và các hệ thống dựa trên Debian khác được lưu trữ trong syslog

- /var/log/auth.log hoặc /var/log/secure: lưu trữ nhật ký xác thực, bao gồm cả đăng nhập thành công, thất bại và phương thức xác thực. Với Debian/Ubuntu được lưu trữ trong /var/log/auth.log; trong khi Redhat/CentOS được lưu trữ trong /var/log/secure

- /var/log/boot.log: lưu trữ thông tin liên quan đến khởi động và mọi thông báo được ghi lại trong quá trình khởi động

- /var/log/maillog hoặc /var/log/mail.log: lưu trữ tất cả các nhật ký liên quan đến máy chủ thư, hữu ích khi bạn cần thông tin về postfix, smtpd hoặc bất cứ dịch vụ nào liên quan đến email đang chạy trên máy chủ của bạn

- /var/log/kern.log: lưu trữ nhật ký kernel và dữ liệu cảnh báo. Nhật ký này cũng có các giá trị để các kernel tùy chỉnh

- /var/log/dmesg: thông báo liên quan đến trình điều khiển thiết bị. Lệnh "dmesg" có thể được sử dụng để xem các thông báo trong tệp này

- /var/log/faillog: chứa thông tin tất cả các lần đăng nhập thất bại, rất hữu ích để có thể biết được thông tin về các vi phạm bảo mật, chẳng hạn như nỗ lực hack thông tin đăng nhập

- /var/log/cron: lưu trữ các thông báo liên quan đến cron job, chẳng hạn như khi crom deamon khởi tạo 1 công việc hoặc các thông báo lỗi liên quan

- /var/log/yum.log: nếu cài đặt các gói bằng lệnh "yum", nhật ký này lưu trữ tất cả thông tin liên quan, có thể hữu ích trong việc xác định xem 1 gói và các thành phần đã được cài đặt đúng chưa. Với Debian/Ubuntu và các phiên bản dùng hệ thống quản lý gói dpkg là dpkg.log

- /var/log/httpd: 1 thư mục chứa các tệp error_log và access_log của deamon apache httpd. Các error_log chứa tất cả các lỗi gặp phải của httpd. Những lỗi này bao gồm các vấn đề về bộ nhớ và các lỗi liên quan đến hệ thống khác. access_log chứa 1 bản ghi của tất cả các yêu cầu nhận được qua HTTP

- /var/log/mysqld.log hoặc /var/log/mysql.log: tệp nhật ký MySQL ghi nhật ký tất cả thông báo debug, thất bại, thành công. Chứa thông tin về việc bắt đầu, dừng và khởi động lại MySQL deamon mysqld. Redhat/CentOS/Fedora và các hệ thống dựa trên Redhat khác sử dụng /var/log/mysqld.log; trong khi Debian/Ubuntu sử dụng thư mục /var/log/mysql.log

- /var/log/btmp : ghi chú tất cả các lần đăng nhập thất bại.

- /var/log/wtmp : bản ghi của mỗi lần đăng nhập / đăng xuất.

- /var/log/lastlog: thông tin về lần đăng nhập cuối cùng cho tất cả người dùng. Tập tin nhị phân này có thể được đọc bằng lệnh "lastlog"

Ngoài ra còn có thư mục /var/run, đây là 1 thư mục được trỏ tới /run, chứa các thông tin về hệ thống hiện tại đang hoạt động

#### Đọc log

Bên dưới là mẫu cảu 1 tệp log

<img src="img/01.png">

Cú pháp chung sẽ là:

- Ngày

- Thời gian chính xác

- Server hostname (tên máy tính)

- Tên dịch vụ / tiến trình

- Thông báo

Để đọc 1 tệp log thì ta có thể dùng bất cứ trình soạn thảo hoặc xem văn bản nào vì chúng đơn giản chỉ là những "clear text", trừ 1 số file log đặc biệt cần sử dụng lệnh riêng

| Command | Ý nghĩa | Ghi chú |
| --- | --- | --- |
| more <đường dẫn file> | đọc nội dung toàn bộ file log | in ra thành từng trang trên màn hình, sử dụng dấu cách để chuyển trang |
| head <đường dẫn file> | in ra nội dung 10 dòng đầu tiên của file log | sử dụng tùy chọn `-n` để in ra số dòng theo yêu cầu (n là 1 số bất kỳ) |
| tail <đường dẫn file> | in ra nội dung 10 dòng cuối cùng của file log | sủ dụng tùy chọn `-n` để in ra số dòng theo yêu cầu |
| tailf <đường dẫn file> | in ra nội dung 10  dòng cuối cùng của file log theo thời gian thực | sử dụng tùy chọn `-n` để in ra số dòng theo yêu cầu |

