# 🔑 Senior Keywords — Angular SSR, Hydration & Advanced Interviews

> Từ khóa phục vụ phỏng vấn Senior Angular Developer.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| SSR & Hydration | `Non-Destructive Hydration` | Kỹ thuật tái sử dụng DOM đã được Server render thay vì xóa đi vẽ lại hoàn toàn trong Angular 17+ | `Angular Hydration Non Destructive SSR` |
| Modern Template | `Deferrable Views (@defer)` | Cú pháp trì hoãn tải và render các block UI nặng dựa trên điều kiện (`on viewport`, `on interaction`, `on idle`) | `Angular Deferrable Views defer block lazy loading` |
| Template Compiler | `Control Flow Syntax (@if, @for)` | Cú pháp điều khiển mới thay thế `*ngIf`, `*ngFor` với cú pháp gọn và hiệu năng biên dịch cao hơn | `Angular Control Flow Syntax if for track` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **ViewContainerRef & EmbeddedViewRef**: API thao tác trực tiếp với DOM ngầm của Angular để tạo dynamic components.
- **Pure vs Impure Pipes**: Pure Pipe chỉ tính toán lại khi tham chiếu đầu vào đổi; Impure Pipe chạy mỗi chu kỳ Change Detection.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **TrackBy Function in @for**: Cung cấp định danh duy nhất để Angular chỉ cập nhật đúng phần tử thay đổi trong danh sách.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Hydration Mismatch Warnings**: Nguyên nhân xảy ra lỗi lệch DOM giữa Server và Client (do dùng `window`, `Math.random()`) và cách khắc phục.

