## Grok filter

Plugin và filter cực kỳ hữu ích để xác định rõ ràng các dữ liệu. Việc giám sát và truy xuất có thể được tinh chỉnh để chỉ chú ý đến các bộ dữ liệu nhất định. Và Grok là 1 trong những plugin filter được sủ dụng phổ biến nhất trong Logstash.

### Grok là gì

Grok là 1 dạng biểu thức chính quy (Regular Expression), được sử dụng để phân tích cấu trúc văn bản, khớp các dòng với các biểu thức thông thường sau đó ánh xạ các phần của văn bản thành các phần và hành động dựa trên ánh xạ.

Grok cung cấp một cách để phân tích dữ liệu nhật ký phi cấu trúc thành một định dạng có thể được truy vấn.

Plugin Grok Filter rất hữu ích để phân tích nhật ký sự kiện và phân chia thông điệp cho nhiều trường. Thay vì tạo các biểu thức thông thường, người dùng sẽ sử dụng các mẫu được xác định trước để phân tích nhật ký.

Grok được áp dụng khá đa dạng. Mỗi kiểu dữ liệu sẽ được khai báo các mẫu pattern, và có thể sử dụng lẫn nhau trong bộ lọc filter.

### Sử dụng Grok

Grok hoạt động bằng cách kết hợp các pattern để khớp với log của bạn.

Plugin Grok được cài đặt với Logstash theo mặc định, do đó không cần phải cài đặt riêng nó.

Các pattern Grok theo mặc định, bạn có thể tìm thấy chúng ở [đây](https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns)

Hoặc bạn có thêm pattern cảu riêng bạn. Nếu bạn cần pattern để phù hợp với log của bạn, hãy truy cập trang [web](http://grokdebug.herokuapp.com/) hoặc [web](http://grokconstructor.appspot.com/), chúng sẽ khá hữu ích cho bạn.

Cú pháp cho Grok pattern là:

`%{SYNTAX:SEMANTIC}`

`SYNTAX` là tên của pattern sẽ khớp với text của bạn.

`SEMANTIC` là định danh bạn đặt cho phần text được khớp

Ví dụ: `3.45` sẽ được khớp với pattern `NUMBER` và `1.1.1.1` sẽ được khớp với pattern `IP`. Hơn nữa `3.45` có thẻ là thời lượng của 1 sự kiện, vì vậy bạn có gọi nó 1 cách đơn giản `duration`, còn `1.1.1.1` có thể xác định `client` request.

Đối với ví dụ trên, Grok pattern sẽ như sau:

`%{NUMBER:duration} %{IP:client}`

Ngoài ra bạn có thể thêm 1 chiuyển đổi loại dữ liệu cho Grok pattern của mình. Theo mặc định, tất cacr ngữ nghĩa được lưu dưới dạng chuỗi. Nếu bạn muốn chuyển đổi kiểu dữ liệu của 1 ngữ nghĩa, ví dụ thay đổi chuỗi thành 1 số nnguyên thì sau đó kết hợp với kiểu dữ liệu đích. Ví dụ: `%{NUMBER:num:int}`, trong đó chuyển đổi `num` từ 1 chuỗi thành 1 số nguyên. Hiện tại các chuyển đổi được hỗ trợ là `int` và `float`.

### Custom pattern

Đôi khi Logstash không có pattern mà bạn cần, đối với điều này, bạn có 1 vài lựa chọn.

Đầu tiên, bạn có thể sử dụng cú pháp Oniguruma để match 1 đoạn text và lưu nó thành 1 trường

`(?<filed_name_>pattern)`

Ví dụ: nhật ký postfix có `queue id` có giá trị thập lục phân gồm 10 hoặc 11 ký tự. Bạn có thể làm như sau:

`(?<queue_id>[0-9A-F]{10,11})`

Tiếp theo, tạo 1 custom pattern:

- Tạo một thư mục được gọi `patterns` với một tệp trong đó được gọi là `extra` (tên tệp không quan trọng, việc đặt tên tùy thuộc vào bạn)

- Trong tệp đó, hãy viết pattern bạn cần với cú pháp `pattern_name regex`

Ví dụ tạo postfix queue id như trên:

```
# contents of ./patterns/postfix:
POSTFIX_QUEUEID [0-9A-F]{10,11}
```

Sau đó sử dụng option `patterns_dir` trong plugin này để báo cho Logstash biết thư mục custom pattern của bạn ở đâu. Dưới đây là một ví dụ đầy đủ với nhật ký mẫu:

```
filter {
      grok {
        patterns_dir => ["./patterns"]
        match => { "message" => "%{SYSLOGBASE} %{POSTFIX_QUEUEID:queue_id}: %{GREEDYDATA:syslog_message}" }
      }
    }
```

`Jan  1 06:25:43 mailserver14 postfix/cleanup[21403]: BEF25A72965: message-id=<20130101142543.5828399CCAF@mailserver14.example.com>`

Ví dụ log ở trên sẽ khớp và kết quả trong các trường sau:

- `timestamp`: `Jan  1 06:25:43`

- `logsource`: `mailserver14`

- `program`: `postfix/cleanup`

- `pid`: `21403`

- `queue_id`: `BEF25A72965`

- `syslog_message`: `message-id=<20130101142543.5828399CCAF@mailserver14.example.com>`

Một tùy chọn khác là xác định các pattern inline trong filter bằng cách sử dụng `pattern_definitions`. Điều này chủ yếu là để thuận tiện và cho phép người dùng xác định một pattern có thể được sử dụng chỉ trong filter đó. Pattern mới được xác định `pattern_definitions` này sẽ không có sẵn bên ngoài filter grok cụ thể đó

### Grok filter configuration option

| setting | input type | required |
| --- | --- | --- |
| break_on_match | boolean | No |
| keep_empty_captures | boolean | No |
| match | hash | No |
| named_captures_only | boolean | No |
| overwrite | array | No |
| pattern_definitions | hash | No |
| patterns_dir | array | No |
| patterns_files_glob | string | No |
| tag_on_failure | array | No |
| tag_on_timeout | string | No |
| timeout_millis | number | No |