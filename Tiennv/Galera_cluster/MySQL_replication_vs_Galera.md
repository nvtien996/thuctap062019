## MySQL replication vs Galera

1 CSDL là trái tim của bất kỳ ứng dụng nào theo đúng nghĩa đen.

Khi mà các ứng dụng trở nên ngày một phổ biến hơn thì việc truy cập đọc/ghi vào các CSDL cũng tăng lên. Do đó, điều này có thể trở thành 1 nút thắt cổ chai khiến cho hiệu suất của ứng dụng bị giảm đi. Và để giải quyết vấn đề này, người ta có các giải pháp như sử dụng MySQL cluster hay Galera, v.v.

### Tại sao phân cụm cơ sở dữ liệu?

Bây giờ chúng ta hãy xem sự cần thiết phải phân cụm cơ sở dữ liệu.

Khi bạn có 1 trang web với số lượng lớn lượt truy cập, số lượng yêu cầu thực sự đến CSDL sẽ cao. Như vậy, tốc độ của trang web sẽ phụ thuộc vào tốc độ của máy chủ có thể nhận được kết quả từ CSDL.

Đồng thời, máy chủ CSDL cũng phải ghi lại dữ liệu vào các bảng, vì vậy nó phải thực hiện các hoạt động đồng thời đọc/ghi. Đây sẽ là nơi cần phải phân cụnm máy chủ CSDL.

Nói cách khác, trong phân cụm CSDL, có 1 nhóm máy chủ để xử lý công việc chứ không phải chỉ là 1 máy chủ. Do đó nó có thể cung cấp dự phòng dữ liệu, cũng như tính cân bằng tải. Ngoài ra điều này còn làm cho ứng dụng web có tính sẵn sàng cao.

Tuy nhiên, không có 1 giải pháp nào phù hợp cho tất cả đối với CSDL clustering. Nó phụ thuộc phần lớn vào loại ứng dụng web, số lượng đọc/ghi vào CSDL, loại nội dung, v.v.

### Điểm giống nhau giữa cả hai giải pháp là gì?

MySQL Group Replication và Galera đều sử dụng các bộ ghi (write set). Tập hợp ghi này là 1 tập hợp các mã định danh toàn cầu (GUID) duy nhất của từng mục logic được thay đổi bởi transaction khi thực hiện (mục có thể là một hàng, một bảng, một đối tượng siêu dữ liệu, v.v.)

Vì vậy, Group Replication và Galera sử dụng các sự kiện nhật ký nhị phân ROW và cùng transaction data, các bản ghi của nó được truyền trực tiếp đồng bộ từ máy chủ nhận được (Master cho transaction cụ thể đó) đến các thành viên/node khác trong cụm.

Sau đó, chúng sẽ xác nhận writeset cục bộ và hàng đợi không đồng bộ các thay đổi được chấp nhận sẽ được áp dụng.

Sau đó, cả hai giải pháp sẽ sử dụng các bộ ghi để kiểm tra xung đột giữa các giao dịch đồng thời thực hiện trên
các bản sao khác nhau. Thủ tục này được đặt tên chứng nhận. Vì vậy, chúng sẽ xác nhận tập hợp ghi cục bộ và hàng đợi không đồng bộ các thay đổi được chấp nhận sẽ được áp dụng.

Cả 2 triển khai đều sử dụng một công cụ giao tiếp nhóm để quản lý quorums, membership, message passing, v.v.

### Điểm khác biệt

Sự khác biệt lớn nhất là Group Replication (GR) là một plugin cho MySQL, được tạo bởi MySQL, được đóng gói và phân phối với MySQL theo mặc định. Ngoài ra, GR có sẵn và được hỗ trợ trên tất cả các Nền tảng MySQL : Linux, Windows, Solaris, OSX, FreeBSD.

Tuy vậy, cũng có nhiều khác biệt như:

- Group Communication:

Galera đang sử dụng lớp hệ thống giao tiếp nhóm độc quyền, thực hiện QoS đồng bộ ảo dựa trên giao thức Đặt hàng vòng đơn Totem.

MySQL Group Replication sử dụng Hệ thống truyền thông nhóm (GCS) dựa trên một biến thể của thuật toán Paxos phổ biến.

- InnoDB:

So với Galera là 1 bản patch MySQL và thêm một lớp bổ sung để có thể "kill" local transaction khi có xung đột

Group Replication sử dụng High Priority Transactions trong InnoDB, cho phép nó để đảm bảo phát hiện và xử lý xung đột đúng cách.

- Binary Log:

Ngay cả khi được yêu cầu binlog_format=ROW, Galera không cần phải bật nhật ký nhị phân

Galera sử dụng một tệp bổ sung có tên gcache (Galera Cache) để ghi các event. Dữ liệu được lưu trữ bên trong tệp này không thể được sử dụng cho bất kỳ mục đích nào khác ngoài IST (Incremental State Transfer). Tệp Galera Cache được sử dụng để lưu trữ các tập tin theo kiểu bộ đệm tròn và có kích thước được xác định trước.

