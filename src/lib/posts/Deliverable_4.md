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

### 1.2. Đề xuất và ý tưởng
#### 1.2.1. Đề xuất ý tưởng
Giả sử chúng ta có 2 server Memcached:
```markdown
| Tên server        | IP             | Vai trò                  |
| ----------------- | -------------- | ------------------------ |
| memcached_main    | 192.168.1.10   | Server chính (primary)   |
| memcached_backup  | 192.168.1.11   | Server dự phòng (backup) |
```
Laravel app sẽ cố gắng lấy dữ liệu từ memcached_main, nếu lỗi → tự động chuyển sang memcached_backup.

#### 1.2.2. Ý tưởng tổng thể
- Nhóm sẽ sử dụng try-catch để bắt lỗi khi truy cập Memcached.

- Nếu truy cập thất bại (server chính chết, timeout, hoặc mất kết nối), hệ thống sẽ fallback sang server dự phòng để lấy dữ liệu.

- Điều này đảm bảo ứng dụng vẫn trả về dữ liệu cho người dùng → không gián đoạn dịch vụ.

### 1.3. Chi tiết các bước thực hiện Fault Tolerance
Bước 1: Cấu hình Laravel sử dụng nhiều Memcached server
```php
'memcached' => [
    'servers' => [
        [
            'host' => '192.168.1.10',
            'port' => 11211,
            'weight' => 100,
        ],
        [
            'host' => '192.168.1.11',
            'port' => 11211,
            'weight' => 50,
        ],
    ],
],
```
Giải thích:

- Laravel sẽ biết có 2 server, nhưng chưa có cơ chế fallback rõ ràng.

- Ta sẽ tự xử lý fallback thủ công để đảm bảo kiểm soát được mọi lỗi.

Bước 2: Viết code xử lý fallback thủ công

Đây là phần quan trọng nhất. Ta viết một hàm riêng trong Controller hoặc Service để lấy dữ liệu từ Memcached nhưng có cơ chế chịu lỗi rõ ràng.
```php
use Memcached;

public function getProductCache($productId)
{
    try {
        // Kết nối tới server chính
        $cache = new Memcached();
        $cache->addServer('192.168.1.10', 11211);
        
        // Lấy dữ liệu
        $data = $cache->get("product_$productId");

        // Nếu có lỗi truy xuất thì ném lỗi để xử lý fallback
        if ($cache->getResultCode() !== Memcached::RES_SUCCESS) {
            throw new \Exception("Lỗi truy cập Memcached chính");
        }
    } catch (\Exception $e) {
        // Fallback: kết nối tới server dự phòng
        $fallback = new Memcached();
        $fallback->addServer('192.168.1.11', 11211);
        $data = $fallback->get("product_$productId");

        // Ghi log để quản trị biết sự cố
        \Log::warning("Memcached chính lỗi, dùng server dự phòng", [
            'error' => $e->getMessage(),
            'product_id' => $productId
        ]);
    }

    return $data ?? 'Không có dữ liệu';
}
```
Tóm tắt nguyên lý hoạt động:
```markdown
| Giai đoạn      | Diễn giải                                               |
| -------------- | ------------------------------------------------------- |
| Kết nối chính  | Laravel kết nối `192.168.1.10` để lấy cache             |
| Kiểm tra lỗi   | Nếu `getResultCode()` không thành công thì ném lỗi      |
| Fallback       | Trong `catch`, kết nối `192.168.1.11` và lấy dữ liệu    |
| Logging        | Laravel ghi log sự kiện server chính bị lỗi để dev biết |
| Hoạt động tiếp | Dù server chính lỗi, hệ thống vẫn chạy mượt mà          |
```
### 1.4. Kiểm thử chịu lỗi
Nhóm sẽ mô phỏng đơn giản bằng cách:

Tắt Memcached chính:
```bash
systemctl stop memcached  # trên máy 192.168.1.10
```
- Gửi nhiều request đến route sử dụng hàm getProductCache().

- Laravel sẽ tự động sử dụng server backup và ghi log.

Xem log:

Mở file:
```bash
tail -f storage/logs/laravel.log
```
Sẽ thấy:
```css
[warning] Memcached chính lỗi, dùng server dự phòng
```
Như vậy, nhóm có thể giám sát được hệ thống khi xảy ra lỗi



