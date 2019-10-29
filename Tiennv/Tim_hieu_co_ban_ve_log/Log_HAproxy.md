## Log HAproxy

### Tìm hiểu về HAproxy

#### Tổng quan về HAProxy

HAProxy, viết tắt của High Availability Proxy, là một phần mềm cân bằng tải open source cho TCP/HTTP. Nó có thể chạy trên Linux, Solaris và FreeBSD. Mục đích chính của nó là dùng để cải thiện hiệu năng và tính tin cậy của hệ thống bằng cách dẫn tải đến các server khác. HAProxy cũng được sử dụng tại nhiều các công ty nổi tiếng như GitHub, Imgur, Instagram,...

Tính sẵn sàng cao (HA) là khả năng của một hệ thống hoặc thành phần hệ thống được vận hành liên tục trong một khoảng thời gian dài mong muốn. Tính khả dụng có thể được đo tương đối với "hoạt động 100%" hoặc "không bao giờ thất bại". Trong công nghệ thông tin (CNTT), một tiêu chuẩn sẵn có rộng rãi nhưng khó đạt được cho một hệ thống hoặc sản phẩm được gọi là tính khả dụng "five nines" ( 99,999 phần trăm).

Cân bằng tải (Load Balancing) là một phương pháp phân phối khối lượng tải trên nhiều máy tính hoặc một cụm máy tính để có thể sử dụng tối ưu các nguồn lực, tối đa hóa thông lượng, giảm thời gian đáp ứng và tránh tình trạng quá tải trên máy chủ. Trong điện toán, cân bằng tải cải thiện việc phân phối khối lượng công việc trên nhiều tài nguyên máy tính, chẳng hạn như máy tính, cụm máy tính, liên kết mạng, đơn vị xử lý trung tâm hoặc ổ đĩa. Cân bằng tải nhằm mục đích tối ưu hóa việc sử dụng tài nguyên, tối đa hóa thông lượng, giảm thiểu thời gian phản hồi và tránh quá tải cho bất kỳ tài nguyên nào. Sử dụng nhiều thành phần với cân bằng tải thay vì một thành phần có thể làm tăng độ tin cậy và tính khả dụng thông qua dự phòng. Cân bằng tải thường liên quan đến phần mềm hoặc phần cứng chuyên dụng, chẳng hạn như bộ chuyển mạch đa lớp hoặc một quá trình máy chủ hệ thống tên miền.

#### Các khái niệm trong HAProxy

1. Proxy

Proxy là 1 internet server trung gian làm nhiệm vụ chuyển tiếp, kiểm soát thông tin và tạo sự an toàn cho sự truy cập Internet giữa client và server. Máy cài đặt proxy gọi là proxy server. Proxy hay máy cài đặt proxy có địa chỉ IP và một cổng truy cập cố định. Ví dụ: 123.234.111.222:80. Địa chỉ IP của proxy trong ví dụ là 123.234.111.222 và cổng truy cập là 80.

Khi bạn gửi đi một web request, request đó sẽ tới proxy server trước tiên. Proxy server sau đó sẽ thay bạn thực hiện yêu cầu web, nhận các phản hồi từ web server và chuyển bạn đến trang web dữ liệu để bạn có thể xem trang trong trình duyệt của mình. Khi proxy server chuyển tiếp web request của bạn, server có thể gây ra các thay đổi trong dữ liệu gửi đi mà vẫn cung cấp cho bạn thông tin mà bạn mong muốn. Một proxy server có thể thay đổi địa chỉ IP nên web server sẽ không thể biết được vị trí chính xác của bạn là ở đâu. Proxy server có thể mã hóa dữ liệu và ẩn chúng (không thể đọc được) trong lúc chuyển tiếp. Và cuối cùng, một proxy server có thể chặn truy cập một trang web nhất định dựa trên địa chỉ IP.

- Forward proxy: Là khái niệm proxy chúng ta dùng hàng ngày, nó là thiết bị đứng giữa 1 client và tất cả các server mà client đó muốn truy cập vào.

- Reverse proxy: là 1 proxy ngược, nó đứng giữa 1 server và tất cả các client và server mà server đó phục vụ. Ưu điểm của nó là khả năng quản lý tập trung, giúp chúng ta có thể kiếm soát mọi yêu cầu do client gửi lên server mà chúng ta cần bảo vệ.