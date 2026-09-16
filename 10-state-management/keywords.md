# 🔑 Senior Keywords — State Management Architecture Across Frameworks

> Từ khóa so sánh các trường phái quản lý state trong toàn bộ hệ sinh thái Web.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| State Paradigm | `Flux Uni-directional Data Flow` | Luồng dữ liệu 1 chiều: Action -> Dispatcher -> Store -> View (Redux / NgRx / Vuex) | `Flux Architecture Uni directional Data Flow Redux` |
| State Paradigm | `Proxy Mutability & Change Tracking` | Cơ chế theo dõi biến đổi tự động qua ES6 Proxy cho phép viết code dạng mutable nhưng sinh ra immutable state (Valtio / MobX / Vue 3) | `Proxy Mutability Change Tracking MobX Valtio` |
| State Paradigm | `Fine-Grained Reactive Signals Graph` | Đồ thị phản ứng hạt mịn kết nối trực tiếp biến state với node DOM cụ thể không qua Virtual DOM diffing (SolidJS / Angular Signals / Vue Vapor) | `Fine Grained Reactivity Signals Graph SolidJS` |
| Real-time State | `Optimistic UI Updates & Race Conditions` | Cập nhật UI ngay lập tức trước khi server phản hồi và cơ chế rollback/cancel request trùng lặp khi người dùng thao tác nhanh | `Optimistic UI Updates Race Condition CancelToken Axios RxJS` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **State Normalization Pattern**: Tổ chức state dạng chuẩn hóa DB (`entities`, `ids`) loại bỏ dư thừa dữ liệu.
- **Push-based vs Pull-based Reactivity**: RxJS/Signals đẩy dữ liệu từ nguồn phát (Push); React re-render chủ động kéo dữ liệu (Pull).

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **State Rehydration Strategy**: Đồng bộ và làm ấm state từ SSR/LocalStorage mà không kích hoạt re-render thừa.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Architecting Real-time Financial Dashboard**: Thiết kế hệ thống state cho bảng giá chứng khoán cập nhật 1000 ticks/giây chống nghẽn UI.