## 2. Distributed Communication
### 2.1. Mục tiêu
Các nút/ quá trình trong hệ thống phải giao tiếp với nhau qua các giao thức mạng thực tế như HTTP, gRPC, messaging, v.v. Phải hoạt động được trên nhiều máy, không chạy được trên chỉ một máy.

### 2.2. Mô hình hệ thống phân tán
```markdown
| Thành phần                | Mô tả                              |
| ------------------------- | ---------------------------------- |
| Laravel App A (Machine 1) | Gửi request HTTP (API) đến App B   |
| Laravel App B (Machine 2) | Nhận request, xử lý và trả kết quả |
| Memcached servers         | Lưu cache dùng chung hoặc riêng    |
```
 Hai máy độc lập, mỗi máy chạy một app Laravel riêng biệt, giao tiếp thông qua HTTP.

### 2.3. Ý tưởng
Laravel App A (giao diện người dùng hoặc API gateway) sẽ gửi request HTTP (qua Guzzle) tới App B để yêu cầu xử lý dữ liệu hoặc lấy thông tin.

Tình huống ví dụ:

- App A gửi product_id sang App B để App B truy xuất thông tin sản phẩm từ Memcached hoặc DB, rồi gửi kết quả lại.

### 2.4. Chi tiết thực hiện
Bước 1: App B viết route API để nhận request

- Máy 2 – Laravel App B
```php
// routes/api.php
Route::get('/api/product/{id}', [ProductController::class, 'getProduct']);
```
Controller:
```php
// app/Http/Controllers/ProductController.php

use Illuminate\Support\Facades\Cache;

class ProductController extends Controller
{
    public function getProduct($id)
    {
        $product = Cache::get("product_$id");

        if (!$product) {
            return response()->json(['error' => 'Không tìm thấy sản phẩm'], 404);
        }

        return response()->json(['product' => $product]);
    }
}
```
```markdown
| Thành phần                | Mục đích                                   |
| ------------------------- | ------------------------------------------ |
| Route `/api/product/{id}` | Cho phép nhận request HTTP GET từ máy khác |
| `Cache::get(...)`         | Truy xuất từ Memcached hoặc cache đã setup |
| `response()->json(...)`   | Trả kết quả dưới dạng JSON qua HTTP        |
```
Bước 2: App A gửi request HTTP đến App B

- Máy 1 – Laravel App A

Cài đặt Guzzle HTTP client:
```bash
composer require guzzlehttp/guzzle
```
Gửi request đến App B:
```php
use GuzzleHttp\Client;

public function fetchProductFromRemote($productId)
{
    $client = new Client();

    try {
        $response = $client->get("http://192.168.1.20/api/product/$productId"); // IP máy B
        $data = json_decode($response->getBody()->getContents(), true);

        return $data['product'] ?? 'Không có sản phẩm';
    } catch (\Exception $e) {
        \Log::error("Lỗi giao tiếp HTTP với App B", ['message' => $e->getMessage()]);
        return 'Lỗi kết nối đến nút từ xa';
    }
}
```
Tóm tắt nguyên lý hoạt động
```markdown
| Bước | Laravel App A (Máy 1)                | Laravel App B (Máy 2)                   |
| ---- | ------------------------------------ | --------------------------------------- |
| 1    | Gửi HTTP GET tới `/api/product/{id}` | Nhận request qua route API              |
| 2    | Dùng Guzzle để gửi request           | Controller xử lý & lấy dữ liệu từ cache |
| 3    | Nhận JSON kết quả                    | Trả dữ liệu JSON                        |
| 4    | Hiển thị / xử lý dữ liệu             | -                                       |
```
### 2.5. Làm sao để đảm bảo hoạt động trên nhiều máy
Laravel App A và App B được deploy trên hai máy khác nhau, ví dụ:
```markdown
| Laravel App | IP           |
| ----------- | ------------ |
| App A       | 192.168.1.19 |
| App B       | 192.168.1.20 |
```
- App A cần biết địa chỉ IP của App B (cấu hình trong .env hoặc trong service file).

- Hệ thống cần đảm bảo:

```bash
ping 192.168.1.20
curl http://192.168.1.20/api/product/1
```
Nếu lệnh này chạy được thì giao tiếp HTTP phân tán đã hoàn chỉnh.

