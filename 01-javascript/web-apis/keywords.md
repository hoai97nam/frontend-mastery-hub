# 🔑 Senior Keywords — Web APIs & Real-time Communication

> Từ khóa chuyên sâu về giao thức kết nối thời gian thực và các Web APIs hiện đại của trình duyệt.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Real-time Protocol | `WebSocket Framing Protocol` | Giao thức 2 chiều full-duplex chạy trên TCP với khung dữ liệu frame siêu nhẹ, không tốn HTTP overhead | `WebSocket framing mask bit binary data` |
| Real-time Resilience | `Exponential Backoff with Jitter` | Thuật toán reconnect thông minh tăng thời gian chờ ngẫu nhiên tránh hiện tượng Thundering Herd cho server | `Exponential backoff jitter reconnect algorithm` |
| Real-time Protocol | `SSE Chunked Transfer` | Server-Sent Events sử dụng header `text/event-stream` trên kết nối HTTP đơn hướng truyền dữ liệu liên tục | `Server Sent Events HTTP Chunked Transfer Encoding` |
| Browser Storage | `IndexedDB Transaction Lock` | Cơ chế khóa giao dịch Read-Only / Read-Write đảm bảo tính toàn vẹn dữ liệu bất đồng bộ trong trình duyệt | `IndexedDB transaction lock concurrency` |
| Multi-threading | `Web Worker Message Passing` | Cơ chế truyền dữ liệu giữa Main Thread và Worker thread qua `postMessage` hoặc `Structured Clone / Transferable Objects` | `Web Worker Transferable Objects ArrayBuffer` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **HTTP Upgrade Handshake**: Quá trình bắt tay chuyển từ giao thức HTTP/1.1 sang WebSocket bằng header `Upgrade: websocket` và `Sec-WebSocket-Accept`.
- **HTTP Connection Hold (Long Polling)**: Kỹ thuật giữ kết nối HTTP ở trạng thái chờ trên Server cho tới khi có sự kiện mới hoặc timeout.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Transferable Objects**: Chuyển giao quyền sở hữu bộ nhớ (`ArrayBuffer`) giữa các thread mà không tốn chi phí copy dữ liệu.
- **BroadcastChannel API**: Kỹ thuật giao tiếp giữa các Tabs/Windows cùng origin mà không cần thông qua Server hay LocalStorage polling.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **SSE vs WebSocket Decision Matrix**: Phân biệt khi nào dùng SSE (thông báo 1 chiều từ server, tự động reconnect) vs WebSocket (chat, game 2 chiều).
- **Browser Storage Limits & Quota**: Cách quản lý dung lượng lưu trữ IndexedDB/Storage API và xử lý lỗi `QuotaExceededError`.

