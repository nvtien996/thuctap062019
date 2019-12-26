## Tìm hiểu về Open vSwitch

### Tổng quan Open vSwitch

1. Open vSwitch là gì?

- Giống như Linux Bridge, Open vSwitch là phần mềm cung cấp virtual switch cho các giải pháp ảo hóa network.

- Là phần mềm mã nguồn mở, được cấp phép theo giấy phép Apache 2 nguồn mở, sử dụng cho ảo hoá vswitch trong môi trường ảo hoá của server.

- Phần lớn mã được viết bằng C độc lập với nền tảng và dễ dàng chuyển sang các môi trường khác.

- Open vSwitch có thể forward traffic giữa các máy VM trên cùng một máy chủ vật lý và forward traffic giữa các máy VM và các máy vật lý.

- Open vSwitch được thiết kế tương thích với các switch hiện đại.

- Open vSwitch phù hợp làm việc như là một switch ảo trong môi trường máy ảo VM. Ngoài việc kiểm soát và có khả năng hiển thị giao diện chuẩn cho các lớp mạng ảo, nó được thiết kế để hỗ trợ phân phối trên nhiều máy chủ vật lý. Open vSwitch hỗ trợ nhiều công nghệ ảo hoá Linux-based như là Xen/Xen Server, KVM và Virtual Box.

- Open vSwitch có thể chạy trên các nền tảng Linux, FreeBSD, Windows, non-POSIX embedded Systems,...

2. Các tính năng

Open vSwitch hỗ trợ các tính năng sau:

- Multicast snooping

- Hỗ trợ tính năng VLAN chuẩn 802.IQ với các cổng trunk và access như một switch layer thông thường.

- Hỗ trợ giao diện NIC bonding có hoặc không có LACP trên cổng uplink switch.

- LACP (IEEE 802.1AX-2008)

- IETF tự động đính kèm SPBM và hỗ trợ LLDP thô sơ cần thiết

- Giám sát liên kết BFD và 802.1ag

- STP (IEEE 802.1D-1998) và RSTP (IEEE 802.1D-2004)

- Hỗ trợ cho HFSC qdisc

- Khả năng hiển thị trong giao tiếp giữa các máy ảo thông qua NetFlow, sFlow (R), IPFIX, SPAN, RSPAN và GRE-tunneled mirrors

- Hỗ trợ cấu hình QoS (Quality of Service) và các chính sách thêm vào khác.

- Kiểm soát lưu thông trên mỗi giao diện VM

- Hỗ trợ tạo tunnel GRE, VXLAN, STT và LISP.

- Liên kết NIC với cân bằng tải nguồn-MAC, active backup và L4 hashing

- Hỗ trợ tính năng quản lý các kết nối 802.1aq

- Hỗ trợ OpenFlow các phiên bản từ 1.0 trở lên

- Hỗ trợ IPv6

- Cấu hình cơ sở dữ liệu với C và Python.

- Hoạt động forwarding với hiệu suất cao sử dụng module trong nhân Linux

- Tùy chọn công cụ chuyển tiếp kernel và không gian người dùng

- Multi-table forwarding pipeline với flow-caching engine

- Chuyển tiếp lớp trừu tượng để dễ dàng chuyển sang nền tảng phần mềm và phần cứng mới