-> Như vậy đã đáp ứng yêu cầu Distributed Communication
```markdown
| Yêu cầu                               | Đáp ứng chưa? | Ghi chú                               |
| ------------------------------------- | ------------- | ------------------------------------- |
| Giao tiếp qua HTTP/gRPC               | ✅             | Dùng Guzzle + API REST                |
| Các node đặt trên nhiều máy khác nhau | ✅             | Laravel chạy độc lập ở 2 IP khác nhau |
| Có xử lý lỗi kết nối                  | ✅             | Try-catch và ghi log                  |
| Truyền và nhận dữ liệu thực tế        | ✅             | JSON object                           |
```

## 3. Sharding
### 3.1. Mục tiêu
Nhóm triển khai Sharding – phân mảnh dữ liệu, tức là chia nhỏ dữ liệu ra nhiều nút Memcached khác nhau, nhóm chọn Sharding là vi:

- Memcached vốn không hỗ trợ replication chuẩn (không giống Redis).

- Laravel hỗ trợ nhiều server Memcached, nên ta có thể phân mảnh dữ liệu theo khóa thủ công.

- Sharding giúp tăng hiệu năng, giảm tải cho từng node Memcached.

### 3.2. Ý tưởng tổng quát
Giả sử có 2 Memcached Server:
```markdown
| Server | IP           |
| ------ | ------------ |
| A      | 192.168.1.10 |
| B      | 192.168.1.11 |
```
Chúng ta muốn lưu các dữ liệu sản phẩm, đơn hàng, người dùng... nhưng chia đều theo khóa (sharding) như sau:

- Các ID chẵn (2, 4, 6, ...) lưu ở Server A

- Các ID lẻ (1, 3, 5, ...) lưu ở Server B

Điều này giúp phân phối dữ liệu đều hơn, tránh quá tải một server.
### 3.3. Cách triển khai
Laravel cho phép kết nối đến nhiều Memcached servers, nhưng để shard dữ liệu theo ý muốn, chúng ta cần viết logic thủ công để:

- Xác định server nào phù hợp với key

- Kết nối đến đúng server

- Lưu và truy xuất từ đúng nơi

Bước 1: Viết Service MemcachedShardManager
```php
<?php

namespace App\Services;

use Memcached;

class MemcachedShardManager
{
    protected $evenClient;
    protected $oddClient;

    public function __construct()
    {
        // Kết nối tới server A (dành cho key chẵn)
        $this->evenClient = new Memcached();
        $this->evenClient->addServer('192.168.1.10', 11211);

        // Kết nối tới server B (dành cho key lẻ)
        $this->oddClient = new Memcached();
        $this->oddClient->addServer('192.168.1.11', 11211);
    }

    // Xác định shard
    protected function getClientByKey($key)
    {
        // Giả sử key là product_1, product_2 => tách ID
        $id = (int) filter_var($key, FILTER_SANITIZE_NUMBER_INT);
        return ($id % 2 === 0) ? $this->evenClient : $this->oddClient;
    }

    public function set($key, $value, $ttl = 3600)
    {
        $client = $this->getClientByKey($key);
        return $client->set($key, $value, $ttl);
    }

    public function get($key)
    {
        $client = $this->getClientByKey($key);
        return $client->get($key);
    }
}
```
Bảng tóm tắt hoạt động
```markdown
| Thành phần                | Vai trò chính                                      |
| ------------------------- | -------------------------------------------------- |
| `evenClient`              | Kết nối tới Memcached server chứa các key ID chẵn  |
| `oddClient`               | Kết nối tới Memcached server chứa các key ID lẻ    |
| `getClientByKey($key)`    | Phân tích key để chọn đúng server (dựa trên `% 2`) |
| `set($key, $value, $ttl)` | Lưu dữ liệu vào đúng server dựa trên key           |
| `get($key)`               | Truy xuất dữ liệu từ đúng server dựa trên key      |

```
Bước 2: Sử dụng trong Controller
```php
use App\Services\MemcachedShardManager;

public function cacheProduct($id)
{
    $cache = new MemcachedShardManager();

    // Lưu sản phẩm vào đúng shard
    $key = "product_$id";
    $value = ['name' => 'Áo thun', 'price' => 199000];
    $cache->set($key, $value);

    return "Đã lưu vào shard thích hợp";
}

public function getProduct($id)
{
    $cache = new MemcachedShardManager();

    $key = "product_$id";
    $product = $cache->get($key);

    return response()->json(['data' => $product]);
}
```
Bảng tóm tắt hoạt động
```markdown
| Hàm              | Mục đích                              | Hoạt động chính                                      | Kết quả           |
| ---------------- | ------------------------------------- | ---------------------------------------------------- | ----------------- |
| `cacheProduct()` | Lưu dữ liệu vào Memcached shard       | Dựa vào ID, phân tích key để lưu vào server A hoặc B | Đã lưu đúng shard |
| `getProduct()`   | Lấy dữ liệu từ đúng shard trong cache | Phân tích lại key để tìm đúng nơi đã lưu             | Trả JSON sản phẩm |
```
### 3.4. Kiểm thử Sharding
Kiểm tra dữ liệu được lưu đúng server:

