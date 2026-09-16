# 🔑 Senior Keywords — Tailwind CSS Engine & Design Systems

> Từ khóa về JIT Engine, Tailwind Merge và Utility-First Architecture.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Tailwind Engine | `Just-In-Time (JIT) Compiler` | Compiler quét mã nguồn phát hiện class name và sinh CSS tương ứng On-demand trong thời gian thực | `Tailwind JIT engine build performance` |
| Class Resolution | `Tailwind Merge Algorithm` | Giải quyết xung đột class name CSS (ví dụ `px-2` vs `px-4`) bằng cách phân tích AST class thay vì chuỗi đơn thuần | `tailwind-merge class conflict resolution` |
| Design System | `Design Tokens Mapping` | Cấu hình `tailwind.config.js` để đồng bộ token màu sắc, typography từ Figma vào utility class | `Tailwind config design tokens extension` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Content Purging & Scanner**: Thuật toán regex quét chuỗi trong source code để lọc bỏ các class không được sử dụng khỏi bundle production.
- **Arbitrary Values Syntax**: Cú pháp `w-[123px]` cho phép tạo utility class động mà không cần khai báo trước.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **CSS Bundle Size Capping**: Đảm bảo dung lượng CSS luôn nằm trong ngưỡng < 15KB gzipped bất kể quy mô dự án.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Dynamic Class Name Antipattern**: Lỗi ghép chuỗi class (`text-${color}-500`) khiến JIT scanner không phát hiện được class và cách khắc phục dùng Safelist hoặc Full Class Lookup.

