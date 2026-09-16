# 🔑 Senior Keywords — Vue 3 Composition API & Reactivity Mechanics

> Từ khóa về hệ thống Reactivity Proxy của Vue 3 và Composition API.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Reactivity System | `Proxy-based Reactivity (track & trigger)` | Cơ chế Vue 3 dùng ES6 Proxy để tự động lồng `track()` ghi nhận dependency và `trigger()` kích hoạt effect khi gán giá trị | `Vue 3 Reactivity Proxy track trigger` |
| Reactivity Primitive | `ref RefImpl vs reactive Proxy` | Phân biệt `ref()` bọc giá trị đơn trong đối tượng `RefImpl` với `value` getter/setter vs `reactive()` bọc trực tiếp Object bằng Proxy | `Vue 3 ref RefImpl vs reactive Proxy` |
| Advanced Reactivity | `shallowRef & triggerRef` | Tạo ref chỉ theo dõi cấp ngoài cùng để tối ưu hiệu năng cho các object dữ liệu cực lớn | `Vue 3 shallowRef triggerRef performance` |
| Effect Scope | `effectScope API` | Gộp và hủy hàng loạt các reactive effects (`computed`, `watch`) cùng một lúc khi ngắt kết nối module | `Vue 3 effectScope clean up reactive effects` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Unwrapping Refs in Templates**: Vue tự động tháo bọc `.value` cho `ref` khi truy cập trong `<template>`.
- **Read-only & Reactive Wrappers**: Hàm `readonly()` tạo bọc Proxy chặn tất cả các thao tác mutation đối tượng.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **`markRaw` Optimization**: Đánh dấu đối tượng không bao giờ biến thành Reactive để tránh chi phí tạo Proxy không cần thiết (dùng cho map/chart instances).

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Losing Reactivity in Destructuring**: Lỗi mất tính phản ứng khi destructure object từ `reactive()` và cách dùng `toRefs()` để khắc phục.

