## Tìm hiểu về KVM

### Giới thiệu

KVM (Kernel-based Virtual Machine) là 1 module ảo hóa mã nguồn mở được tích hợp vào trong Linux. Cụ thể, nó cho phép bạn biến Linux thành 1 hypervisor cho phép máy chủ chạy nhiều môi trường ảo bị cô lập gọi là máy khách hoặc máy ảo (VM).

Avi Kivity bắt đầu phát triển KVM vào giữa năm 2006 tại Qumranet - 1 startup company về công nghệ, sau đó được Redhat mua lại vào ngày 4 tháng 9 năm 2008 với giá 107 triệu đô la và bắt đầu phát triển, phổ biến KVM Hypervisor. KVM nổi lên vào tháng 10 năm 2006 và được sáp nhập vào dòng chính của Linũ trong phiên bản kernel 2.6.20, được phát hành ngày 5 tháng 2 năm 2007.

KVM là giải pháp ảo hóa cho hệ thống Linux trên nền tảng phần cứng x86 có các module mở rộng hỗ trợ ảo hóa (Intel VT-x hoặc AMD-V), hỗ trợ cho rất nhiều hđh khách bao gồm Windows, Linux, BSD, Solaris, Haiku OS, ReactOS và hđh nghiên cứu AROS. Sử dụng kết hợp với QEMU, KVM có thể chạy Mac OS X.

### KVM hoạt động ntn?

Về bản chất, KVM không thực sự là 1 hypervisor có chức năng giả lập phần cứng để chạy các máy ảo. Chính xác hơn KVM chỉ là 1 module của nhân Linux hỗ trợ cơ chế ánh xạ các chỉ dẫn trên CPU ảo (của guest VM) sang chỉ dẫn trên CPU vật lý (của host chứa VM). Hoặc có thể hình dung KVM giống như 1 driver cho hypervisor để sử dụng được tính năng ảo hóa của các vi xử lý Intel như Intel VT-x hay như của AMD như AMD-V, với mục đích là tăng hiệu suất cho guest VM.

KVM chuyển đổi Linux thành hypervisor loại 1 (bare-metal). Tất cả hypervisor cần 1 số thành phần ở mức độ hđh như trình quản lý bộ nhớ, lập lịch cho các triến trình, input/output stack, driver thiết bị, trình quản lý bảo mật, ngăn xếp mạng ... để chạy các máy ảo. KVM có tất cả các thành phần đó vì nó là 1 phần của nhân Linux. Mọi máy ảo được triển khai như 1 tiến trình Linux thông thường, được lên lịch bởi bộ lập lịch Linux tiêu chuẩn, với phần cứng ảo chuyên dụng như card mạng, graphic adapter, CPU, bộ nhớ, ổ đĩa.

### Kiến trúc của KVM

Trong kiến trúc KVM, máy ảo được triển khai như 1 tiến trình Linux thông thường, được lên lịch bởi bộ lập lịch Linux tiêu chuẩn. Trong thực tế, mỗi CPU ảo xuất hiện như 1 tiến trình Linux thông thường. Điều này cho phép KVM được hưởng lợi từ tất cả các tính năng của nhân Linux.

Cấu trúc tổng quan:

<img src="img/07.png">

Linux có tất cả các cơ chế của 1 VMM cần thiết để vận hành các máy ảo, vì vậy các nhà phát triển không xây dụng lại mà chỉ thêm vào đó 1 vài thành phần để hỗ trợ ảo hóa. KVM được triển khai như 1 module hạt nhân có thể được nạp vào để mở rộng Linux bởi những khả năng này.

Trong môi trường Linux thông thường, mỗi process chạy / sử dụng kernel-mode hoặc user-mode. KVM đưa ra 1 chế độ thứ 3, đó là guest-mode. Nó dựa trên CPU có khả năng ảo hóa với kiến trúc Intel VT hoặc AMD SVM. 1 process trong guest-mode bao gồm cả kernel-mode và user-mode.

<img src="img/08.png">

Trong cấu trúc tổng quan của KVM bao gồm 3 thành phần chính:

<img src="img/09.png">

- KVM kernel module:

	- là 1 phần trong dòng chính của Linux kernel
	
	- cung cấp giao diện chung cho Intel VMX và AMD SVM (thành phần hỗ trợ ảo hóa phần cứng)
	
	- chứa những mô phỏng cho các instructions và CPU modes không được hỗ trợ bởi Intel VMX và AMD SVM

- quemu - kvm: chương trình dòng lệnh để tạo các mây ảo, thường được vận chuyển dưới dạng các package "kvm" hoặc "quemu-kvm". Có 3 chức năng chính:

	- thiết lập VM các các thiết bị I/O
	
	- thực thi mã khách thông qua KVM kernel module
	
	- mô phỏng các thiết bị I/O và di chuyển các guest từ host này sang host khác

- libvirt management stack:

	- cung cấp API để quản lý các tool như virsh để có thể giao tiếp và quản lý các VM
	
	- cung cấp chế độ quản lý từ xa an toàn

### Các tính năng của KVM

