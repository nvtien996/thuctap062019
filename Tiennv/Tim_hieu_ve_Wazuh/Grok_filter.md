## Grok filter

Plugin và filter cực kỳ hữu ích để xác định rõ ràng các dữ liệu. Việc giám sát và truy xuất có thể được tinh chỉnh để chỉ chú ý đến các bộ dữ liệu nhất định. Grok được sử dụng để khớp các tham số cụ thể với 1 lượng lớn dữ liệu không có cấu trúc nhất định để chỉ trích xuất những gì cần thiết. Nó cho phép Logstash thiết lập các tiêu chí để theo dõi và phân tích với thiết lập theo mẫu.

Grok là 1 dạng biểu thức chính quy (Regular Expression), được sử dụng để phân tích cấu trúc văn bản, khớp các dòng với các biểu thức thông thường sau đó ánh xạ các phần của văn bản thành các phần và hành động dựa trên ánh xạ.

Grok cung cấp một cách để phân tích dữ liệu nhật ký phi cấu trúc thành một định dạng có thể được truy vấn.

Plugin Grok Filter rất hữu ích để phân tích nhật ký sự kiện và phân chia thông điệp cho nhiều trường. Thay vì tạo các biểu thức thông thường, người dùng sẽ sử dụng các mẫu được xác định trước để phân tích nhật ký.

Grok được áp dụng khá đa dạng. Mỗi kiểu dữ liệu sẽ được khai báo các mẫu pattern, và có thể sử dụng lẫn nhau trong bộ lọc filter