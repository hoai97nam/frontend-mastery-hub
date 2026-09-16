# 🔑 Senior Keywords — SCSS Architecture & Modular Styling

> Từ khóa thiết kế kiến trúc Sass/SCSS quy mô lớn.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Architecture Pattern | `7-1 SCSS Structure` | Mô hình tổ chức 7 thư mục (abstracts, base, components, layout, pages, themes, vendors) và 1 file `main.scss` | `Sass 7-1 pattern architecture` |
| Modular Sass | `@use vs @import` | Hệ thống module mới của Sass khắc phục vấn đề global namespace pollution của `@import` bằng cách tạo namespace riêng | `Sass use vs import module system` |
| Optimization | `%placeholder vs @mixin` | Dùng `%placeholder` với `@extend` để nhóm CSS selector giảm dung lượng CSS xuất ra so với nhân bản code của `@mixin` | `Sass placeholder extend vs mixin output size` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Sass Module Namespacing**: Cơ chế gọi hàm/variable với namespace (`math.$pi`, `variables.$color-primary`).
- **Design Tokens Integration**: Cầu nối giữa Sass compile-time variables và CSS runtime custom properties.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **CSS Custom Properties Bridging**: Chuyển các biến SCSS động thành CSS variables để thay đổi theme thời gian thực mà không cần re-compile.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **@extend Specificity Side-effects**: Giải thích bẫy phình to CSS selector và thay đổi thứ tự ưu tiên khi dùng `@extend` không cẩn thận.

