## Tổng quan về Template và Snapshot trong KVM

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

- Hai khái niệm mà người dùng cần phân biệt đó là `clone` và `template`. Nếu `clone` đơn thuần chỉ là một bản sao của máy ảo thì `template` được coi là master copy của VM, nó có thể được dùng để tạo ra rất nhiều `clone` khác nữa. Template là bản sao chính của máy ảo thường bao gồm HĐH khách, bộ ứng dụng và cấu hình VM cụ thể. Các template được sử dụng khi cần triển khai nhiều máy ảo và đảm bảo rằng chúng phù hợp và được chuẩn hóa.

- Có hai phương thức để triển khai máy ảo từ `template` đó là `Thin` và `Clone`

	- Thin: Máy ảo được tạo ra theo phương thức này sẽ sử dụng template như một base image, lúc này nó sẽ được chuyển sang trạng thái read only. Cùng với đó, sẽ có một ổ mới hỗ trợ "copy on write" được thêm vào để lưu dữ liệu mới. Phương thức này tốn ít dung lượng hơn tuy nhiên các VM được ra sẽ phụ thuộc vào base image, chúng sẽ không chạy được nếu không có base image.
	
	- Clone: Máy ảo được tạo ra là một bản sao hoàn chỉnh và hoàn toàn không phụ thuộc vào template cũng như máy ảo ban đầu. Tuy nhiên, nó sẽ chiếm dung lượng trên ổ đĩa giống như máy ảo ban đầu.

- Template thực chất là máy ảo được chuyển đổi sang. Quá trình này gồm 3 bước:

	- Bước 1: Cài đặt máy ảo với các phần mềm cần thiết để biến nó thành template.
	
	- Bước 2: Loại bỏ những cài đặt như password SSH, địa chỉ MAC,... để đảm bảo rằng nó sẽ không được áp dụng vào các máy ảo được tạo ra từ template này.
	
	- Bước 3: Đánh dấu máy ảo là template bằng việc đổi tên.

### 4. Các bước cụ thể tạo template với máy ảo CentOS 7

- Cài đặt Cen 7 trên KVM, triển khai những dịch vụ cần thiết

- Shutdown máy ảo với câu lệnh `virsh shutdown VMname`

- Sử dụng công cụ `virt-sysprep` để "niêm phong" máy ảo:

    - Cần cài đặt gói `libguestfs-tools-c` để có thể sử dụng được công cụ này. Nó được sử dụng để loại bỏ những thông tin cụ thể của hệ thống đồng thời niêm phong và biến máy ảo trở thành template.

    - Ở đây ta có thể sử dụng 2 tùy chọn để dùng `virt-sysprep` đó là `-a` và `-d`. Tuỳ chọn `-d` được sử dụng với tên hoặc UUID của máy ảo, tuỳ chọn `-a` được sử dụng với đường dẫn tới ổ đĩa máy ảo.

- Bây giờ ta có thể đánh dấu máy ảo trở thành template. Người dùng cũng có thể backp file XML và tiến hành "undefine" máy ảo trong libvirt.

- Sử dụng `virt-manager` để thay đổi tên máy ảo, đối với việc backup file XML, sử dụng câu lệnh: `virsh dumpxml Template_VMname > /root/Template_VMname.xml`

- Để undefine máy ảo, sử dụng câu lệnh `virsh undefine VMname`