- Trên máy 192.168.1.10 (chứa chẵn):
```bash
memcached-tool 127.0.0.1:11211 stats | grep curr_items
```
Trên máy 192.168.1.11 (chứa lẻ):
```bash
memcached-tool 127.0.0.1:11211 stats | grep curr_items
```
Khi lưu product_2, nó chỉ xuất hiện ở server A, còn product_3 thì chỉ ở server B.

-> Như vậy, nhóm đã đáp ứng được tiêu chí Sharding
- Phân chia dữ liệu theo khóa (key)
- Hoạt động trên nhiều server
- Có logic định tuyến dữ liệu rõ ràng
- Có thể mở rộng thêm shard khác
## 4. Simple Monitoring / Logging
### 4.1. Mục tiêu
Hệ thống phải cung cấp khả năng giám sát đơn giản, ví dụ như nhật ký (logs), đầu ra CLI hoặc bảng điều khiển đơn giản. Mục tiêu của nhóm:
- Biết dữ liệu nào đang được lưu

- Lưu ở server nào (chẵn/lẻ)

- Khi nào dữ liệu được đọc/ghi thành công

- Nếu có lỗi thì log lỗi ra đâu đó dễ theo dõi

### 4.2. Giải pháp tổng thể
```markdown
| Thành phần                 | Cách triển khai cụ thể                                                               |
| -------------------------- | ------------------------------------------------------------------------------------ |
| Logging dữ liệu lưu và đọc | Ghi log khi set/get (thành công/thất bại), hiển thị thông tin server, key, thời gian |
| CLI kiểm tra tình trạng    | Lệnh Artisan để xem trạng thái shard (số lượng key, trạng thái kết nối)              |
| Log lỗi                    | Bắt lỗi kết nối server và log vào file `storage/logs/memcached.log`                  |
| Giám sát CLI đơn giản      | Tạo Artisan command tên `shard:status` để in thông tin phân bố dữ liệu               |
```
- Ghi log trong MemcachedShardManager.php
Chỉnh lại set() và get() để ghi log:
```php
use Illuminate\Support\Facades\Log;

public function set($key, $value, $ttl = 3600)
{
    $client = $this->getClientByKey($key);
    $success = $client->set($key, $value, $ttl);

    Log::channel('memcached')->info("SET [$key] vào " . $this->getShardName($key) . " - Thành công: " . ($success ? 'YES' : 'NO'));

    return $success;
}

public function get($key)
{
    $client = $this->getClientByKey($key);
    $result = $client->get($key);

    Log::channel('memcached')->info("GET [$key] từ " . $this->getShardName($key) . " - " . (is_null($result) ? "Không tìm thấy" : "Đã lấy"));

    return $result;
}

protected function getShardName($key)
{
    $id = (int) filter_var($key, FILTER_SANITIZE_NUMBER_INT);
    return ($id % 2 === 0) ? 'Server A (chẵn)' : 'Server B (lẻ)';
}
```
Bảng tóm tắt hoạt động
```markdown
| Chức năng        | Mô tả chi tiết                                                                 |
| ---------------- | ------------------------------------------------------------------------------ |
| `set()`          | Ghi log khi lưu key, hiển thị shard tương ứng và kết quả (thành công/thất bại) |
| `get()`          | Ghi log khi đọc key, chỉ ra shard và có lấy được hay không                     |
| `getShardName()` | Xác định tên shard dựa vào số lẻ/chẵn từ key                                   |
```
Nhóm sẽ thiết kế chức năng logging vào class quản lý Memcached để dễ giám sát. Mỗi lần lưu hoặc truy xuất dữ liệu (thông qua set/get), hệ thống sẽ tự động ghi lại log bao gồm tên key, server xử lý (chẵn/lẻ), và kết quả. Việc xác định server nào dựa trên ID trong key — nếu ID chẵn thì lưu ở server A, nếu lẻ thì server B. Các log này được lưu vào file riêng để khi cần có thể kiểm tra lại nhanh chóng.

