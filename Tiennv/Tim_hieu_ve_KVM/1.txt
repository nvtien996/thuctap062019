## Migrate máy ảo trên KVM

### 1. Tổng quan

<img src="img/116.png">

- Migration là quá trình di chuyển máy ảo từ host vật lí này sang một host vật lí khác. Migration được sinh ra để làm nhiệm vụ bảo trì nâng cấp hệ thống. Ngày nay tính năng này đã được phát triển để thực hiện nhiều tác vụ hơn:

	- Cân bằng tải: Di chuyển VMs tới các host khác kh phát hiện host đang chạy có dấu hiệu quá tải.
	
	- Bảo trì, nâng cấp hệ thống: Di chuyển các VMs ra khỏi host trước khi tắt nó đi.
	
	- Khôi phục lại máy ảo khi host gặp lỗi: Restart máy ảo trên một host khác.
	
	- Trong OpenStack, việc migrate được thực hiện giữa các node compute với nhau hoặc giữa các project trên cùng 1 node compute.

### 2. Cơ chế migrate máy ảo

- Việc migrate có 2 cơ chế:

	- Offline Migrate: là cơ chế cần phải tắt guest trước khi thực hiện việc di chuyển image và file xml của guest sang host khác.

	- Live Migrate: là cơ chế di chuyển guest khi guest vẫn còn đang hoạt động, quá trình di chuyển rất nhanh và trong suốt với người dùng.

- Cơ chế cơ bản của live-migrate: Về cơ bản cơ chế di chuyển vm khi vm vẫn đang hoạt động. Quá trình trao đổi diễn ra nhanh các phiên làm việc kết nối hầu như không cảm nhận được sự gián đoạn nào. Quá trình Live Migrate được diễn ra như sau:

	- Bước đầu tiên của quá trình Live Migrate 1 snapshot ban đầu của VM cần chuyển trên host KVM1 được chuyển sang VM trên host KVM2.

	- Trong trường hợp người dùng đang truy cập VM tại host KVM1 thì những sự thay đổi và hoạt động trên host KVM1 vẫn diễn ra bình thường, tuy nhiên những thay đổi này sẽ không được ghi nhận.

	- Những thay đổi của VM trên host KVM1 được đồng bộ liên tục đến host KVM2.

	- Khi đã đồng bộ xong thì VM trên host KVM1 sẽ offline và các phiên truy cập trên host KVM1 được chuyển sang host KVM2.

### 3. Chuẩn bị

Host kvm1 cài đặt KVM đã tạo 1 máy ảo, IP 100.100.100.11

Host kvm2 cài đặt KVM chưa tạo máy ảo để làm host đích, IP 100.100.100.12

Đều dùng CentOS 7

### 4. Offline migrate

- Để Migrate Offline thì đầu tiên phải shutdown VM trước:

`virsh shutdown VMname`

Sau khi đã shutdown VM, sử dụng câu lệnh sau để migrate máy ảo sang host 100.100.100.11:

`virsh migrate --offline VMname qemu+ssh://100.100.100.11/system`