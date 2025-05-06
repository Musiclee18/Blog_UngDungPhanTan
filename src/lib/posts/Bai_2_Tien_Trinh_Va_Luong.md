---
title: "Tiến Trình Và Luồng"
date: "2025-5-5"
updated: "2025-5-5"
categories:
  - "sveltekit"
  - "markdown"
  - "Tiến Trình Và Luồng" 
coverImage: "/images/bai_2.webp"
coverWidth: 16
coverHeight: 9
excerpt: Tìm hiểu về tiến trình và luồng
---

Lê Tiến Nhạc -- 22010108

# Tiến trình & Luồng

## 1. Đánh giá hiệu năng máy tính Lenovo Legion Y7000

### CPU: Intel Core i7-13650HX
- 14 nhân (6 hiệu năng cao + 8 tiết kiệm điện), 20 luồng, xung nhịp tối đa 4.9GHz.
- Với kiến trúc hybrid, CPU này xử lý đa nhiệm hiệu quả, phù hợp cho các tác vụ nặng như lập trình, dựng hình 3D, và chơi game.

### GPU: NVIDIA GeForce RTX 4060 8GB
- Kiến trúc Ada Lovelace, hỗ trợ DLSS 3 và Ray Tracing.
- Đảm bảo chơi mượt các game AAA ở thiết lập cao, hỗ trợ tốt cho phần mềm đồ họa và AI.

### RAM: 24GB DDR5
- Tốc độ cao, hỗ trợ đa nhiệm mượt mà.
- Phù hợp cho lập trình, thiết kế, và xử lý dữ liệu lớn.

### Tổng kết
> Cấu hình này rất mạnh mẽ, phù hợp cho học tập, làm việc và giải trí trong ngành CNTT.

---

## 2. 12 bài toán sử dụng đa luồng / đa tiến trình

```markdown
| STT | Bài toán                              | Loại sử dụng       | Ghi chú                                          |
|-----|---------------------------------------|--------------------|--------------------------------------------------|
| 1   | Xử lý ảnh song song                   | Đa luồng           | Mỗi ảnh xử lý trong một luồng                    |
| 2   | Máy chủ HTTP                          | Đa tiến trình      | Mỗi request tạo 1 tiến trình con                 |
| 3   | Trình duyệt web                       | Cả hai             | Mỗi tab là 1 tiến trình, trong tab có luồng      |
| 4   | Dịch tệp lớn (PDF, video)             | Đa luồng           | Tách thành từng phần để dịch song song           |
| 5   | Game Engine                           | Đa luồng           | Render, âm thanh, vật lý tách luồng riêng        |
| 6   | IDE biên dịch code                    | Đa tiến trình      | Mỗi dự án/module là 1 tiến trình                 |
| 7   | Ứng dụng chat thời gian thực          | Đa luồng           | Luồng gửi & nhận hoạt động độc lập               |
| 8   | Phát video trực tuyến                 | Đa tiến trình      | Xử lý mã hóa, stream trong tiến trình khác nhau  |
| 9   | Hệ thống ngân hàng                    | Đa tiến trình      | Mỗi giao dịch xử lý riêng để đảm bảo an toàn     |
| 10  | Trình soạn thảo văn bản               | Đa luồng           | Kiểm tra chính tả, autosave chạy nền             |
| 11  | Điều khiển công nghiệp IoT            | Đa tiến trình      | Mỗi thiết bị là một tiến trình giám sát riêng    |
| 12  | Huấn luyện học máy                    | Đa tiến trình      | Dữ liệu & mô hình chia nhỏ, train song song      |
```

---

## 3. Khi nào dùng Thread, Process, hoặc cả hai?

![Bảng so sánh Thread vs Process](/images/anhmuc3bai2.jpg)


---

## 4. Cách ChatGPT sử dụng hệ thống phân tán để huấn luyện

### Tổng quan:
ChatGPT (GPT-3.5/GPT-4) được huấn luyện trên hệ thống phân tán sử dụng hàng ngàn GPU kết nối trong cụm máy chủ. Mục tiêu là xử lý hàng trăm tỷ tham số của mô hình và hàng terabyte dữ liệu văn bản một cách hiệu quả.

### Kỹ thuật chính:
- **Data Parallelism**: Chia dữ liệu thành các phần và gửi đến các GPU khác nhau để train đồng thời.
- **Model Parallelism**: Chia mô hình lớn (nhiều layer hoặc tensor) để mỗi GPU xử lý một phần.
- **Pipeline Parallelism**: Các GPU hoạt động theo chuỗi, truyền dữ liệu từ GPU này sang GPU khác theo từng lớp mạng.
- **Distributed Training Framework**: Sử dụng các công cụ như NVIDIA Megatron, DeepSpeed, hoặc PyTorch Distributed để quản lý và tối ưu hóa huấn luyện.
- **High-speed interconnect**: Kết nối tốc độ cao giữa các GPU như NVLink, InfiniBand để đảm bảo dữ liệu không bị nghẽn.

### Hạ tầng phần cứng:
- Cụm máy chủ chứa hàng nghìn GPU NVIDIA A100
- Dữ liệu lưu trữ phân tán trên hệ thống lưu trữ tốc độ cao
- Hệ thống giám sát & điều phối như Kubernetes, Ray hoặc SLURM

### Tài liệu tham khảo:
- [AssemblyAI: How ChatGPT Actually Works](https://assemblyai.com/blog/how-chatgpt-actually-works)
- [Wired: These Startups Are Building Advanced AI Models Without Data Centers](https://www.wired.com/story/these-startups-are-building-advanced-ai-models-over-the-internet-with-untapped-data)
```
