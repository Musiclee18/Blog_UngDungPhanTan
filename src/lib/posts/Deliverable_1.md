---
title: "Deliverable_1: Đề xuất mô tả đề tài giữa kì"
date: "2025-5-11"
updated: "2025-5-11"
categories:
  - "sveltekit"
  - "markdown"
  - "Deliverable_1: Đề xuất mô tả đề tài giữa kì" 
coverImage: "/images/mo-hinh-memcached.png"
coverWidth: 16
coverHeight: 9
excerpt: Tối ưu hóa hiệu suất hệ thống web bán hàng phân tán sử dụng Memcached
---

Lê Tiến Nhạc - 22010108

## Đề tài: Tối ưu hóa hiệu suất hệ thống web bán hàng phân tán sử dụng Memcached

## 1. Mục đích của thư viện Memcached

### Mục đích của Memcached:
Memcached là một công cụ lưu trữ dữ liệu tạm thời trong bộ nhớ (in-memory), giúp tối ưu tốc độ truy cập dữ liệu bằng cách lưu trữ các thông tin truy cập thường xuyên (như thông tin người dùng, giỏ hàng, sản phẩm) trong bộ nhớ RAM. Điều này giúp giảm tải cho cơ sở dữ liệu chính và nâng cao hiệu suất cho các ứng dụng cần tốc độ truy cập nhanh.

### Vấn đề mà Memcached giải quyết:
- **Tối ưu hiệu suất**: Giảm thiểu độ trễ khi truy cập dữ liệu, đảm bảo người dùng có thể thực hiện các thao tác nhanh chóng mà không phải chờ đợi.
- **Giảm tải cho cơ sở dữ liệu**: Memcached giúp giảm bớt các truy vấn tới cơ sở dữ liệu bằng cách lưu trữ các kết quả truy vấn đã được thực hiện vào bộ nhớ.
- **Quy mô mở rộng dễ dàng**: Memcached hỗ trợ các tính năng phân mảnh dữ liệu (sharding) và sao chép (replication), giúp phân phối và duy trì dữ liệu trong các hệ thống phân tán.

### Điểm mạnh của Memcached:
- **Hiệu suất cao**: Do sử dụng bộ nhớ RAM, tốc độ truy xuất dữ liệu nhanh chóng, thích hợp cho các ứng dụng có lượng truy cập cao.
- **Dễ triển khai và tích hợp**: API của Memcached dễ sử dụng và có thể tích hợp nhanh chóng với các ứng dụng web, đặc biệt trong môi trường phân tán.
- **Khả năng mở rộng**: Hỗ trợ phân tán (sharding) và sao chép (replication), giúp hệ thống có thể mở rộng quy mô dễ dàng mà không ảnh hưởng đến hiệu suất.

### Điểm yếu của Memcached:
- **Không bảo vệ tính nhất quán mạnh**: Memcached chỉ là một hệ thống cache, không đảm bảo tính nhất quán dữ liệu giữa các nút trong các tình huống phân tán phức tạp.
- **Không có tính năng lưu trữ lâu dài**: Memcached chỉ lưu trữ dữ liệu tạm thời, không phải là cơ sở dữ liệu bền vững (persistent storage).

### So sánh với các thư viện/framework khác:
- **Redis**: So với Memcached, Redis cung cấp nhiều tính năng hơn như hỗ trợ các kiểu dữ liệu phức tạp (list, set, sorted set) và cơ chế sao lưu dữ liệu (persistence), nhưng Memcached vẫn ưu việt hơn về tốc độ trong các tác vụ đơn giản và ít tính toán phức tạp.
- **Database như MySQL/PostgreSQL**: Các hệ quản trị cơ sở dữ liệu như MySQL, PostgreSQL có tính nhất quán mạnh mẽ và lưu trữ dữ liệu lâu dài, nhưng chúng không hiệu quả trong việc xử lý các yêu cầu truy vấn tần suất cao. Memcached giúp giảm tải và nâng cao hiệu suất cho các ứng dụng web.

