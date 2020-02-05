## Tìm hiểu về Check_mk

## 1. Tổng quan

<img src="img01.png">

Dự án Check_mk được phát triển từ năm 2008 như là một plugin của Nagios Core.

Năm 2010 dự án OMD (Open Monitoring Distribution) được khởi động bởi Mathias Kettner, là sự kết hợp của Nagios, Check_MK, NagVis, PNP4Nagios, DocuWiki, ...tạo nên sự linh hoạt trong giám sát. Các distro của OMD đang là OMD-LABS và CHECK_MK RAW.

Năm 2015, phiên bản đơn giản của OMD ra đời gọi là Check_MK.

Đây là một phần mềm được phát triển bằng Python và C ++ để theo dõi Cơ sở hạ tầng CNTT. Nó được sử dụng để giám sát các máy chủ, ứng dụng, mạng, cơ sở hạ tầng đám mây (công cộng, riêng, lai), container, storages, cơ sở dữ liệu ...

Checkmk có sẵn trong bốn phiên bản khác nhau:

- Checkmk Raw Edition hoàn toàn miễn phí, 100% mã nguồn mở, và toàn diện hệ thống giám sát CNTT.

- Checkmk Enterprise - Free Edition là phiên bản miễn phí của sản phẩm thương mại, bị giới hạn ở 2 sites, mỗi với tối đa 10 host có thể được theo dõi.

- Checkmk Enterprise - Standard Edition bao gồm nhiều tính năng bổ sung có liên quan, cung cấp một khả năng mở rộng lớn hơn, hỗ trợ doanh nghiệp thông qua các nhà cung cấp hoặc các partner network. Nó có sẵn dưới dạng gói thuê bao hàng năm, với giá khởi điểm là 600 €, chưa kể thuế.

- Checkmk Enterprise - Managed Services Edition dựa trên Enterprise - Standard Edition và cung cấp các tính năng đặc biệt nhắm vào các công ty muốn cung cấp Checkmk dưới dạng dịch vụ được quản lý, giá khởi điểm là 1200 €, chưa kể thuế.

Các phiên bản Check_mk này có sẵn cho một loạt các nền tảng, đặc biệt là các phiên bản khác nhau của Debia, Ubunt, SLES (SUSE Linux Enterprise Server) và RedHat/CentOS và cả dưới dạng Docker Image.

Các Checkmk agent được sử dụng để thu thập dữ liệu có sẵn cho 11 nền tảng, bao gồm cả Windows.