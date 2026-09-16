# 🔑 Senior Keywords — ES6+ Metaprogramming & Modern Features

> Các từ khóa về Metaprogramming, Proxy, Symbols và Async Generators.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Metaprogramming | `ES6 Proxy Traps` | Can thiệp trực tiếp vào các thao tác cơ bản của object (get, set, has, deleteProperty) phục vụ reactivity | `ES6 Proxy traps Reflect API reactivity` |
| Metaprogramming | `Reflect API` | Cung cấp các phương thức chuẩn hóa để thao tác với object, trả về boolean thay vì ném exception | `JavaScript Reflect API Proxy traps` |
| GC Collections | `WeakMap Ephemerons` | Cấu trúc dữ liệu lưu trữ key dưới dạng tham chiếu yếu, cho phép GC tự động xóa entry khi key không còn trỏ đến | `JavaScript WeakMap Ephemerons Garbage Collection` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Well-Known Symbols**: Các Symbol đặc biệt (`Symbol.iterator`, `Symbol.toPrimitive`, `Symbol.species`) cho phép tùy biến hành vi ngôn ngữ.
- **Async Generators (`for await...of`)**: Kết hợp Generator với Promise tạo luồng stream xử lý dữ liệu bất đồng bộ tuần tự.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **WeakSet for Object Tagging**: Đánh dấu đối tượng mà không gây ra memory leak nhờ cơ chế weak reference.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Proxy Performance Overhead**: Phân tích chi phí hiệu năng khi bọc Proxy xung quanh các mảng/đối tượng lớn trong ứng dụng real-time.

