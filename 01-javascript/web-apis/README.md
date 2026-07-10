# 🌐 Web APIs — Các API Trình Duyệt Tiêu Chuẩn

> Tài liệu chuyên sâu về các API tiêu chuẩn được cung cấp bởi trình duyệt (Web Platform APIs) giúp xây dựng ứng dụng Frontend mạnh mẽ.

---

## 📂 Cấu Trúc Thư Mục

| File | Nội dung |
| :--- | :--- |
| 📑 [`README.md`](./README.md) | Bản đồ học tập các Web APIs của trình duyệt |
| ├── 📑 [`real-time-communication.md`](./real-time-communication.md) | Tài liệu chuyên sâu về WebSocket, Server-Sent Events (SSE / EventSource), Short/Long Polling |

---

## 🗺️ Bản Đồ Học Tập Web APIs (Roadmap)

Trong phát triển frontend hiện đại, việc làm chủ các API của trình duyệt giúp giảm thiểu sự phụ thuộc vào các thư viện bên thứ ba và tối ưu hóa hiệu năng ứng dụng. Bản đồ học tập bao gồm:

1. **Network & Real-time APIs:**
   - [`real-time-communication.md`](./real-time-communication.md): WebSocket, EventSource (SSE), Polling.
   - Fetch API, AbortController (hủy bỏ request).
2. **Storage & Caching APIs:**
   - localStorage, sessionStorage, Cookie.
   - IndexedDB (CSDL phía client cho ứng dụng offline).
   - Cache Storage API (sử dụng trong Service Workers).
3. **Background Processing & Multi-threading:**
   - Web Workers (chạy tác vụ nặng ở luồng riêng biệt).
   - Service Workers (offline caching, push notifications, background sync).
4. **Performance & Hardware APIs:**
   - Performance API (đo lường tốc độ, TTI, FCP).
   - Intersection Observer (lazy loading ảnh, infinite scroll).
   - Resize Observer & Mutation Observer.
