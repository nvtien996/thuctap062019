## Tổng quan về Templete và Snapshot trong KVM

### 1. Template

- Template là 1 dạng file image pre-configured của hđh được dùng để tạo nhanh các máy ảo.

- 1 ví dụ được đặt ra là giả sử rằng bạn có 4 máy để làm web server. Thông thường , bạn sẽ phải cài 4 máy ảo rồi lần lượt cài đặt hđh cho từng máy, sau đó lại tiếp tục cài đặt các dịch vụ cũng như phần mềm. Điều này hiển nhiên là tốn rất nhiều thời gian và template sẽ giúp bạn giải quyết vấn đề này.

- Hình ảnh bên dưới đây mô tả các bước theo ví dụ trên nếu bạn cài bằng tay. Các bước từ 2-5 chỉ là những tasks lặp đi lặp lại và sẽ tiêu tốn rất nhiều thời gian không cần thiết.

<img src="img/113.png">

- Với việc sử dụng template, số bước cần thực hiện sẽ được rút ngắn đi rất nhiều, chỉ cần thực hiện 1 lần các bước từ 1-5. Sau đó sử dụng template là đã có thể triển khai 4 web server còn lại 1 cách rất dễ dàng. Điều này giúp người dùng tiết kiệm rất nhiều thời gian.

<img src="img/114.png">

### 2. Snapshot

- Snapshotlà trạng thái của hệ thống ở 1 thời điểm nhất định, nó sẽ lưu lại cả những cài đặt và dữ liệu. Với snapshot, bạn có thể quay trở lại trạng thái của máy ảo ở 1 thời điểm nào đó rất dễ dàng.

- libvirt hỗ trợ việc tạo snapshot khi máy ảo đang chạy. Mặc dù vậy, tốt hơn hết là nên shutdown hoặc suspend trước khi tiến hành tạo snapshot.

- Có 2 loại snapshot chính được hỗ trợ bởi libvirt:

	- Internal: Trước và sau khi tạo snapshot, dữ liệu chỉ được lưu trên 1 ở đĩa duy nhất. Người dùng có thể tạo internal snapshot bằng công cụ virt-manager. Mặc dù vậy, nó vẫn có 1 vài hạn chế
	
		- chỉ hỗ trợ duy nhất định dạng qcow2
		
		- VM sẽ bị ngưng lại khi tiến hành snapshot
		
		- không hoạt động với LVM storage pools
	
	- External: Dựa theo cơ chế copy-on-write. Khi snapshot được tạo, ổ đĩa ban đầu sẽ có trạng thái “read-only” và có một ổ đĩa khác chồng lên để lưu dữ liệu mới:
	
		- Ổ đĩa được chồng lên được tạo ra có định dạng qcow2, hoàn toàn trống và nó có thể chứa lượng dữ diệu giống như ổ đĩa ban đầu. External snapshot có thể được tạo với bất kì định dạng ổ đĩa nào mà libvirt hỗ trợ. Tuy nhiên không có công cụ đồ họa nào hỗ trợ cho việc này.

	<img src="img/114.png">

### 3. Tạo và quản lý template

- Hai khái niệm mà người dùng cần phân biệt đó là `clone` và `template`. Nếu `clone` đơn thuần chỉ là một bản sao của máy ảo thì `template` được coi là master copy của VM, nó có thể được dùng để tạo ra rất nhiều `clone` khác nữa.

- Có hai phương thức để triển khai máy ảo từ `template` đó là `Thin` và `Clone`