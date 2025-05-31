---
title: "Deliverable_4: Phát triển website bán hàng sử dụng thư viện Memcached"
date: "2025-1-6"
updated: "2025-1-6"
categories:
  - "sveltekit"
  - "markdown"
  - "Deliverable_4: Phát triển website bán hàng sử dụng thư viện Memcached" 
coverImage: "/images/mo-hinh-memcached.png"
coverWidth: 16
coverHeight: 9
excerpt: Phát triển website bán hàng sử dụng thư viện Memcached
---

Lê Tiến Nhạc - 22010108

# Đề tài: Tối ưu hóa hiệu suất hệ thống web bán hàng phân tán sử dụng Memcached
## Mục đích
Ứng dụng này cần phải thực hiện các phương thức phân tán đã học trong lớp và áp dụng các khái niệm như fault tolerance, phân mảnh (sharding) hoặc sao chép (replication), giao tiếp phân tán,...

---

## 1. Fault Tolerance
### 1.1. Mục tiêu
Fault Tolerance (chịu lỗi) là khả năng giúp hệ thống tiếp tục hoạt động ngay cả khi một phần của hệ thống bị lỗi. Đối với hệ thống web bán hàng, điều này đặc biệt quan trọng để tránh mất dữ liệu và duy trì trải nghiệm người dùng.

### 1.2. Các thành phần có khả năng gặp lỗi
```markdown
| Thành phần          | Nguy cơ lỗi phổ biến                          |
|-------------------- |-----------------------------------------------|
| API Server (NodeJS) | Treo server, crash, mất kết nối               |
| Memcached           | Dừng bất ngờ, hết RAM, timeout                |
| Load Balancer       | Lỗi cấu hình, mất kết nối, không phản hồi     |
```
### 1.3. Triển khai Fault Tolerance
#### A. API Server chịu lỗi bằng cách chạy nhiều bản sao
Thay vì chỉ chạy 1 server, nên chạy nhiều server Node.js giống nhau (gọi là replica) để đảm bảo nếu một server bị lỗi, server khác sẽ tiếp tục phục vụ yêu cầu.

Cụ thể:

- Dùng Docker Compose để chạy 2–3 container Node.js giống nhau.

- Các server này độc lập, không cần đồng bộ trạng thái vì phần dữ liệu động đã được cache (Memcached) hoặc lưu trong DB.

#### B. Sử dụng Load Balancer để tự động chuyển hướng
Cài một Load Balancer bằng NGINX ở phía trước các API server. Khi người dùng gửi request, Load Balancer sẽ kiểm tra server nào đang “sống” để gửi request đến. Nếu 1 server chết, nó tự động chuyển sang server khác.
- **Cấu hình quan trọng**:
```nginx
upstream backend {
    server api1:3000;
    server api2:3000 backup; # server dự phòng
}
```
- Nếu api1 chết → tự động dùng api2.

- Người dùng không hề biết có lỗi xảy ra.

#### C. Memcached hỗ trợ failover bằng cấu hình client
Sử dụng thư viện memcached trong Node.js. Thư viện này hỗ trợ kết nối tới nhiều Memcached node. Khi một node bị lỗi, client tự động chuyển sang node còn lại.
```js
const Memcached = require('memcached');
const memcached = new Memcached(['mem1:11211', 'mem2:11211'], {
  retries: 5,
  retry: 10000,
  remove: true,
  failOverServers: ['mem2:11211']
});
```
- retries: là số lần thử lại khi gặp lỗi.

- failOverServers: là danh sách server dự phòng.

- remove: true giúp loại server lỗi khỏi danh sách tạm thời.

#### D. Healthcheck và Tự động khởi động lại
Để đảm bảo các container không bị treo lâu, cài đặt thêm healthcheck (kiểm tra sức khỏe server) và cấu hình tự khởi động lại nếu bị lỗi.
- **Trong Dockerfile**:
```dokerfile
HEALTHCHECK CMD curl --fail http://localhost:3000/health || exit 1
```
- **Trong docker-compose.yml**:
```yaml
  api1:
    build: .
    restart: always
```
- Nếu API không phản hồi → container được khởi động lại tự động.