- Tạo file cấu hình log riêng
Trong config/logging.php, thêm channel memcached:
```php
'memcached' => [
    'driver' => 'single',
    'path' => storage_path('logs/memcached.log'),
    'level' => 'info',
],
```
Nhóm sẽ tạo một channel log riêng tên là memcached, để tách biệt log ghi nhận hoạt động lưu và lấy cache. Việc này giúp theo dõi từng thao tác một cách dễ dàng mà không bị trộn với các log khác trong hệ thống.
Bảng tóm tắt hoạt động:
```markdown
| Chức năng   | Mô tả chi tiết                                                               |
| ----------- | ---------------------------------------------------------------------------- |
| Channel log | Ghi riêng log của Memcached vào file `memcached.log` để dễ quản lý, theo dõi |
| Mức độ log  | Mức log là `info`, phù hợp để ghi nhật ký thao tác                           |
```
- Kết quả:
Khi chạy cacheProduct(2):
storage/logs/memcached.log
```log
[2025-06-01 20:20:00] local.INFO: SET [product_2] vào Server A (chẵn) - Thành công: YES
```
Khi gọi getProduct(2):
```log
[2025-06-01 20:20:10] local.INFO: GET [product_2] từ Server A (chẵn) - Đã lấy
```
Bảng tóm tắt hoạt động:
```markdown
| Nội dung log         | Ý nghĩa                                                           |
| -------------------- | ----------------------------------------------------------------- |
| `SET [product_2]...` | Cho biết key `product_2` đã được lưu vào shard chẵn và thành công |
| `GET [product_2]...` | Cho biết key được đọc lại thành công từ đúng shard                |
```
### 4.3. Tạo command CLI đơn giản để theo dõi
Chạy:
```bash
php artisan make:command ShardStatusCommand
```
Trong ShardStatusCommand.php:
```php
namespace App\Console\Commands;

use Illuminate\Console\Command;

class ShardStatusCommand extends Command
{
    protected $signature = 'shard:status';
    protected $description = 'Xem tình trạng Memcached Shards';

    public function handle()
    {
        $servers = [
            'Server A (chẵn)' => '192.168.1.10',
            'Server B (lẻ)'   => '192.168.1.11',
        ];

        foreach ($servers as $label => $ip) {
            $stats = @fsockopen($ip, 11211, $errno, $errstr, 1)
                ? shell_exec("echo stats | nc $ip 11211 | grep curr_items")
                : "Lỗi kết nối ($errno)";

            $this->info("[$label] - $ip");
            $this->line($stats);
            $this->line('---------------------------');
        }

        return 0;
    }
}
```
Nhóm viết thêm một lệnh Artisan tên là shard:status để kiểm tra tình trạng cache trên từng shard Memcached.
Cụ thể, lệnh này gửi lệnh stats đến từng server qua cổng 11211 và hiển thị ra số lượng key đang được lưu (curr_items).
Nhờ vậy, Nhóm có thể biết dữ liệu có đang phân đúng không, hoặc có shard nào bị lỗi hay không. Nếu không kết nối được, lệnh cũng báo lỗi rõ ràng.

Bảng tóm tắt hoạt động:
```markdown
| Chức năng              | Mô tả chi tiết                                                              |
| ---------------------- | --------------------------------------------------------------------------- |
| Command `shard:status` | Cho phép chạy thủ công từ terminal để kiểm tra từng Memcached server        |
| `fsockopen()`          | Kiểm tra server có đang chạy không (nếu lỗi thì báo lỗi)                    |
| `shell_exec()`         | Gửi lệnh `stats` đến server qua netcat (`nc`) để lấy số lượng item đang lưu |
```
Kết quả CLI:

Chạy lệnh:
```bash
php artisan shard:status
```
Kết quả:

```pgsql
[Server A (chẵn)] - 192.168.1.10
STAT curr_items 3
---------------------------
[Server B (lẻ)] - 192.168.1.11
STAT curr_items 2
---------------------------
```
## 5.Basic Stress Test
### 5.1. Mục tiêu
Thực hiện kiểm tra tải cơ bản bằng cách mô phỏng nhiều yêu cầu, nhiều khách hàng và quan sát hành vi của hệ thống như sau:

