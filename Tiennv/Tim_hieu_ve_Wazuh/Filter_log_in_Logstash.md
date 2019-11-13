## Filter log in Logstash

Trong bộ công cụ ELK, Logstash xử lý nhiệm vụ nặng về tài nghuyên là tổng hợp và xử lý nhật ký log. Công việc xử lý được thực hiện bởi Logstash đảm bảo các thông điệp log được phân tích cú pháp và cấu trúc chính xác, và chính cấu trúc này cho phép bạn phân tích và trực quan hóa dữ liệu dễ dàng hơn khi lập chỉ mục trong Elasticsearch. Việc xử lý chính xác được thực hiện dựa trên dữ liệu được xác định bởi bạn trong phần bộ lọc của tệp cấu hình Logstash. Trong phần này, bạn có thể chọn mộ số các plugin bộ lọc Logstash được hỗ trợ để xac định chính xác cách chuyển đổi nhật ký. Plugin bộ lọc được sử dụng phổ biến nhất là Grok, nhưng cũng có 1 số plugin cực kỳ hữu ích khác mà bạn có thể sử dụng.

1. Grok

Như đã đề cập ở trên, Grok là plugin bộ lọc được sử dụng phổ biến nhất trong Logstash. Mặc dù thực tế là nó không dễ sử dụng nhưng Grok rất phổ biến vì những gì nó cho phép bạn làm là đưa ra cấu trúc cho các bản ghi không có cấu trúc.

Lấy thông điệp log sau làm ví dụ:

`2016-07-11T23:56:42.000+00:00 INFO [MySecretApp.com.Transaction.Manager]:Starting transaction for session-464410bf-37bf-475a-afc0-498e0199f008`

Grok pattern chúng ta sẽ sử dụng:

```
filter {
 grok {
   match => { "message" =>"%{TIMESTAMP_ISO8601:timestamp} 
%{LOGLEVEL:log-level} \[%{DATA:class}\]:%{GREEDYDATA:message}" }
        }
}
```

Sau khi xử lý, thông điệp log sẽ được phân tích cú pháp như sau:

```
{
 "timestamp" => "2016-07-11T23:56:42.000+00:00",
 "log-level" =>"INFO",
 "class" =>"MySecretApp.com.Transaction.Manager"
 "message" => "Starting transaction for session -464410bf-37bf-475a-afc0-498e0199f008"
 }
```

Đây là cách Elaticsearch lập chỉ mục thông điệp log. Được sắp xếp theo định dạng này, thông điệp log đã được chia thành các trường có tên logic, sau đó có thể được truy vấn, phân tích và hiển thị dễ dàng hơn.

2. Mutate

1 filter plugin Logstash phổ biến khác là Mutate. Đúng như tên gọi của nó, bộ lọc này cho phép bạn tác động lên thông điệp log bằng cách biến đổi các lĩnh vực khác nhau. Ví dụ bạn có thể sử dụng bộ lọc để thay đổi các trường, nối chúng lại với nhau, đổi tên chúng và còn hơn thế nữa.

Sử dụng nhật ký ở trê làm ví dụ, sử dụng tùy chọn cấu hình "lowercase" cho mutate plugin. chúng ta có thể chuyển đổi trường "log-level" thành chữ thường:

```
filter {
 grok {...}
 mutate {
   lowercase => [ "log-level" ]
  }
 }
```

Plugin Mutate là 1 cách tuyệt vời để thay đổi dịnh dạng nhật ký của bạn. 1 danh sách đầy đủ các tùy chọn cấu hình khác nhau cho plugin được liệt kê ở [đây](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html).

3. Date

Làm thế nào bạn có thể phân tích các bản ghi và sự kiện nếu chúng không được sắp xếp chính xác theo thứ tự thời gian?

"Date" filter plugin có thể được sử dụng để lấy dấu thời gian và ngày từ 1 thông điệp log và xác định nó là trường timestamp (@timestamp) cho nhật ký. Sau khi được xác định, trường timestamp này sẽ sắp xếp các bản ghi theo thứ tự thời gian chính xác và giúp bạn phân tích chúng hiệu quả hơn.

Có nhiều cách mà thời gian có thể được dịnh dạng trong nhật ký.

Dưới đây là 1 ví dụ về nhật ký truy cập Apache:

