# 🔑 Senior Keywords — Vue Compiler Optimizations & Vapor Mode

> Từ khóa nâng cao phỏng vấn Senior Vue Developer.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| Virtual DOM | `Patch Flags & Block Tree` | Compiler gắn cờ đánh dấu đúng vị trí binding động (text, class, style) giúp Virtual DOM bỏ qua việc diff các node tĩnh | `Vue 3 Patch Flags Block Tree Virtual DOM diffing` |
| Compilation Optimization | `Static Hoisting` | Tự động kéo các phần tử HTML tĩnh ra ngoài render function để tạo 1 lần duy nhất và tái sử dụng mãi mãi | `Vue 3 Compiler Static Hoisting` |
| Future Architecture | `Vue Vapor Mode` | Hướng đi không sử dụng Virtual DOM của Vue, biên dịch thẳng code Vue thành các câu lệnh thao tác DOM trực tiếp sắc bén giống Svelte | `Vue Vapor Mode no Virtual DOM compilation` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **Slot Flag Optimization**: Phân biệt Stable Slot và Dynamic Slot để quyết định ranh giới re-render của parent/child.
- **Custom Renderer API**: Cung cấp `createRenderer()` cho phép dùng Vue render lên Canvas, WebGL hoặc Terminal.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **Vapor Mode Performance Gain**: Giảm dung lượng bộ nhớ RAM và tăng tốc độ render lên 2-3 lần so với Virtual DOM.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Why Vue 3 uses Proxy over Object.defineProperty**: Giải thích 3 hạn chế lớn của `Object.defineProperty` trong Vue 2 (không bắt được thêm/xóa prop, không bắt được mảng index) và cách Proxy khắc phục.