- Giả lập nhiều yêu cầu đồng thời (nhiều client truy cập)

- Quan sát hành vi của hệ thống Memcached phân mảnh

- Kiểm tra tính ổn định và tốc độ phản hồi

### 5.2. Hướng giải quyết
- Tạo endpoint đơn giản để đọc/ghi dữ liệu cache product_id

- Dùng công cụ stress test như ab (Apache Benchmark) hoặc wrk để gửi hàng trăm yêu cầu đồng thời.

- Ghi log / đo thời gian phản hồi và kiểm tra trạng thái từng shard

- Ghi nhận log để đánh giá tải, và phát hiện vấn đề nếu có.

Bước 1 – Tạo route stress test

Trong routes/web.php hoặc routes/api.php:
```php
Route::get('/stress/{id}', function ($id) {
    $cache = new \App\Services\MemcachedShardManager();

    $key = "product_$id";
    $value = ['name' => 'Áo thun', 'price' => rand(100000, 300000)];

    $cache->set($key, $value);   // lưu cache
    $result = $cache->get($key); // đọc lại cache

    return response()->json(['cached' => $result]);
});
```
Bảng tóm tắt hoạt động:
```markdown
| Thành phần     | Chức năng                                |
| -------------- | ---------------------------------------- |
| `/stress/{id}` | Nhận ID sản phẩm giả lập                 |
| `set()`        | Ghi dữ liệu vào shard phù hợp (chẵn/lẻ)  |
| `get()`        | Đọc lại ngay sau đó để kiểm tra phản hồi |
```

Bước 2 – Chạy stress test với ab

Trong terminal (máy thật):
```bash
ab -n 500 -c 50 http://your-ip/stress/2
```
Ý nghĩa:

- -n 500: gửi tổng cộng 500 request

- -c 50: đồng thời 50 kết nối

- /stress/2: ID sản phẩm → sẽ ghi vào shard A (chẵn)

Bước 3 – Kiểm tra hành vi hệ thống
- Mở log:
```bash
tail -f storage/logs/memcached.log
```
- Quan sát:
Các dòng log SET và GET sẽ xuất hiện liên tục.

Kiểm tra nếu có false, timeout, lỗi.

- Kiểm tra tình trạng shard:
```bash
php artisan shard:status
```
Xem curr_items tăng theo số lượng key đã test.

Bước 4 – Ghi lại và đánh giá

- Sau test:

Ghi lại tốc độ phản hồi từ ab:
```sql
Requests per second:    300 [#/sec]
Time per request:       150 ms
Failed requests:        0
```
Kiểm tra shard có bị nghẽn không (curr_items có tăng bất thường không)

So sánh shard A và B xem có mất cân bằng tải không

-> Kết luận tóm tắt:
```markdown
| Mục tiêu                 | Đã thực hiện                   |
| ------------------------ | ------------------------------ |
| Tải đồng thời cao        | Dùng `ab -c 50 -n 500`         |
| Ghi + đọc dữ liệu cache  | `/stress/{id}` route           |
| Giám sát hoạt động shard | Artisan + log riêng            |
| Kiểm tra phản hồi        | `Requests/sec`, `Time/request` |
```
- Kết quả sau khi test:
```markdown
| Tiêu chí                   | Kết quả đạt được                                                      |
| -------------------------- | --------------------------------------------------------------------- |
| Công cụ test               | Apache Benchmark (`ab -n 500 -c 50`)                                  |
| Tổng số request            | 500                                                                   |
| Số request đồng thời       | 50                                                                    |
| Tốc độ xử lý               | \~213 request/giây                                                    |
| Tỉ lệ lỗi                  | 0% (Không có request nào bị lỗi)                                      |
| Thời gian mỗi request      | Trung bình \~234 ms                                                   |
| Shard phân phối            | Dữ liệu `product_2` → shard A (192.168.1.10)                          |
| Số lượng key trên Server A | 500                                                                   |
| Số lượng key trên Server B | 5 (không liên quan tới test)                                          |
| Log hệ thống               | Ghi đầy đủ từng `set/get` → xác nhận đúng shard, không có lỗi         |
| Monitoring (CLI)           | `php artisan shard:status` hiển thị số lượng `curr_items` từng server |
```
-> Như vậy, hệ thống hoạt động ổn định dưới áp lực cao, phân mảnh dữ liệu đúng, không lỗi, dễ theo dõi và kiểm soát