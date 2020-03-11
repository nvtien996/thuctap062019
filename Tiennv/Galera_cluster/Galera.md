## Tìm hiểu về Galera cluster

### 1. Giới thiệu

Maria Galera Cluster là một synchronous multi-master cluster, có nghĩa là tự động đồng bộ trong nội bộ cụm mà node nào cũng có thể coi là master.

Sử dụng galera cluster, application có thể read/write trên bất cứ node nào. Một node có thể được thêm vào cluster cũng như gỡ ra khỏi cluster mà không có downtime về dịch vụ, cách thức cũng đơn giản.

Bản thân các database như mariadb, percona xtradb không có tính năng multi master được tích hợp sẵn bên trong. Các database này sẽ sử dụng một galera replication plugin để sử dụng tính năng multi master do galera cluster cung cấp. Về bản chất, galera replication plugin sử dụng một phiên bản mở rộng của mysql replication api, bản mở rộng này có tên là wsrep api.

Dùng wsrep api, galera cluster sẽ thực hiện được certification based replication, một kỹ thuật cho phép thực hiện multi master trên nhiều node. Một writeset, chính là một transaction cần được replication trên các node. Transaction này sẽ được certificate trên từng node nhận được (qua replication) xem có conflict với bất cứ transaction nào đang có trong queue của node đó không. Nếu có thì replicated writeset này sẽ bị node discard. Nếu không thì replicated writeset này sẽ được applied. Một transaction chỉ xem là commit sau khi đã pass qua bước certificate trên tất cả các node. Điều này đảm bảo transaction đó đã được phân phối trên tất cả các node.

Hướng tiếp cận này gọi là virtual synchronous replication. Thực tế, một transaction sẽ được replicate đến tất cả các node để thực hiện certificate, nhưng sau đó, trong trường hợp pass qua tất cả các certification, quá trình apply writeset trên các node lại bất đồng bộ vì rất nhiều nguyên nhân khác nhau như năng lực xử lý các node khác nhau, do switch context của CPU… Apply writeset (quá trình data thực sự được ghi xuống table) giữa các node bất đồng bộ do đó là hiện tượng tự nhiên.

Galera được hỗ trợ bởi MySQL bản >= 5.5 và MariaDB bản >=5.5

Trong bản MariaDB 5.5 và 10.0, Galera cluster là 1 gói riêng được cài đặt thêm chứ không phải là gói chuẩn của MariaDB server

Trong MariaDB 10.1, 2 gói MariaDB server và MariaDB Galera server được kết hợp với nhau kèm theo những gói phụ thuộc được cài đặt tự động khi cài MariaDB, tuy nhiên Galera chỉ hoạt động khi đã cấu hình bật lên giống như 1 plugin

Kể từ MariaDB 10.3 trở về trước, MariaDB Galera Cluster sử dụng Galera 3, còn từ phiên bản MariaDB 10.4 trở lên, MariaDB Galera Cluster sử dụng Galera 4

MariaDB Galera Cluster là một bản vá của MySQL-wsrep thực hiện bởi Codership và hiện tại chỉ hỗ trợ Linux

Galera hỗ trợ Cloud (như AWS zones) và WAN

Quá trình đồng bộ dữ liệu xảy ra ở thời điểm commit:

- Độ trễ của kế nối mạng có thể làm chậm quá trình commit, nhưng tránh được disk I/O

- Conflicts được phát hiện tại thời điểm commit

Galera cluster sử dụng row-based binary log events (không phải binary log files)

Galera có thể được cài đặt để đồng bộ chỉ một hay vài tables hoặc databases sử dụng những bộ lọc (replication filters).

- 2 bộ lọc được khuyến khích sử dụng trong Galera là **replicate-do-db**, **replicate-ignore-db**, cho cả hai loại engine InnoDB và MyISAM.

- Ngược lại, những bộ lọc sau không được khuyến khích sử dụng: **binlog-do-db**, **binlog-ignore-db**, **replicate-wild-do-db**, **replicate-wild-ignore-db**.

### 2. Chức năng và phương thức đồng bộ

Chức năng của Galera replication được cài đặt như một thư viện chia sẻ và có thể được link với bất kỳ hệ thống xử lý transaction nào nếu hệ thống này cài đặt wsrep API hooks.

Thư viện Galera Replication là một stack giao thức cung cấp chức năng cho việc chuẩn bị, nhân bản và áp dụng các tập ghi transaction. Nó bao gồm các modules:

- wsrep API: xác định giao diện, trách nhiệm cho DBMS và nhà cung cấp replication

- wsrep hooks: là việc tích hợp wsrep bên trong engine DBMS

- Galera provider: cài đặt wsrep API cho thư viện Galera

- Tầng certification: chịu trách nhiệm chuẩn bị tập ghi (write sets) và thực hiện việc chứng nhận

- Replication: quản lý giao thức nhân bản và cung cấp toàn bộ khả năng ordering

- GCS framework: cung cấp kiến trúc plugin cho các hệ thống giao tiếp nhóm. Nhiều cài đặt GCS có thể được thích nghi (adapt), được thử nghiệm với những cài đặt in-house như: vsbes và gemini

Galera sử dụng 3 phương thức để đồng bộ dữ liệu: mysqldump, rsync, xtrabackup-v2