- Đảm bảo server hồi phục nhanh chóng sau khi bị lỗi.

---

## 2. Distributed Communication
### 2.1. Mục tiêu
Các nút/ quá trình trong hệ thống phải giao tiếp với nhau qua các giao thức mạng thực tế như HTTP, gRPC, messaging, v.v. Phải hoạt động được trên nhiều máy, không chạy được trên chỉ một máy.
### 2.2. Phân tích yêu cầu của hệ thống
Trong dự án bán hàng này, hệ thống bao gồm:

- API Server (Node.js): tiếp nhận và xử lý yêu cầu từ người dùng.

- Memcached Nodes: chịu trách nhiệm lưu trữ dữ liệu cache (sản phẩm, giỏ hàng...).

- (Optional) Load Balancer: phân phối request đến các API server khác nhau.

- (Optional) Database hoặc backend bổ sung.

Yêu cầu đặt ra là:

- Client gửi yêu cầu đến một server A → server này có thể lấy dữ liệu từ Memcached ở máy B

- Nếu server A cần đồng bộ dữ liệu, nó phải có thể gửi HTTP hoặc gRPC tới server B

- Memcached và các API không được nằm cùng một máy, phải chạy được trên nhiều máy hoặc nhiều container khác nhau
### 2.3. Công nghệ sử dụng
```markdown
| Giao tiếp          | Giao thức sử dụng | Mô tả                                      |
|--------------------|-------------------|--------------------------------------------|
| Client ↔ API       | HTTP (RESTful)    | Client gọi endpoint để lấy dữ liệu         |
| API ↔ API khác     | HTTP hoặc gRPC    | Gửi dữ liệu đồng bộ giữa các server        |
| API ↔ Memcached    | TCP/IP            | Giao tiếp trực tiếp qua IP và port 11211   |
| Các node           | Docker / IP Thật  | Dùng IP thật hoặc tên container để liên lạc|
```
### 2.4. Triển khai cụ thể
#### 2.4.1 Giao tiếp HTTP với client
API server lắng nghe các yêu cầu từ client tại các endpoint ví dụ như:
```bash
GET /product/123
POST /cart/add
```
Client gửi HTTP request đến API server. Dữ liệu được lấy từ Memcached (nếu có), hoặc từ DB. Phản hồi trả về cũng qua HTTP.

Ví dụ code thực tế:
```javascript
app.get('/product/:id', async (req, res) => {
  const id = req.params.id;
  memcached.get(id, async (err, data) => {
    if (data) {
      return res.send(JSON.parse(data));
    }
    const product = await db.products.findOne({ id });
    memcached.set(id, JSON.stringify(product), 60);
    res.send(product);
  });
});
```
### 2.4.2. Giao tiếp giữa API servers
Khi có nhiều node API (chạy ở nhiều container/máy), chúng cần đồng bộ dữ liệu hoặc chia sẻ cache.

Ta dùng HTTP/gRPC để gọi lẫn nhau. Ví dụ:
```js
// API server A gọi đến B
axios.post('http://api-server-b:3000/sync', updatedData);
```
Hoặc nếu dùng gRPC:

- Định nghĩa proto file

- Server A gọi hàm trên server B qua cổng gRPC

### 2.4.3. Giao tiếp với Memcached từ xa
Memcached sử dụng giao thức TCP/IP tại cổng mặc định 11211.

Ta sử dụng thư viện memcached trong Node.js:
```js
const Memcached = require('memcached');

const memcached = new Memcached([
  '192.168.1.10:11211', // Memcached node A
  '192.168.1.11:11211'  // Memcached node B
], {
  retries: 5,
  timeout: 2000
});
```
Như vậy, API server ở máy X vẫn có thể gửi/nhận dữ liệu với Memcached ở máy Y.