- Bảo mật: KVM sử dụng kết hợp SE Linux và secure virtualization (sVirt) để tăng cường khả năg bảo mật và cô lập cho các máy ảo. SE Linux thiết lập ranh giới bảo mật xung quanh máy ảo. sVirt mở rộng khả năng của SE Linux, cho phép bảo mật Kiểm soát truy cập bắt buộc (MAC) được áp dụng cho các guest VM và ngăn ngừa lỗi ghi nhãn thủ công. sVirt đảm bảo máy ảo không thể bị truy cập bởi các tiến trình (hoặc máy ảo) khác, việc này có thể được mở rộng thêm bởi người quản trị hệ thống định nghãi ra các quyền hạn đặc biệt: như là nhóm các máy ảo chia sẻ chung nguồn tài nguyên.

- Lưu trữ: KVM có thể sử dụng bất kỳ bộ lưu trữ nào được Linux hỗ trợ, bao gồm ổ đĩa cục bộ hay hệ thống lưu trữ qua mạng (NAS). Multipath I/O có thể được sử dụng để cải thiện việc lưu trữ và cung cấp dự phòng. KVM cũng hỗ trọ việc chia sê file system nên có thể share image các máy ảo cho nhiều host khác nhau. Disk image trong KVM hỗ trợ thin provisioning, cho phép phân bổ lưu trữ thoe yêu cầu. Định dạng image của KVM là QCOW2, hỗ trợ việc sanpshot nhiều mức, nén và mã hóa dữ liệu.

- Hỗ trợ phần cứng: KVM có thể sử dụng nhiều nền tảng phần cứng được chứng nhận hỗ trợ Linux. Do các nhà cung cấp phần cứng thường xuyên đóng góp cho sự phát triển của kernel, nên các tính năng phần cứng mới cũng thường nhanh chóng được áp dụng.

- Quản lý bộ nhớ: KVM kế thừa các tính năng quản lý bộ nhớ của Linux, bao gồm cả non-uniform memory access cho phép tận dụng hiệu quả bộ nhớ cho các hệ thống đa xử lý và kernel same-page merging cho phép các máy ảo chia sẻ vùng nhớ, các yêu cầu bộ nhớ giống nhau trên các máy ảo có thể được xử lý chung để tiết kiệm bộ nhớ xử lý.

- Live migration: KVM hỗ trợ live migration, đoa là khả năng di chuyển 1 máy ảo đang chạy giữa các máy chủ vật lý mà không bị gián đoạn dịch vụ. Máy ảo vẫn được bật, các kết nối mạng vẫn được duy trì và các ứng dụng vẫn tiếp tục hoạt động trong khi máy ảo được di chuyển. KVM cũng lưu trặng thái hiện tại của máy ảo để có thể lưu trữ và chạy tiếp sau này.

- Hiệu suất và khả năng mở rộng: KVM kế thừa hiệu năng của Linux, mở rộng để phù hợp với yêu cầu tải nếu số lượng máy khách và các yêu cầu tăng lên. KVM cho phép khối lượng công việc ứng dụng đòi hỏi khắt khe nhất được ảo hóa và là cơ sở cho nhiều thiết lập ảo hóa doanh nghiệp, chẳng hạn như trung tâm dữ liệu và đám mây riêng (thông qua Openstack).

<img src="img/06.png">

- Lập kế hoạch và kiểm soát tài nguyên: Trong mô hình KVM, VM là 1 tiến trình Linux, được lên lịch và quản lý bởi kernel. Bộ lập lịch Linux cho phép kiểm soát chi tiết các tài nguyên được phân bổ cho 1 tiến trình Linux và đảm bảo chất lượng cho 1 tiến trình cụ thể. Trong KVM, điều này bao gồm bộ lập lịch hoàn toàn công bằng, các nhóm kiểm soát, không gian tên mạng và tiện ích mở rộng thời gian thực.

- Độ trễ thấp và mức độ ưu tiên cao hơn: Nhân Linux có các phần mở rộng thời gian thực cho phép các ứng dụng dựa trên VM chạy ở độ trễ thấp hơn với mức độ ưu tiên tốt hơn (so với bare-metal). Kernel cũng phân chia các tiến trình đòi hỏi thời gian tính toán dài thành các thành phần nhỏ hơn, sau đó được lên lịch và xử lý tương ứng.

> 1 số lưu ý về KVM và QEMU:
Có thể hình dung KVM như là driver cho hypervisor để sử dụng được virtualization extension của CPU vật lý nhằm tăng hiệu năng cho guest VM.
QEMU là 1 emulator nên nó có bộ dịch của nó là Tiny Code Generator (TCG) để xử lý các yêu cầu trên CPU ảo và giả lập kiến trúc của máy ảo. Nên có thể coi QEMU là 1 hypervisor type 2. Lúc tạo VM bằng QEMU có VirtType là KVM thì khi đó các chỉ dẫn có nghĩa đối với CPU ảo sẽ được QEMU sử dụng KVM để ánh xạ thành các chỉ dẫn có nghĩa đối với CPU vật lý. Làm như vậy sẽ nhanh hơn là chỉ chạy độc lập QEMU, vì nếu không có KVM thì QEMU sẽ phải về sử dụng translator của riêng nó là TCG để chuyển dịch các chỉ dẫn của CPU ảo rồi đem thực thi trên CPU vật lý.
=> Khi QEMU/KVM kết hợp nhau thì tạo thành type-1 hypervisor.
QEMU cần KVM để tăng cường hiệu năng và ngược lại KVM cần QEMU (modified version) để cung cấp giải pháp full virtualization hoàn chỉnh.