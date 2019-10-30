## Tổng quan về Wazuh

Wazuh là một hệ thống phát hiện xâm nhập dựa trên máy chủ (HIDS) mã nguồn mở. Nó được sinh ra như một nhánh của OSSEC HIDS, và sau đó được tích hợp với Elastic Stack và OpenSCAP, phát triển thành một giải pháp toàn diện hơn. Wazuh thực hiện phát hiện bảo mật, phân tích nhật ký, cảnh báo dựa trên thời gian và phản hồi tích cực. Nó cung cấp phát hiện xâm nhập cho hầu hết các hệ điều hành, bao gồm Linux, OpenBSD, FreeBSD, OS X, Solaris, và Windows. Wazuh có một kiến trúc đa nền tảng, tập trung cho phép nhiều hệ thống được giám sát và quản lý.

Các tính năng nổi bật của giải pháp này:

- Quản lý và phân tích nhật ký: Các Agent của Wazuh đọc nhật ký ứng dụng và hệ điều hành và chuyển tiếp chúng đến Server để phân tích và lưu trữ dựa trên tập luật.

- Giám sát toàn vẹn tệp: Wazuh giám sát hệ thống tệp, xác định các thay đổi về nội dung, quyền, quyền sở hữu và thuộc tính của các tệp.

- Phát hiện xâm nhập và bất thường: Agent quét hệ thống tìm kiếm phần mềm độc hại, rootkit hoặc dị thường đáng ngờ. Và có thể phát hiện các tệp ẩn, các quy trình bị che giấu hoặc các trình nghe mạng chưa đăng ký.

- Giám sát chính sách và tuân thủ: Wazuh giám sát các tệp cấu hình để đảm bảo chúng tuân thủ các chính sách bảo mật, tiêu chuẩn hoặc hướng dẫn cứng được cài đặt trước. Agent thực hiện quét định kỳ để phát hiện các ứng dụng được biết là dễ bị xâm nhập, chưa được bảo vệ hoặc được cấu hình không an toàn.

Dưới đây là một mô tả ngắn gọn về các công cụ này và những gì họ làm:

<img src="img/01.png">

Wazuh được tích hợp OSSEC, OpenSCAP và Elastic Stack tạo thành một giải pháp thống nhất và toàn diện

<img src="img/02.png">

