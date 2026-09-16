# 🔑 Senior Keywords — JavaScript Core Concepts & Memory Management

> Tập hợp từ khóa cốt lõi về bản chất thực thi, quản lý bộ nhớ và Event Loop trong V8 Engine.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Memory & GC | `Lexical Environment` | Giải thích cách Closure lưu giữ môi trường biến gốc ngay cả khi outer function đã return | `Lexical Environment vs Variable Environment V8` |
| Memory & GC | `Mark-and-Sweep` | Thuật toán Garbage Collection chính của V8, duyệt từ GC Roots để phát hiện object unreachable | `V8 Garbage Collector Mark and Sweep algorithm` |
| Memory & GC | `Detached DOM Tree` | Nguyên nhân memory leak khi node HTML bị tháo khỏi DOM nhưng vẫn bị giữ tham chiếu trong JS | `Chrome DevTools Detached DOM elements memory leak` |
| Async Runtime | `Microtask Starvation` | Hiện tượng Microtask Queue liên tục nhận job mới (Promise recursion) làm chặn Macrotask & UI render | `Event loop microtask queue starvation requestAnimationFrame` |
| Async Runtime | `Generational GC` | Phân chia bộ nhớ V8 thành Young Generation (Scavenger) và Old Generation (Mark-Sweep-Compact) | `V8 Young Generation Scavenger vs Old Generation` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Execution Context & Scope Chain**: Môi trường chứa biến và hàm đang thực thi. Gồm Creation Phase (hoisting/binding) và Execution Phase.
- **Temporal Dead Zone (TDZ)**: Khoảng thời gian từ khi biến `let`/`const` được khởi tạo phạm vi đến khi được gán giá trị, truy cập trong khoảng này bắn `ReferenceError`.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **WeakRef & FinalizationRegistry**: Cơ chế quản lý tham chiếu yếu cho phép GC thu hồi object và chạy cleanup callback khi object bị giải phóng.
- **Memory Leak Prevention**: Gán `null` cho tham chiếu lớn không dùng, hủy `addEventListener`, `clearInterval` và tránh closure giữ scope biến global.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Event Loop Microtask vs Macrotask Order**: Thứ tự ưu tiên: Call Stack -> Microtasks (Promise, MutationObserver, queueMicrotask) -> Render Pipeline -> Macrotasks (setTimeout, setInterval).
- **Closure Trap in Loops**: Vấn đề `var` trong vòng lặp gán callback dẫn tới chia sẻ cùng 1 tham chiếu biến, giải pháp dùng `let` hoặc IIFE.