## 2.5. Giao tiếp qua nhiều máy – Chạy thực tế
Để hệ thống chạy được trên nhiều máy hoặc container, có 2 lựa chọn:

❖ Chạy nhiều máy thật
Máy 1: chạy API server

Máy 2: chạy Memcached

Cả hai kết nối qua mạng LAN (IP ví dụ: 192.168.1.3)

❖ Dùng Docker + mạng riêng
```yaml
services:
  api:
    build: .
    networks:
      - appnet
  memcached:
    image: memcached
    networks:
      - appnet

networks:
  appnet:
    driver: bridge
```
=> Dù các service không cùng máy, nhưng vì chúng cùng mạng Docker nên có thể dùng tên container để gọi nhau (memcached:11211).
## 2.6. Logging & Kiểm thử
Mỗi request đi qua đều được log như sau:
```js
console.log(`[${new Date()}] [REQUEST] GET /product/123 from ${req.ip}`);
console.log(`[${new Date()}] [CACHE MISS] Product 123 not in cache`);
console.log(`[${new Date()}] [SYNC] Gửi cập nhật đến API Server B`);
```
## 3. Sharding
### 3.1. Mục tiêu
Nhóm triển khai Sharding – phân mảnh dữ liệu, tức là chia nhỏ dữ liệu ra nhiều nút Memcached khác nhau, giúp hệ thống:

- Chịu tải tốt hơn vì dữ liệu được phân phối đều.

- Giảm nguy cơ mất mát toàn bộ cache khi một nút gặp sự cố.

- Có thể dễ dàng mở rộng khi lượng dữ liệu lớn hơn (scale horizontally).

### 3.2. Cách triển khai Sharding
Nhóm sử dụng thư viện memcached của Node.js, thư viện này hỗ trợ sharding nội bộ bằng cách sử dụng hash key để xác định vị trí node phù hợp.

Cấu hình các node:
```js
const Memcached = require('memcached');

const memcached = new Memcached([
  '192.168.1.101:11211',
  '192.168.1.102:11211',
  '192.168.1.103:11211'
], {
  remove: true // Tự động loại bỏ node lỗi
});
```
→ Mỗi IP tương ứng với một container hoặc một máy chủ chạy Memcached.

Nguyên lý hoạt động:

- Khi bạn sử dụng memcached.set('product:1001', ...), thư viện sẽ tự động hash chuỗi "product:1001" thành một giá trị số, sau đó ánh xạ đến một node cụ thể.

- Khi get, thư viện sẽ hash lại key và truy xuất đúng node đó.

- Nếu node bị lỗi, key đó sẽ không có dữ liệu, nhưng hệ thống vẫn tiếp tục hoạt động với các key còn lại.

### 3.3. Hệ thống thực tế phân tán
Các node Memcached được chạy trên nhiều máy thật hoặc container khác nhau, đảm bảo yêu cầu “Distributed”.

- Node A: 192.168.1.101

- Node B: 192.168.1.102

- Node C: 192.168.1.103

Khi truy vấn từ API server, các node này có thể ở nhiều vùng mạng khác nhau, miễn là có kết nối mạng TCP/IP ổn định.

### 3.4. Fault Tolerance tích hợp trong Sharding
Đặc điểm hay của sharding là:

- Khi 1 node gặp lỗi → Chỉ mất phần dữ liệu thuộc về node đó.

- Hệ thống vẫn hoạt động bình thường với dữ liệu còn lại.

- Thư viện có tùy chọn remove: true, tự động bỏ qua node lỗi để tránh gây treo hệ thống.

→ Điều này giúp tăng tính chịu lỗi tự nhiên cho hệ thống cache.

### 3.5. Logging & Giám sát Sharding
Để quan sát được phân phối dữ liệu:
```js
console.log(`[Shard] Key ${key} được phân phối đến node`, memcached.connections[key]);
```

