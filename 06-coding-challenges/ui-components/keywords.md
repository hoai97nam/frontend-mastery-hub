# 🔑 Senior Keywords — High-Performance UI Coding Challenges

> Từ khóa xây dựng các UI component phức tạp đòi hỏi tối ưu hiệu năng cao.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Performance UI | `Virtualized List (Windowing)` | Kỹ thuật chỉ render đúng các phần tử HTML nằm trong khung nhìn viewport và tái sử dụng DOM khi scroll | `Virtual List Windowing React Virtualized react-window DOM recycling` |
| Browser Observer | `Intersection Observer Infinite Scroll` | Thay thế sự kiện `scroll` bằng `IntersectionObserver` để phát hiện phần tử cuối trang nạp thêm dữ liệu không gây giật lag | `Intersection Observer Infinite Scroll performance` |
| Timing Control | `Debounce with Leading/Trailing` | Kỹ thuật hoãn thực thi hàm cho tới khi người dùng ngừng thao tác trong khoảng thời gian N ms, hỗ trợ cờ leading/trailing | `Debounce vs Throttle leading trailing options` |
| Browser API | `HTML5 Drag and Drop API` | Sử dụng các sự kiện `dragstart`, `dragover`, `drop` và `DataTransfer` để kéo thả phần tử chuẩn native | `HTML5 Drag and Drop DataTransfer API` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **DOM Node Recycling**: Giữ nguyên số lượng DOM node cố định trong Virtual List và chỉ thay đổi vị trí `transform: translateY()` cùng dữ liệu bên trong.
- **Throttle via `requestAnimationFrame`**: Điều chỉnh tần suất gọi hàm callback khớp với chu kỳ rải điểm ảnh của màn hình (60Hz/120Hz).

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Passive Event Listeners**: Thêm `{ passive: true }` vào listener `touchstart`/`touchmove` để trình duyệt scroll mượt mà không chờ `preventDefault()`.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Build a Debounce/Throttle from scratch**: Viết hàm debounce/throttle thuần trong JS hỗ trợ hủy (`cancel`) và thực thi lập tức (`flush`).

