# Bài 4: Xây Dựng Yêu Cầu Phi Chức Năng - NFR (Dạng Tối ưu)

**1. Mục tiêu:**
Điều hướng AI thiết lập các tiêu chuẩn phi chức năng (Non-Functional Requirements) đáp ứng các rào cản về kiến trúc thực tế.

## 1. Nội dung Prompt

```text
**Vai trò:** Bạn là một Software Architect và System Analyst dày dạn kinh nghiệm.

**Mục tiêu:** Viết Đặc tả Yêu cầu Phi chức năng (Non-Functional Requirements - NFR) cho hệ thống Guai-api của dự án Shop AI.

**Ngữ cảnh:** Hệ thống Shop AI sắp ra mắt và cần chuẩn bị hạ tầng đáp ứng các đợt Flash Sale có lượng truy cập (traffic) đột biến rất lớn. 

**Ràng buộc:** Bắt buộc phân tích và đưa ra chỉ tiêu kỹ thuật số lượng (metrics) và chiến lược hạ tầng rõ ràng cho 3 khía cạnh sau:
1. Thời gian phản hồi (Response Time) tối đa cho các endpoint tra cứu sản phẩm (Product Search).
2. Hiệu suất cơ sở dữ liệu MySQL (yêu cầu đề xuất rõ chiến lược Indexing cho các bảng thường xuyên bị query).
3. Độ trễ (latency) và an toàn bảo mật của luồng cấp phát/xác thực token JWT.

**Định dạng:** Trình bày 100% dưới dạng Bảng Markdown (Markdown Table) gồm các cột: Khía cạnh, Hạng mục NFR, Chỉ tiêu kỹ thuật (Metrics), và Đề xuất kỹ thuật/Chiến lược.
```

## 2. Bảng tài liệu NFR do AI sinh ra

| Khía cạnh | Hạng mục NFR | Chỉ tiêu kỹ thuật (Metrics) | Đề xuất kỹ thuật / Chiến lược |
| :--- | :--- | :--- | :--- |
| **Hiệu năng API (Response Time)** | Tốc độ tra cứu sản phẩm (Product Search) | - 95th percentile (P95) < 150ms.<br>- Max response time < 300ms với 10,000 CCU. | - Triển khai Redis Cache để lưu kết quả query các sản phẩm Hot/Flash Sale.<br>- Sử dụng Elasticsearch cho tìm kiếm full-text thay vì LIKE '%query%' trong DB trực tiếp. |
| **Hiệu suất Cơ sở dữ liệu** | Tối ưu hóa truy vấn MySQL | - Thời gian query DB trung bình < 50ms.<br>- Không có truy vấn quét toàn bảng (Full Table Scan) trong luồng chính. | **Chiến lược Indexing:**<br>- Bảng `Products`: Tạo Composite Index `(category_id, status)` và Index trên `price` cho các filter tìm kiếm.<br>- Bảng `Orders`: Tạo Index trên `user_id` và `created_at`.<br>- Cấu hình Read Replicas (Master-Slave) để tách biệt luồng đọc/ghi dữ liệu. |
| **Bảo mật & Độ trễ Auth** | Cấp phát & Xác thực JWT | - Độ trễ quá trình verify JWT < 10ms/request.<br>- Độ rò rỉ token: Bằng 0. | - Cấu hình Secret Key độ dài tối thiểu 256-bit (HS256) hoặc RSA (RS256).<br>- Token TTL (Time-to-live): Access Token ngắn hạn (15-30 phút), Refresh Token (7 ngày).<br>- Không hit vào DB khi verify token, chỉ giải mã payload trên RAM bằng Security Filter. |