Group Replication sử dụng các tệp nhật ký nhị phân cho mục đích đó. Vì vậy, nếu 1 node bị ngắt trong một thời gian ngắn, nó sẽ thực hiện đồng bộ hóa từ nhật ký nhị phân của donor node. Đây được gọi là IST trong Galera (từ gcache khi có dữ liệu) và Automated Distributed Recovery trong GR.

- GTID

Galera có Global Transaction ID của riêng nó, đã tồn tại từ MySQL 5.5 và độc lập với tính năng GTID của MySQL được giới thiệu trong MySQL 5.6. Nếu GTID của MySQL được bật trên cụm dựa trên Galera, cả hai số đều tồn tại với các chuỗi và UUID riêng.

Giống như Galera, GR có một UUID được quy cho cụm. Sự khác biệt với Galera là ngay cả khi tất cả các node trong cùng 1 Group chia sẻ cùng 1 UUID, trong GR, chúng có phạm vi số thứ tự riêng (được xác định bởi group_Vplication_gtid_ass que_block_size ).

- Master-Master, Master-Slave

Galera luôn luôn là multi-master theo mặc định, vì vậy việc bạn ghi vào node nào không quan trọng.

Group Replication, chỉ có 1 node là primary (master) và các node khác được đưa vào chế độ chỉ đọc tự động.

Đây là lý do tại sao theo mặc định, trong MySQL Group Replication, cluster chạy ở Single Primary Mode (được điều khiển bởi group_Vplication_single_primary_mode). Điều này có nghĩa là chính nó sẽ tự động bầu ra 1 node master (trong trường hợp thất bại của node trước)

- Replication: Majority vs. All

Galera cung cấp đồng bộ các write transaction cho TẤT CẢ các node trong cụm.

Tuy nhiên, Group Replication chỉ cần phần lớn các node xác nhận transaction. Trong ví dụ về cụm gồm 3 node, nếu 1 node gặp sự cố hoặc mất kết nối mạng, 2 node còn lại tiếp tục chấp nhận ghi (hoặc chỉ node chính trong chế độ Single-Primary mode) ngay cả trước khi node bị lỗi bị xóa khỏi cụm.

Tuy nhiên quy tắc đa số trên mạng trong Group Replication không có nghĩa là nó sẽ đảm bảo rằng bạn sẽ không mất bất kỳ dữ liệu nào nếu các đa số node bị mất.

Trong Galera, một sự gián đoạn mạng với 1 node khiến những node khác chờ đợi nó và việc ghi chờ xử lý có thể được thực hiện sau khi kết nối được khôi phục hoặc node bị lỗi bị xóa khỏi cụm sau khi hết thời gian. Vì vậy, khả năng mất dữ liệu trong một kịch bản tương tự là thấp hơn, vì các transaction luôn đạt đến tất cả các node.

- Hỗ trợ mạng WAN:

MySQL Group Replication không khuyến khích cho việc sử dụng mạng WAN. Nó được thiết kế để được triển khai trong môi trường nơi mà các máy chủ rất gần nhau. Hiệu suất và sự ổn định có thể bị ảnh hưởng bởi cả độ trễ mạng và băng thông mạng. Giao tiếp hai chiều phải được duy trì mọi lúc giữa tất cả các node.

Những người sáng lập Galera thực sự khuyến khích dùng thử nó trong các môi trường phân tán địa lý và một số cài đặt dành riêng cho mạng WAN có sẵn (quan trọng nhất là các phân đoạn WAN).

Nhưng cả hai công nghệ đều cần một mạng lưới đáng tin cậy để có hiệu suất tốt.

- State Transfers:

Galera có 2 loại state transfer cho phép đồng bộ dữ liệu với các node khi cần: tăng dần (IST) và đầy đủ (SST). IST được sử dụng khi 1 node đã ra khỏi cụm trong một thời gian và một khi nó kết nối lại với các node khác có các bộ ghi bị thiếu vẫn còn trong bộ đệm Galera. SST hữu ích nếu không thể sử dụng IST, đặc biệt là khi 1 node mới được thêm vào cụm. SST tự động cung cấp cho node với dữ liệu mới được chụp dưới dạng snapshot từ 1 trong các node đang chạy (donor node). Phương pháp SST phổ biến nhất là sử dụng Percona XtraBackup - phương thức snapshot nhanh chóng, non-blocking binary data (hot backup).

Trong Group Replication, state transfer hoàn toàn dựa trên nhật ký nhị phân với các vị trí GTID. Nó yêu cầu 1 donor node với tất cả binary log (bao gồm các bản ghi cho các nút mới).

- Auto Increment Settings:

Galera điều chỉnh các giá trị auto_increment_increment và auto_increment_offset theo số lượng thành viên trong một cụm. Vì vậy, đối với cụm 3 nút, auto_increment_increment sẽ là 3 và auto_increment_offset từ 1 đến 3 (tùy thuộc vào số node). Nếu số lượng node thay đổi sau đó, chúng được cập nhật ngay lập tức. Tính năng này có thể bị vô hiệu hóa bằng cách sử dụng cài đặt wsrep_auto_increment_control. Nếu cần, các cài đặt này có thể được đặt bằng tay.

