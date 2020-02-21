## Nagvis

<img src="img/267.png">

Ở đây ta có 1 ví dụ về network map. NagVis là một addon trực quan hóa cho hệ thống quản lý mạng nổi tiếng Nagios® (và Icinga là một nhánh của Nagios). NagVis chịu trách nhiệm cho các bản đồ như vậy. Tiện ích mở rộng Nagios cho phép trực quan hóa dữ liệu của Nagios, ví dụ như cơ sở hạ tầng, máy chủ, các port hoặc tiến trình.

Sử dụng nagvis giúp chúng ta có thể vẽ được mối liên hệ giữa các thiết bị trong mạng và theo dõi được hiện trạng của mạng cũng như từng services. Sử dụng dữ liệu được cung cấp bởi 1 backend, nó sẽ cập nhật các đối tượng được đặt trên bản đồ theo các khoảng nhất định để phản ánh trạng thái hiện tại. Những bản đồ này cho phép sắp xếp các đối tượng để hiển thị chúng theo các bố cục khác nhau:

- vật lý (ví dụ: tất cả các máy chủ trên tủ rack / trong phòng)

- logic (ví dụ: tất cả các máy chủ ứng dụng)

- địa lý (ví dụ: tất cả host trong vùng địa lý)

- business processes (ví dụ: tất cả máy chủ / dịch vụ liên quan đến một quy trình)

Nói chung, NagVis là một công cụ trình bày thông tin được Nagios thu thập và chuyển bằng cách sử dụng backend.

Các backend được hỗ trợ là:

- mklivestatus (mặc định kể từ NagVis 1.5)

- NDOUtils / IDOUtils (yêu cầu MySQL)

- merlin (yêu cầu MySQL)

Backend lấy thông tin từ quy trình Nagios (mklivestatus) hoặc từ cơ sở dữ liệu (NDOUtils / IDOUtils, merlin).

Bạn có thể đặt tất cả các đối tượng từ Nagios (Máy chủ, Dịch vụ, Nhóm máy chủ, Nhóm dịch vụ) trên bản đồ. Mỗi bản đồ có thể được cấu hình thông qua tập tin cấu hình riêng của nó. Bạn có thể chỉnh sửa các tệp cấu hình trực tiếp bằng cách sử dụng trình soạn thảo văn bản hoặc các cơ chế cấu hình web. Hơn nữa, bạn có thể thêm một số đối tượng NagVis đặc biệt vào bản đồ. Các đối tượng này là hình dạng, hộp văn bản và đối tượng tham chiếu cho các bản đồ khác.

Mỗi đối tượng trên bản đồ của bạn có thể được cấu hình để phù hợp với nhu cầu của bạn. Ví dụ, có các liên kết đến giao diện Nagios trên mỗi đối tượng đại diện cho một đối tượng Nagios. Bạn có thể dễ dàng tùy chỉnh các liên kết này.

Có một menu di chuột được bật theo mặc định. Menu này hiển thị thông tin chi tiết cho từng đối tượng. Nó có thể dễ dàng được sửa đổi bằng cách thay đổi các mẫu template. Bạn cũng có thể vô hiệu hóa menu di chuột.

Theo mặc định, trạng thái của các đối tượng được hiển thị bằng các biểu tượng trên bản đồ. Bạn có thể thay đổi các biểu tượng này bằng cách thêm các biểu tượng từ trang chủ NagVis hoặc tạo biểu tượng của riêng bạn. Trạng thái của các đối tượng cũng có thể được hiển thị dưới dạng đường hoặc dưới dạng tiện ích.

### Thiết lập NagVis

NagVis được tích hợp đầy đủ trong Checkmk và được định cấu hình để bạn có thể ngay lập tức bắt đầu thêm các yếu tố từ giám sát của mình vào bản đồ.

Để khởi động NagVis, trước tiên tại phần Sidebar-Snapins hãy nhấn nút ![](https://checkmk.com/bilder/button_sidebar_addsnapin_lo.png) ở phía dưới bên trái. Chọn snapin NAGVIS MAPS, kéo nó vào side-bar phía bên trái, và cuối cùng khởi động NagVis bằng nút ![](https://checkmk.com/bilder/button_view_snapin_edit.png).

Ngoài ra, một cách khác để sử dụng nagvis là truy cập vào đường dẫn ip_server_check_mk/site/nagvis

Trong đó:

ip_server_check_mk: là địa chỉ của máy server

site: là tên site mà chúng ta tạo trên OMD

Giao diện nagvis

<img src="img/268.png">

Chọn `Options`, sau đó chọn `Manage Maps`

<img src="img/269.png">

Sau đó hệ thống đưa ra 1 bảng để tạo một map mới, điền các thông tin cần thiết rồi nhấp chọn `Create`

<img src="img/270.png">

ở phần `Map Type` có tổng cộng 5 loại bản đồ khác nhau: regular, dynamic, automatic và interactive/non-interactive geographic maps

*Regular Map* là loại bản đồ tiêu chuẩn. Các bản đồ có thể thể hiện bất kỳ kịch bản mong muốn nào - từ switch port, đến phòng máy chủ, đến cơ sở hạ tầng hoàn chỉnh. Các yếu tố (Biểu tượng, Lines, v.v.) được thêm riêng vào bản đồ từ inventory của các máy chủ và dịch vụ của Checkmk.

*Dynamic Maps* phần lớn giống với *Regular Map*, tuy nhiên có một sự khác biệt đáng kể: máy chủ, dịch vụ, Máy chủ lưu trữ và nhóm dịch vụ không được chỉ định rõ ràng mà thay vào đó bằng cách sử dụng các bộ lọc ở dạng biểu thức chính quy, để nói chính xác hơn, dưới dạng Livestatus-Filter-Definitions hợp lệ. Theo cách này, các máy chủ và dịch vụ mới, và tương tự, mọi thay đổi sẽ tự động trên bản đồ mà không cần thêm hành động thủ công.

*Automaps based on parent/child relations* bạn có thẻ xem ảnh ở phần đầu. Đây là các bản đồ cấu trúc mạng được tạo hoàn toàn tự động từ Parent-Child-Relationships được xác định trong Checkmk.

*Geographical Maps* à các bản đồ được hiển thị bằng các tài liệu bản đồ từ Open-Street-Map(OSM)-Projekt cho nền của chúng.