`200.183.100.141 - - [25/Nov/2016:16:17:10 +0000] "GET /wp-content/force-download.php?file=../wp-config.php HTTP/1.0" 200 3842 "https://hack3r.com/top_online_shops" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; YTB720; GTB7.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)"`

Sử dụng plugin date filter như sau, chúng ta có thể trích xuất mẫu ngày và thời gian và xác định nó là trường @timestamp theo đó tất cả các bản ghi sẽ được sắp xếp theo:

```
filter {
 
   grok {
 
      match => { "message" => "%{COMBINEDAPACHELOG}"}
      
      }
   
 
   date {
        match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
        target => "@timestamp"
     }
 }
```

Điều quan trọng cần lưu ý là nếu bạn không sử dụng date filter, Logstash sẽ tự động đặt dấu thời gian dựa trên thời gian nhập.

4. JSON

JSON là 1 định dạng cực kỳ phổ biến cho các bản ghi vì nó cho phép người dùng viết các thông diệp có cấu trúc và tiêu chuẩn hóa, có thể dễ dàng đọc và phân tích.

Để duy trì cấu trúc JSON của toàn bộ thông điệp hoặc trường cụ thể, plugin "json" filter cho phép bạn trích xuất và duy trì cấu trúc dữ liệu json trong thông điệp log.

Ví dụ dưới đây là nhật ký truy cập Apache được minh họa dưới dạng JSON:

```
{ "time":"[30/Jul/2017:17:21:45 +0000]", 
"remoteIP":"192.168.2.1", "host":"my.host.local",
 
"request":"/index.html", "query":"", "method":"GET", 
"status":"200",
 
"userAgent":"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; 
Trident/4.0; YTB720; GTB7.2; .NET CLR 1.1.4322; .NET CLR 
2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)", 
"referer":"-" }
```

Thay vì có nhật ký được làm thành 1 dòng, chúng ta có thể sử dụng "json" filter để giữ lại cấu trúc dữ liệu:

```
filter {
 json { source =>"message"
        target => "log"
   }
  }
 }
```

Tùy chọn cấu hình nguồn xác định trường nào trong nhật ký là json mà bạn muốn phân tích. Trong ví dụ này, toàn bộ trường thông báo là json.

5. KV

Các cặp khóa giá trị (key value pair - KVP) là 1 định dạng ghi nhật ký phổ biến thường được sử dụng. Giống như JSON, định dạng này phổ biến bởi vì nó có thể đọc được và plugin "kv" filter cho phép bạn tự động phân tích các thông điệp hoặc các trường cụ thể được định dạng theo cách này.

Lấy nhật ký sau làm ví dụ:

```
2017-07025 17:02:12 level=error message="connection refused" 
service="listener" thread=125 customerid=776622 ip=34.124.233.12 
queryid=45
```

Sử dụng "kv" filter để Logstash xử lý:

```
filter {
  kv {
      source => "metadata"
      trim => "\""
      include_keys => [ "level","service","customerid",”queryid” ]
      target => "kv"
   }
 }
```

>Luu ý việc sử dụng ở đây của các tùy chọn cấu hình. Tôi sử dụng source để xác định trường thục hiện tìm kiếm key = vlaue, cắt bỏ các ký tự cụ thể; include_keys để chỉ định các khóa được phân tích cú pháp cần thêm vào nhật ký và nhắm mục tiêu để xác định vùng chứa tất cả các cặp khóa - giá trị sẽ được đặt trong.

Tóm lại, như tôi đã nói ở đàu bài viết, có 1 số lượng lớn các Logstash plugin filter được hỗ trợ mà bạn có thể sử dụng. Tất nhiên việc sử dụng cái nào còn phụ thuộc rất nhiều vào thông điệp log cụ thể mà bạn muốn xử lý.

Các filter plugin cực kỳ hữu ích khác đáng được đề cập là các plugin Geoip (để thêm dữ liệu địa lý cho các trường IP) và các plugin csv (để phân tích nhật ký CSV).

Mặc dù mỗi 1 trong số các plugin này đều hữu ích theo cách riêng của nó, toàn bộ sức mạnh của chúng được giải phóng khi sử dụng cùng nhau để phân tích các bản ghi log. Trong hầu hết các trường hợp, rất có thể bạn sẽ sử dụng kết hợp giữa Grok và ít nhất 1 hoặc 2 plugin bổ sung. việc sử dụng kết hợp này sẽ đảm bảo nhật ký log của bạn xuất hiện ở đầu kia của Logstash được định dạng hoàn hảo.