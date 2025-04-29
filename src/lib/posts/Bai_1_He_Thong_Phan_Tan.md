---
title: "Hệ thống phân tán"
date: "2025-04-28"
excerpt: "Khám phá các khái niệm cơ bản và ứng dụng của hệ thống phân tán trong các dịch vụ hiện đại."
category: "Công nghệ"
---

# Hệ thống phân tán

## Hệ thống phân tán là gì?

Hệ thống phân tán (Distributed System) là tập hợp các máy tính độc lập được kết nối với nhau qua mạng, phối hợp hoạt động như một hệ thống thống nhất đối với người dùng. Các máy tính này có thể ở những địa điểm địa lý khác nhau, nhưng chúng chia sẻ tài nguyên, dữ liệu và các tác vụ xử lý để cung cấp dịch vụ hiệu quả, tin cậy và mở rộng được.

## Các ứng dụng của hệ thống phân tán

- **Dịch vụ Web**: Các dịch vụ như Google, Facebook, Amazon sử dụng các hệ thống phân tán quy mô lớn để phục vụ hàng tỷ người dùng.
- **Ứng dụng lưu trữ đám mây**: Google Drive, Dropbox, iCloud vận hành trên nền tảng hệ thống phân tán.
- **Hệ thống tài chính ngân hàng**: Hệ thống giao dịch thời gian thực giữa các chi nhánh khác nhau.
- **Game online đa người chơi**: Các máy chủ game MMO (Massively Multiplayer Online) phân phối tải để phục vụ hàng triệu người chơi cùng lúc.
- **Mạng xã hội**: Các nền tảng như Instagram, TikTok cần phân phối dữ liệu đa trung tâm để đảm bảo tốc độ truy cập nhanh.

## Các khái niệm chính của hệ thống phân tán

### Scalability (Khả năng mở rộng)

Khả năng hệ thống mở rộng xử lý nhiều tải hơn bằng cách tăng thêm tài nguyên (máy chủ, lưu trữ, băng thông).

### Fault Tolerance (Khả năng chịu lỗi)

Khả năng tiếp tục hoạt động bình thường ngay cả khi một số thành phần bị lỗi.

### Availability (Khả dụng)

Khả năng hệ thống luôn sẵn sàng đáp ứng yêu cầu từ người dùng với thời gian chết tối thiểu.

### Transparency (Tính trong suốt)

Người dùng và lập trình viên có thể tương tác với hệ thống như một thể thống nhất, ẩn đi sự phức tạp bên dưới.

Các dạng transparency:
- Location Transparency
- Replication Transparency
- Concurrency Transparency
- Failure Transparency

### Concurrency (Tính đồng thời)

Nhiều tiến trình hoặc người dùng có thể tương tác với hệ thống cùng lúc mà không gây ra xung đột.

### Parallelism (Tính song song)

Chạy nhiều tác vụ cùng lúc để rút ngắn thời gian xử lý tổng thể.

### Openness (Tính mở)

Hệ thống cho phép tích hợp các thành phần phần mềm và phần cứng từ nhiều nhà cung cấp khác nhau dựa trên các giao thức tiêu chuẩn mở.

### Vertical Scaling (Mở rộng theo chiều dọc)

Tăng năng lực xử lý bằng cách nâng cấp phần cứng (CPU, RAM) trên một máy chủ đơn lẻ.

### Horizontal Scaling (Mở rộng theo chiều ngang)

Thêm nhiều máy chủ mới để phân phối tải công việc, giúp hệ thống mở rộng linh hoạt.

### Load Balancer (Bộ cân bằng tải)

Phân phối lưu lượng truy cập hoặc công việc đồng đều giữa nhiều máy chủ để đảm bảo hiệu suất cao và ngăn ngừa quá tải.

### Replication (Sao chép dữ liệu)

Tạo và duy trì nhiều bản sao của dữ liệu tại nhiều địa điểm khác nhau để tăng khả năng chịu lỗi và khả dụng.

## Ví dụ minh họa: Website bán hàng trực tuyến

**Mô tả:** Một website thương mại điện tử lớn như Amazon phục vụ hàng triệu người dùng truy cập đồng thời.

**Liên hệ với các thuật ngữ:**
- **Scalability**: Khi nhu cầu tăng cao (sale Black Friday), thêm máy chủ mới để xử lý thêm lưu lượng (horizontal scaling).
- **Fault Tolerance**: Một trung tâm dữ liệu bị sự cố, hệ thống tự động chuyển sang trung tâm khác.
- **Availability**: Website luôn online 24/7.
- **Transparency**: Người dùng không biết dữ liệu của họ đến từ máy chủ nào, ở đâu.
- **Concurrency**: Hàng triệu người dùng mua sắm cùng lúc.
- **Parallelism**: Xử lý đơn hàng và thanh toán song song để tiết kiệm thời gian.
- **Openness**: Hệ thống hỗ trợ các API mở để tích hợp với nhiều dịch vụ vận chuyển khác nhau.
- **Vertical Scaling**: Nếu máy chủ cơ sở dữ liệu bị chậm, nâng cấp RAM, CPU.
- **Horizontal Scaling**: Thêm nhiều web server xử lý đơn hàng.
- **Load Balancer**: Phân phối truy cập giữa nhiều web server.
- **Replication**: Dữ liệu đơn hàng được sao lưu sang nhiều trung tâm dữ liệu.

## Kiến trúc của hệ thống phân tán

### Các mô hình kiến trúc phổ biến

1. **Client-Server Architecture**
   - Máy khách (Client) gửi yêu cầu, máy chủ (Server) xử lý và trả về kết quả.
   
2. **Peer-to-Peer Architecture (P2P)**
   - Các nút mạng (node) vừa đóng vai trò máy chủ vừa là máy khách (ví dụ: mạng BitTorrent).

3. **Three-Tier Architecture**
   - Gồm 3 lớp: Presentation (Giao diện người dùng), Logic (Xử lý nghiệp vụ), Data (Dữ liệu).

4. **Microservices Architecture**
   - Chia hệ thống thành nhiều dịch vụ nhỏ, độc lập, giao tiếp với nhau qua API.

5. **Serverless Architecture**
   - Triển khai mã nguồn mà không cần quản lý máy chủ vật lý (dịch vụ như AWS Lambda).

6. **Edge Computing Architecture**
   - Xử lý dữ liệu gần với nguồn tạo ra dữ liệu, thay vì đẩy về trung tâm dữ liệu.

### Ví dụ về kiến trúc hệ thống phân tán

**Ví dụ:**  
- **Microservices + Load Balancer + Database Replication**  
  Một trang web TMĐT chia thành các dịch vụ microservice (giỏ hàng, thanh toán, quản lý kho). Các dịch vụ này được load balancer phân phối lưu lượng và dữ liệu khách hàng được replication để đảm bảo an toàn và tính sẵn sàng.