- rsync: là phương thức mặc định và yêu cầu ít thời gian setup nhất. Điểm yếu của nó là node đầu mối (donor node) sẽ bị block với tất cả các hoạt động trong lúc đang thực hiện SST

- mysqldump: chuyển dữ liệu dưới dạng các lệnh SQL INSERT. Mỗi lệnh này sẽ tốn thời gian để thực thi trên các node đang join vào nếu kích thước của tập dữ liệu là đáng kể. Có một vài kỹ thuật để tăng hiệu suất insert, tuy nhiên nó không thể loại bỏ cản trở lớn nhất là dữ liệu được chuyển theo kiểu row-by-row thay vì file-by-file

- xtrabackup-v2: sử dụng công cụ Percona XtraBackup để thu về một snapshot không bị block (non-blocking snapshot) của node đầu mối (donor node) và sau đó sử dụng snapshot đó để khởi động node mới join vào. Điều này yêu cầu cài đặt phức tạp hơn một chút, nhưng node đầu mối hầu như luôn sẵn sàng cho việc query trong suốt quá trình xử lý. Đây là phương thức nên sử dụng để cài đặt trong nếu chúng ta muốn loại bỏ thời gian chết của node đầu mối

Galera sử dụng các cổng sau:

- Cổng 3306 để kết nối với MySQL client và State Snapshot Transfer (SST) nếu sử dụng phương thức mysqldump để đồng bộ

- Cổng 4567 dùng cho replication traffic, multicast. Sử dụng cả 2 giao thức UDP và TCP

- Cổng 4568 dùng cho Incremental State Transfer

- Cổng 4444 dùng cho State Snapshot Transfer nếu sử dụng phương thức rsync để đồng bộ

### 3. 1 số thuật ngữ liên quan đến Galera Cluster

- wsrep: Write Set REPlication

- SST: State Snapshot Transfer

- IST: Incremental State Transfer

- Donor node: một db server (một node) đã tồn tại trong hệ thống và có thể sử dụng như đầu mối để các node mới kết nối vào cluster

- Joining node: một db server (một node) đang kết nối vào cluster

### 4. Lợi ích và hạn chế

- Lợi ích:

	+ Một giải pháp multi master hoàn chỉnh nên cho phép read/write trên node bất kỳ
	
	+ Synchronous replication
	
	+ Multi thread slave cho phép apply writeset nhanh hơn
	
	+ Không cần failover vì node nào cũng là master
	
	+ Automatic node provisioning: Bản thân hệ database đã tự backup cho nhau. Tuy nhiên, khả năng backup tự nhiên của galera cluster không loại trừ được các sự cố do con người gây ra như xóa nhầm data
	
	+ Hỗ trợ innodb
	
	+ Hoàn toàn trong suốt với application nên application không cần sửa đổi gì
	
	+ Không có Single point of failure vì bất cứ node nào trong hệ cluster cũng là master

- Hạn chế:

	+ Không scale up về dung lượng. Một galera cluster có 3 node thì cả 3 node đó cùng có một data giống hệt nhau. Dung lượng lưu trữ của cả cluster sẽ phụ thuộc vào khả năng lưu trữ trên từng node
	
	+ Vẫn có hiện tượng stale data do bất đồng bộ khi apply writeset trên các node
	
	+ binlog_format phải là row. Cũng giống mysql replication, bạn không được thay đổi binlog_format khi hệ thống galera cluster đang chạy vì có thể làm crash toàn bộ cluster. Yêu cầu bắt buộc này thực ra cũng đem lại một hạn chế. Không phải tự dưng, mysql replication khuyến khích người dùng sử dụng mix binlog_format
	
	+ flush privileges command không được replicate
	
	+ chỉ hỗ trợ flush tables with read lock
	
	+ Với engine MyISAM thì chỉ các DDL command CREATE USER mới được replicate
	
	+ Transaction size từ 128K đến tối đa 1G theo mặc định, admin có thể mở rộng giới hạn này
	
	+ Mọi bản ghi phải có primary key (multi column primary key cũng được support), không hỗ trợ delete bản ghi không có primary key, các dòng do không có primary key sẽ có thứ tự khác nhau giữa các node. Đây là yêu cầu cho thiết kế cơ sở dữ liệu
	
	+ Query log phải đổ vào file, không thể đổ vào table được
	
	+ Windows không được hỗ trợ

1 số hạn chế khác xem thêm ở [đây](https://mariadb.com/kb/en/mariadb-galera-cluster-known-limitations/)

- 1 số đề nghị:

	+ Đồng bộ các node dùng xtrabackup-v2
	
	+ Để tránh conflict, sử dụng wsrep_auto_increment_control để galera cluster kiểm soát giá trị auto_increment_increment và auto_increment_offset của mysql
	
	+ Tránh dùng replication filter với galera cluster
	
	+ Một cụm Galera tối thiểu bao gồm 3 node
	
	+ Sử dụng máy tính thật sản phẩm (Không nên dùng máy ảo để làm Maria Galera Cluster)
	
	+ Phần cứng của các note cần giống nhau, mạng kết nối cẩn ổn định và >= 1Gb. Dung lượng ổ cứng cần lớn hơn 3 lần dung lượng CSDL