Trong Group Replication, auto_increment_increment dường như được sửa ở mức 7 và chỉ auto_increment_offset được đặt khác nhau trên mỗi node. Trường hợp trong chế độ Single-Primary mode cũng như vậy.

- Flow Control:

Cả hai công nghệ đều sử dụng một kỹ thuật để điều tiết ghi khi các nút chậm trong việc áp dụng chúng. Tuy nhiên, kích thước mặc định của hàng đợi cho phép trong cả 2 lại khác nhau:

gcs.fc_limit (Galera) = 16 (giới hạn được tăng tự động dựa trên số lượng các node, tức là đến 28 trong cụm 3 node)

group_replication_flow_control_applier_threshold (Group Replication) = 25000.

- Network Hiccup/Partition Handling:

Trong Galera, khi kết nối mạng giữa các node bị gián đoạn, những node còn lại sẽ tạo thành một chế độ xem cụm mới. Những node bị mất kết nối tiếp tục cố gắng kết nối lại với thành phần chính. Khi kết nối được khôi phục, các node được tách sẽ đồng bộ hóa trở lại bằng IST và tự động nối lại cụm.

Với Group Replication, các node sẽ bị trục xuất khỏi cụm và sẽ không tự động tham gia lại sau khi kết nối mạng được khôi phục. Nó chỉ có thể được join lại 1 cách thủ công.

### Tổng kết

Cụm MySQL chứa các node dữ liệu lưu trữ dữ liệu cụm và node quản lý lưu trữ cấu hình của cụm. Ở đây, các máy khách MySQL trước tiên giao tiếp với node quản lý và sau đó kết nối trực tiếp với các node dữ liệu này.

Để đồng bộ hóa dữ liệu trong các node dữ liệu, cụm MySQL sử dụng một công cụ dữ liệu đặc biệt gọi là NDB (Network Database). Do đó, trong MySQL Cluster thường không có sự sao chép dữ liệu. Nó chỉ có đồng bộ hóa nút dữ liệu. Một lần nữa, nó sử dụng tự động shrading hay còn gọi là tách cơ sở dữ liệu lớn thành các đơn vị nhỏ.

Tương tự, MySQL Cluster hoạt động trong môi trường không chia sẻ. Kết quả là, không có 2 thành phần của cụm sẽ chia sẻ cùng một phần cứng. Cụm sẽ hoạt động đầy đủ khi có ít nhất 1 node trên mỗi node group dữ liệu. Do đó, cụm MySQL tránh được lỗi single point failure và đảm bảo tính khả dụng 99,99%.

Đến với kịch bản thời gian thực, cụm MySQL có thể cung cấp thời gian phản hồi thấp tới dưới 3 ms.

Tiếp tục, chúng ta hãy nhìn vào cụm Galera.

Nói một cách đơn giản, Galera Cluster bao gồm một máy chủ cơ sở dữ liệu và sử dụng Galera Replication Plugin để quản lý sao chép. Nó không có gì khác ngoài 1 cụm cơ sở dữ liệu multi-master hỗ trợ sao chép đồng bộ. Kết quả là, nó cung cấp nhiều bản sao dữ liệu cập nhật. Vì vậy, nó trở nên thực sự hữu ích trong các tình huống có nhu cầu chuyển đổi dự phòng ngay lập tức.

Cụm Galera cho phép đọc và ghi dữ liệu trong bất kỳ nút nào. Một lần nữa, các lợi ích tiêu biểu khác của cụm Galera bao gồm tính nhất quán ghi được đảm bảo, cung cấp node tự động, v.v.

May mắn thay, trong cụm Galera, khi mất kết nối mạng giữa các node, những node vẫn có quyền truy cập sẽ tạo thành chế độ xem cụm mới. Và, những node bị mất tiếp tục cố gắng kết nối lại với thành phần chính. Khi khôi phục kết nối, các node được tách sẽ tự động đồng bộ trở lại và tham gia lại cụm tự động. Điều này trở nên thực sự hữu ích khi bạn có máy chủ ở nhiều vị trí địa lý khác nhau.

Ngoài ra, khá dễ dàng để mở rộng quy mô cụm Galera bằng cách thêm các node. Hơn nữa, quá trình theo dõi trạng thái cụm cũng đơn giản. Do đó, không cần phải có node quản lý như cụm MySQL.

Tuy nhiên, Galera đi kèm với sự hỗ trợ MyISAM hạn chế. Nhưng, nó cho kết quả tốt nhất với công cụ lưu trữ InnoDB.

Nói tóm lại, phân cụm cơ sở dữ liệu bằng cách sử dụng cụm MySQL hoặc thông qua Galera có những lợi thế riêng. Và, sự lựa chọn thực sự phụ thuộc vào kịch bản sử dụng chính xác.

> Bạn có thể tham khảo Benchmarking NDB Vs Galera ở [đây](https://blog.pythian.com/benchmarking-ndb-vs-galera/)