### Ứng dụng:
- **Web bán hàng (E-commerce)**: Sử dụng Memcached để lưu trữ tạm thời thông tin sản phẩm, giỏ hàng, và thông tin người dùng giúp tối ưu hiệu suất và cải thiện trải nghiệm người dùng trong môi trường phân tán.
- **Web ứng dụng có khối lượng truy cập lớn**: Memcached hữu ích cho các trang web có nhiều truy vấn nhanh và cần tối ưu hóa tốc độ như các dịch vụ trực tuyến, mạng xã hội, các hệ thống tìm kiếm.

---

## 2. Nộp kế hoạch dự kiến về bài giữa kỳ

### Đề tài: Tối ưu hóa hiệu suất hệ thống web bán hàng phân tán sử dụng Memcached

### Bài toán:
#### Mô tả bài toán:
Xây dựng một hệ thống web bán hàng phân tán với các tính năng như hiển thị sản phẩm, quản lý giỏ hàng, thanh toán và quản lý tài khoản người dùng. Ứng dụng Memcached để tối ưu hóa việc truy xuất các dữ liệu truy cập thường xuyên (như thông tin sản phẩm, giỏ hàng của người dùng, và lịch sử mua sắm), giảm tải cho cơ sở dữ liệu và đảm bảo hiệu suất cao ngay cả khi hệ thống có lượng người dùng lớn.

#### Các tính năng chính:
- **Hiển thị danh sách sản phẩm**: Lưu trữ thông tin sản phẩm trong bộ nhớ để giảm độ trễ khi người dùng xem danh sách sản phẩm.
- **Quản lý giỏ hàng**: Lưu thông tin giỏ hàng của người dùng vào Memcached để dễ dàng truy cập và thao tác nhanh chóng.
- **Quản lý tài khoản người dùng**: Thông tin tài khoản và lịch sử mua sắm của người dùng sẽ được lưu trữ trong bộ nhớ.
- **Thanh toán**: Sau khi giỏ hàng được xử lý, hệ thống sẽ thực hiện thanh toán, và trạng thái thanh toán sẽ được lưu vào bộ nhớ để dễ dàng truy xuất và xử lý các yêu cầu liên quan đến đơn hàng.

#### Các phương thức phân tán áp dụng:
- **Fault Tolerance**: Hệ thống sẽ đảm bảo hoạt động liên tục ngay cả khi một nút bị lỗi thông qua việc sao chép dữ liệu giữa các nút.
- **Sharding / Replication**: Memcached sẽ được sử dụng để phân mảnh dữ liệu (sharding) hoặc sao chép dữ liệu (replication) giữa các máy chủ trong hệ thống phân tán để tối ưu hiệu suất và giảm độ trễ.
- **Distributed Communication**: Các nút trong hệ thống giao tiếp với nhau qua giao thức HTTP hoặc gRPC để truyền tải dữ liệu và đảm bảo hệ thống hoạt động ổn định.

### Kế hoạch giữa kỳ:
#### Mục tiêu:
Xây dựng một ứng dụng web bán hàng phân tán cơ bản sử dụng Memcached để tối ưu hiệu suất và thực hiện các phương thức phân tán đã học như phân mảnh dữ liệu và sao chép.

#### Các công việc cần thực hiện:
- **Cài đặt và cấu hình Memcached**: Thiết lập Memcached trên các máy chủ và kết nối ứng dụng với Memcached.
- **Phát triển ứng dụng web bán hàng**: Tạo giao diện đơn giản cho trang sản phẩm, giỏ hàng, và thanh toán.
- **Kết nối hệ thống phân tán**: Đảm bảo rằng các nút trong hệ thống có thể giao tiếp với nhau thông qua HTTP/gRPC và dữ liệu được phân tán và sao chép giữa các nút.
- **Stress test**: Thực hiện kiểm tra tải cơ bản bằng cách mô phỏng nhiều yêu cầu và người dùng đồng thời để kiểm tra hiệu suất.

### Dự kiến kết quả giữa kỳ:
- Ứng dụng web bán hàng phân tán cơ bản được triển khai với các chức năng chính hoạt động ổn định.
- Hệ thống có thể sử dụng Memcached để tối ưu hóa hiệu suất truy xuất dữ liệu.