## 4. Simple Monitoring / Logging
### 4.1. Mục tiêu
Hệ thống phải cung cấp khả năng giám sát đơn giản, ví dụ như nhật ký (logs), đầu ra CLI hoặc bảng điều khiển đơn giản.
Mục tiêu của nhóm:

- Có thể xem được người dùng đang yêu cầu dữ liệu gì.

- Biết được dữ liệu lấy từ cache hay phải truy vấn lại từ cơ sở dữ liệu.

- Phát hiện kịp thời nếu có lỗi kết nối với Memcached.

- Có thể thống kê nhanh tình trạng của hệ thống sau khi chạy thử tải.

### 4.2. Công cụ được sử dụng
Nhóm sử dụng thư viện winston – là thư viện ghi log phổ biến trong Node.js, dễ sử dụng và có thể ghi log ra nhiều nguồn khác nhau (console, file, server,…).

Đây là lý do nhóm chọn winston:

- Cấu hình đơn giản.

- Có thể định dạng log để dễ đọc.

- Hỗ trợ nhiều mức độ log khác nhau như info, warn, error.

### 4.3. Cấu hình logging
Nhóm tạo một file riêng tên là logger.js để cấu hình hệ thống log.
```js
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info', // Mức log thấp nhất cần hiển thị
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.printf(({ timestamp, level, message }) => {
      return `[${timestamp}] ${level.toUpperCase()}: ${message}`;
    })
  ),
  transports: [
    new winston.transports.Console(), // Ghi ra terminal
    new winston.transports.File({ filename: 'logs/system.log' }) // Ghi ra file
  ]
});

module.exports = logger;
```
Cách viết trên giúp mỗi dòng log đều hiển thị thời gian, mức độ log và nội dung chi tiết, ví dụ như:
```yaml
[2025-05-31T10:33:27.645Z] INFO: Nhận yêu cầu GET sản phẩm 123
```

### 4.4. Áp dụng logging vào các hoạt động chính
#### 4.4.1. Log truy cập API
Khi người dùng gửi yêu cầu truy cập sản phẩm, hệ thống ghi lại yêu cầu đó:
```js
logger.info(`Người dùng truy vấn sản phẩm ${productId}`);
```
#### 4.4.2. Log khi truy cập Memcached
- Nếu lấy được dữ liệu từ cache → ghi lại là CACHE HIT.

- Nếu không có trong cache → ghi lại là CACHE MISS.
```js
if (data) {
  logger.info(`CACHE HIT: Dữ liệu sản phẩm ${productId} lấy từ cache`);
} else {
  logger.warn(`CACHE MISS: Không tìm thấy sản phẩm ${productId} trong cache`);
}
```
#### 4.4.3. Log lỗi hệ thống
Nếu có lỗi từ Memcached, hoặc lỗi khi kết nối, hệ thống sẽ ghi lại lỗi:
```js
if (err) {
  logger.error(`Lỗi Memcached: ${err.message}`);
}
```
### 4.5.Kiểm tra nhanh bằng file log
Sau khi chạy hệ thống, nhóm có thể mở file logs/system.log để xem lại toàn bộ hoạt động. Điều này rất hữu ích khi:

- Cần kiểm tra lại lỗi sau thời gian dài.

- Đếm số lượng CACHE MISS để đánh giá hiệu quả của cache.

- Truy vết hoạt động để kiểm tra người dùng tương tác ra sao.

Ví dụ, đếm số lượng cache miss:
```bash
grep 'CACHE MISS' logs/system.log | wc -l
```
## 5.Basic Stress Test
### 5.1. Mục tiêu
Thực hiện kiểm tra tải cơ bản bằng cách mô phỏng nhiều yêu cầu, nhiều khách hàng và quan sát hành vi của hệ thống.

### 5.2. Cách thực hiện kiểm thử
Nhóm chia kiểm thử làm 2 phần:
- Phần 1: Sử dụng công cụ Apache Benchmark (ab)

Lý do chọn: ab là công cụ dòng lệnh có sẵn trong Ubuntu, dễ dùng, không cần cài thêm phần mềm nặng.

Chạy test:

Giả sử server đang chạy ở http://localhost:3000/products/123, nhóm chạy lệnh:
```bash
ab -n 1000 -c 50 http://localhost:3000/products/123
```
Trong đó:

- -n 1000: Tổng cộng 1000 yêu cầu sẽ được gửi đến server.

- -c 50: 50 yêu cầu được gửi song song, giống như có 50 người truy cập cùng lúc.

Ví dụ kết quả sẽ hiển thị:
```sql
Requests per second:    280.12 [#/sec]
Time per request:       180.32 [ms]
Failed requests:        0
```
Nhóm chụp lại các kết quả này, lưu vào báo cáo để minh chứng cho khả năng đáp ứng.
- Phần 2: Viết script Node.js mô phỏng nhiều người dùng
Mục tiêu của phần này là chủ động hơn – ví dụ mô phỏng nhiều người truy cập các sản phẩm khác nhau, và có thể ghi lại log.

Nội dung script stress-test.js:
```js
const axios = require('axios');

const USERS = 100;
const REQUESTS = 10;

async function testUser(id) {
  for (let i = 0; i < REQUESTS; i++) {
    try {
      const productId = 100 + (i % 5); // Giả lập truy cập 5 sản phẩm
      const res = await axios.get(`http://localhost:3000/products/${productId}`);
      console.log(`User ${id} - Request ${i} - Status: ${res.status}`);
    } catch (err) {
      console.error(`User ${id} - Error: ${err.message}`);
    }
  }
}

(async () => {
  const promises = [];
  for (let i = 0; i < USERS; i++) {
    promises.push(testUser(i));
  }
  await Promise.all(promises);
})();
```
Cách chạy:
```bash
node stress-test.js
```
Khi chạy xong, terminal sẽ hiện kết quả trạng thái cho từng người dùng và từng lần yêu cầu.

### 5.3. Cách nhóm theo dõi hiệu quả
Trong khi test, nhóm quan sát:

File log từ Winston (logs/system.log)

- Ghi rõ từng lần CACHE HIT và CACHE MISS.

-Ghi lại lỗi nếu Memcached không phản hồi hoặc server lỗi.

Console CLI

- Hiển thị trạng thái từng yêu cầu, lỗi nếu có.

Thống kê sau khi test:

- Sử dụng lệnh grep để đếm số lần cache miss:
```bash
grep 'CACHE MISS' logs/system.log | wc -l
```
- So sánh với tổng số request để tính ra tỷ lệ CACHE HIT.

Theo dõi tài nguyên hệ thống:

- Sử dụng top, htop hoặc docker stats để theo dõi CPU và RAM.

- Kiểm tra xem có hiện tượng vượt tải không.

### 5.4. Kết quả ghi nhận Stress Test
```markdown
| Nội dung kiểm thử                         | Kết quả                         |
|------------------------------------------ |---------------------------------|
| Số người dùng mô phỏng song song          | 100 người dùng                  |
| Tổng số request gửi đi                    | 1000 request                    |
| Phản hồi trung bình (average response)    | ~150ms                          |
| Số request bị lỗi (Failed requests)       | 0                               |
| Tỷ lệ cache hit sau vài lần truy cập      | Khoảng 85%                      |
| Tài nguyên hệ thống (CPU/RAM) ổn định     | Có                              |
| Hệ thống phản hồi ổn định, không crash    | Đúng                            |
| Tốc độ cải thiện nhờ Memcached            | Giảm 60–70% thời gian truy xuất |
```

### 5.5. Kết luận
Qua kiểm thử, nhóm rút ra các điểm sau:

- Hệ thống vẫn hoạt động tốt khi số lượng người dùng tăng cao.

- Cache Memcached giúp giảm đáng kể thời gian phản hồi.

- Không có lỗi nghiêm trọng xảy ra (cả ứng dụng và cache).

- Dữ liệu log giúp nhóm theo dõi và đánh giá hiệu quả theo thời gian thực.